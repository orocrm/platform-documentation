:orphan:

Web Server Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

.. begin_web_server_configuration

The following sections contain recommended configuration for supported web server types and versions.

Apache 2.2
^^^^^^^^^^

.. code-block:: apache
    :linenos:

    <VirtualHost *:80>
        ServerName {$folder_name}.example.com

        DirectoryIndex app.php
        DocumentRoot [$folder_location]}/{$folder_name}/web
        <Directory  [$folder_location]}/{$folder_name}/web>
            # enable the .htaccess rewrites
            AllowOverride All
            Order allow,deny
            Allow from All
        </Directory>

        ErrorLog /var/log/apache2/{$folder_name}_error.log
        CustomLog /var/log/apache2/{$folder_name}_access.log combined
    </VirtualHost>

Apache 2.4
^^^^^^^^^^

.. code-block:: apache
    :linenos:

    <VirtualHost *:80>
        ServerName {$folder_name}.example.com

        DirectoryIndex app.php
        DocumentRoot [$folder_location]}/{$folder_name}/web
        <Directory  [$folder_location]}/{$folder_name}/web>
            # enable the .htaccess rewrites
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog /var/log/apache2/{$folder_name}_error.log
        CustomLog /var/log/apache2/{$folder_name}_access.log combined
    </VirtualHost>

.. note:: Please make sure mod_rewrite and mod_headers are enabled.

Nginx
^^^^^

.. code-block:: nginx
    :linenos:

    server {
        server_name {$folder_name}.example.com;
        root  [$folder_location]}/{$folder_name}/web;

        location / {
            # try to serve file directly, fallback to app.php
            try_files $uri /app.php$is_args$args;
        }

        location ~ ^/(app|app_dev|config|install)\.php(/|$) {
	    fastcgi_pass 127.0.0.1:9000;
	    # or
            # fastcgi_pass unix:/var/run/php/php7-fpm.sock;
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param HTTPS off;
        }

        error_log /var/log/nginx/{$folder_name}_error.log;
        access_log /var/log/nginx/{$folder_name}_access.log;
    }


.. caution::

    Make sure that the web server user has permissions for the ``log`` directory of the application.

    More details on the file permissions configuration are available
    `in the official Symfony documentation`_

.. _`in the official Symfony documentation`: http://symfony.com/doc/current/book/installation.html#book-installation-permissions