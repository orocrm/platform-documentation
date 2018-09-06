.. _dev-guide-system-message-queue-architecture-job:

Job
===

A job is additional information about the message processing task added to the Oro application DB. It allows to view the information about the task (job) in the admin UI and to manage the job (view the status and cancel it) in the UI.

When to Create a Job
--------------------

A :ref:`message processor <dev-guide-system-message-queue-architecture-processor>` can be implemented with or without creating jobs, which means that you need to consider what the best approach is in each specific situations. However, below are a few recommendations to help you: 

**Do Not Create Jobs If:**

* There is an easy fast-executing action, such as status changing etc.
* The action looks like an event listener.

**Create jobs if:**

* The action is complicated and can be executed for a long time.
* You need to monitor execution status.
* You need to run an unique job, i.e. not to allow running a job with the same name until the previous job has finished.
* You need to run a step-by-step action, i.e. the message flow must have several steps (tasks) which run one after another.
* You need to split a job for a set of sub-jobs to run in parallel and monitor the status of the whole task.

Jobs UI
-------

You can view the statuses and statistics for the jobs by navigating to **System > Jobs** in the main menu in the management console of all Oro applications.

.. image:: /dev_guide/system/message_queue/architecture/img/jobs.jpg
    :alt: Scheduled Jobs in admin UI

JobRunner
---------

Jobs are usually created and run with *\\Oro\\Component\\MessageQueue\\Job\\JobRunner*, and one of the following methods is used:

* **runUnique**

  *public function runUnique($ownerId, $name, \\Closure $runCallback)*

  Runs the *$runCallback*. It does not allow another job with the same name to run simultaneously.

* **createDelayed**

  *public function createDelayed($name, \\Closure $startCallback)*

  A sub-job which runs asynchronously (sending its own message). It can only run inside another job.

  Ii is a common approach to create a **delayed job** simultaneously with a **queue message** that contains information about the job. In this case, after receiving the message, the subscribed message processor can run and perform a delayed job by running the *runDelayed* method with the job data.

* **runDelayed**

  *public function runDelayed($jobId, \\Closure $runCallback)*

  This method is used inside a processor for a message which was sent with **createDelayed**.

  The *$runCallback* closure usually returns true or false, the job status depends on the returned value. See the `Jobs Statuses`_ section for the details.

  To reuse the existing processor logic in the scope of job, it may be decorated with *DelayedJobRunnerDecoratingProcessor* which will execute runDelayed, pass the control to the given processor and then handle the result in the format applicable for runDelayed.

A Dependent Job
---------------

Use a dependent job when your job flow has several steps but you want to send a new message when all steps are finished.

In the example below, a root job is created. As soon as its work is completed, it sends two messages with 'topic1' and 'topic2' to the queue.

.. code-block:: php
    :linenos:

    class MessageProcessor implements MessageProcessorInterface
    {
        /**
         * @var JobRunner
         */
        private $jobRunner;

        /**
         * @var DependentJobService
         */
        private $dependentJob;

        public function process(MessageInterface $message, SessionInterface $session)
        {
            $data = JSON::decode($message->getBody());

            $result = $this->jobRunner->runUnique(
                $message->getMessageId(),
                'oro:index:reindex',
                function (JobRunner $runner, Job $job) use ($data) {
                    // register two dependent jobs
                    // next messages will be sent to queue when that job and all children are finished
                    $context = $this->dependentJob->createDependentJobContext($job->getRootJob());
                    $context->addDependentJob('topic1', 'message1');
                    $context->addDependentJob('topic2', 'message2', MessagePriority::VERY_HIGH);

                    $this->dependentJob->saveDependentJob($context);

                    // some work to do

                    return true; // if you want to ACK message or false to REJECT
                }
            );

            return $result ? self::ACK : self::REJECT;
        }
    }

Dependant jobs can only be added to root jobs (i.e., the jobs created with *runUnique*, not *runDelayed*).

Jobs Structure
--------------

A two-level job hierarchy is created for the process where:

* A root job can have a few child jobs.
* A child job can have one root job.
* A child job cannot have child jobs of its own.
* A root job cannot have a root job of its own.
* If we use just *runUnique*, then a parent and a child jobs with the same name are created.
* If we use *runUnique* and *createDelayed* inside it, then a parent and a child job for runUnique are created. Then each run of *createDelayed* adds another child for the *runUnique* parent.

Jobs Statuses
-------------

* **Single job**: When a message is being processed by a consumer and a JobRunner method runUnique is called without creating any child jobs:

    * The root job is created and the closure passed in params runs. The job gets **Job::STATUS_RUNNING** status, the job startedAt field is set to the current time.
    * If the closure returns true, the job status is changed to **Job::STATUS_SUCCESS**, the job stoppedAt field is changed to the current time.
    * If the closure returns false or throws an exception, the job status is changed to **Job::STATUS_FAILED**, the job stoppedAt field is changed to the current time.
    * If someone interrupts the job, it stops working and gets **Job::STATUS_CANCELLED** status, the job stoppedAt field is changed to the current time.
    * If new unique job is created, but the previous job has not finished, its execution time is checked. If the execution time is longer than the configured time_before_stale, (see Stale jobs) **Job::STATUS_STALE** status is set.

* **Child jobs**: When a message is being processed by a consumer, a JobRunner method runUnique is called which creates child jobs with createDelayed:

    * The root job is created and the closure passed in params runs. The job gets **Job::STATUS_RUNNING** status, the job startedAt field is set to the current time.
    * When the JobRunner method createDelayed is called, the child jobs are created and get the **Job::STATUS_NEWstatuses**. The messages for the jobs are sent to the message queue.
    * When a message for a child job is being processed by a consumer and a JobRunner method runDelayed is called, the closure runs and the child jobs get Job::STATUS_RUNNING status.
    * If the closure returns true, the child job status is changed to **Job::STATUS_SUCCESS**, the job stoppedAt field is changed to the current time.
    * If the closure returns false or throws an exception, the child job status is changed to **Job::STATUS_FAILED**, the job stoppedAt field is changed to the current time.
    * When all child jobs are stopped, the root job status is changed according to the child jobs statuses.
    * If someone interrupts a child job, it stops working and gets **Job::STATUS_CANCELLED** status, the job stoppedAt field is changed to the current time.
    * If someone interrupts the root job, the child jobs that are already running finish their work and get the statuses according to the work result (see the description above). The child jobs that are not run yet are cancelled and get **Job::STATUS_CANCELLED** statuses.
    * If the root job status changes to **Job::STATUS_STALE**, its children automatically get the same status. (see Stale Jobs)

* **Also**: If a jobs closure returns true, the process method which runs this job should return **self::ACK**. If a job closure returns false, the process method which runs this job should return **self::REJECT**.

Stale Jobs
----------

It is not possible to create two unique jobs with the same name. That is why, if one unique job is not able to finish its work, it can block another job. To handle such situation, use **stale jobs**.

By default, *JobProcessor* uses *NullJobConfigurationProvider*, so a unique job will never be "stale". If you want to change that behavior, create your own provider that implements *JobConfigurationProviderInterface*.

The *JobConfigurationProvider::getTimeBeforeStaleForJobName($jobName);* method should return the number of seconds after which a job is considered "stale". If you do not want job to be staled, return null or -1.

In the example below, all jobs are treated as "stale" after an hour.

.. code-block:: php
    :linenos:

    <?php

    use Oro\Component\MessageQueue\Provider\JobConfigurationProviderInterface;

    class JobConfigurationProvider implements JobConfigurationProviderInterface
    {
        /**
         * {@inheritdoc}
         */
        public function getTimeBeforeStaleForJobName($jobName)
        {
            return 3600;
        }
    }

    $jobProcessor = new JobProcessor(/* arguments */);
    $jobProcessor->setJobConfigurationProvider(new JobConfigurationProvider());

In this case, if the second unique job with the same name is created but the previous job has not been updated for more than one hour and has not started a child, it gets the **Job::STATUS_STALE** status, and a new job is created.

Additionally, if the processor tries to finish a "stale" job, it is removed.

Related Cookbook Articles
-------------------------

* :ref:`A Simple Way to Run Several Message Processors in Parallel <dev-cookbook-system-mq-simple-run-parallel>`
* :ref:`Flow to Run Parallel Jobs via Creating a Root Job and Child Jobs <dev-cookbook-system-mq-run-root-child-jobs>`
* :ref:`Run Only a Single Job in Message Processor <dev-cookbook-system-mq-run-single-job>`
* :ref:`Run the Job That Has Two or More Steps <dev-cookbook-system-mq-run-two-steps-job>`
