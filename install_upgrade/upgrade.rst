.. _upgrade:

Upgrading to the Newer Version
------------------------------

.. begin

To retrieve source code of a new version and upgrade your OroPlatform instance, please execute the following steps:

1. ``cd`` to the OroPlatform root folder and switch the application to the maintenance mode.

.. code-block:: bash

     cd /path/to/application
     sudo -uwww-data bin/console lexik:maintenance:lock --env=prod

2. Stop the cron tasks.

   .. code-block:: bash

       crontab -e -uwww-data

   .. note::

      The www-data may be replaced with the user under which your web server runs.

   Comment this line.

   .. code-block:: text
       :linenos:

       */1 * * * * /usr/bin/php /path/to/application/bin/console --env=prod oro:cron >> /dev/null

3. Stop all running consumers.

4. Create backups of your database and source code.

5. Pull the changes from the necessary tag (`3.1.2`) in the application repository (e.g. ``https://github.com/oroinc/platform-application.git``) or download the latest OroPlatform version from the `download section on the oroinc.com/oroplatform <https://oroinc.com/oroplatform/download>`_ , unpack archive and overwrite existing system files.

   .. note::

      If you have any customization or third party extensions installed, make sure that:
        - your changes to "src/AppKernel.php" file are merged to the new file.
        - your changes to "src/" folder are merged and it contains the custom files.
        - your changes to "composer.json" file are merged to the new file.
        - your changes to configuration files in "config/" folder are merged to the new files.

   .. code-block:: bash

       sudo -uwww-data git pull
       sudo -u your_user_for_admin_tasks php composer.phar update --prefer-dist --no-dev

6. Upgrade the composer dependency and set up the right owner to the retrieved files.

   .. code-block:: bash

       sudo -u your_user_for_admin_tasks php composer.phar update --prefer-dist --no-dev

7. Remove old caches.

   .. code-block:: bash

       sudo rm -rf var/cache/*
       sudo rm -rf public/js/*
       sudo rm -rf public/css/*

8. Upgrade the platform.

   .. code-block:: bash

       sudo -u www-data php bin/console oro:platform:update --env=prod --force

9. Remove the caches.

   .. code-block:: bash

       sudo -u www-data bin/console cache:clear --env=prod

   or, as alternative:

   .. code-block:: bash

       sudo rm -rf var/cache/*
       sudo -u www-data bin/console cache:warmup --env=prod

10. Enable cron.

    .. code-block:: bash

        crontab -e -uwww-data

    Uncomment this line.

    .. code-block:: text
        :linenos:

        */1 * * * * /usr/bin/php /path/to/application/bin/console --env=prod oro:cron >> /dev/null

11. Switch your application back to normal mode from the maintenance mode.

    .. code-block:: bash

        sudo -u www-data bin/console lexik:maintenance:unlock --env=prod

12. Run the consumer(s).

    .. code-block:: bash

        sudo -u www-data bin/console oro:message-queue:consume --env=prod


    .. note::

       If PHP bytecode cache tools (e.g. opcache) are used, PHP-FPM (or Apache web server) should be restarted after the upgrade to flush cached bytecode from the previous installation.
