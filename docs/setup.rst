.. _setup:

Setup
=====

Paperless isn't a very complicated app, but there are a few components, so some
basic documentation is in order.  If you follow along in this document and
still have trouble, please open an `issue on GitHub`_ so I can fill in the
gaps.

.. _issue on GitHub: https://github.com/danielquinn/paperless/issues


.. _setup-download:

Download
--------

The source is currently only available via GitHub, so grab it from there,
either by using ``git``:

.. code:: bash

    $ git clone https://github.com/danielquinn/paperless.git
    $ cd paperless

or just download the tarball and go that route:

.. code:: bash

    $ cd to the directory where you want to run Paperless
    $ wget https://github.com/danielquinn/paperless/archive/master.zip
    $ unzip master.zip
    $ cd paperless-master


.. _setup-installation:

Installation & Configuration
----------------------------

You can go multiple routes with setting up and running Paperless:

 * The `bare metal route`_
 * The `vagrant route`_
 * The `docker route`_


The `Vagrant route`_ is quick & easy, but means you're running a VM which comes
with memory consumption, cpu overhead etc. The `docker route`_ offers the same
simplicity as Vagrant with lower resource consumption.

The `bare metal route`_ is a bit more complicated to setup but makes it easier
should you want to contribute some code back.

.. _Vagrant route: setup-installation-vagrant_
.. _docker route: setup-installation-docker_
.. _bare metal route: setup-installation-bare-metal_
.. _Docker Machine: https://docs.docker.com/machine/


.. _setup-installation-bare-metal:

Standard (Bare Metal)
+++++++++++++++++++++

1. Install the requirements as per the :ref:`requirements <requirements>` page.
2. Within the extract of master.zip go to the ``src`` directory.
3. Copy ``../paperless.conf.example`` to ``/etc/paperless.conf`` and open it in
   your favourite editor.  As this file contains passwords.  It should only be
   readable by user root and paperless!  Set the values for:

   Set the values for:

    * ``PAPERLESS_CONSUMPTION_DIR``: this is where your documents will be
      dumped to be consumed by Paperless.
    * ``PAPERLESS_OCR_THREADS``: this is the number of threads the OCR process
      will spawn to process document pages in parallel.
    * ``PAPERLESS_PASSPHRASE``: this is only required if you want to use GPG to
      encrypt your document files.  This is the passphrase Paperless uses to
      encrypt/decrypt the original documents.  Don't worry about defining this
      if you don't want to use encryption (the default).

4. Initialise the SQLite database with ``./manage.py migrate``.
5. Create a user for your Paperless instance with
   ``./manage.py createsuperuser``. Follow the prompts to create your user.
6. Start the webserver with ``./manage.py runserver <IP>:<PORT>``.
   If no specifc IP or port are given, the default is ``127.0.0.1:8000``
   also known as http://localhost:8000/.
   You should now be able to visit your (empty) installation at
   `Paperless webserver`_ or whatever you chose before.  You can login with the
   user/pass you created in #5.

7. In a separate window, change to the ``src`` directory in this repo again,
   but this time, you should start the consumer script with
   ``./manage.py document_consumer``.
8. Scan something or put a file into the  ``CONSUMPTION_DIR``.
9. Wait a few minutes
10. Visit the document list on your webserver, and it should be there, indexed
    and downloadable.

.. caution::

    This installation is not secure. Once everything is working head over to
    `Making things more permanent`_

.. _Paperless webserver: http://127.0.0.1:8000
.. _Making things more permanent: setup-permanent_

.. _setup-installation-docker:

Docker Method
+++++++++++++

1. Install `Docker`_.

   .. caution::

      As mentioned earlier, this guide assumes that you use Docker natively
      under Linux. If you are using `Docker Machine`_ under Mac OS X or
      Windows, you will have to adapt IP addresses, volume-mounting, command
      execution and maybe more.

2. Install `docker-compose`_. [#compose]_

   .. caution::

       If you want to use the included ``docker-compose.yml.example`` file, you
       need to have at least Docker version **1.10.0** and docker-compose
       version **1.6.0**.

       See the `Docker installation guide`_ on how to install the current
       version of Docker for your operating system or Linux distribution of
       choice. To get an up-to-date version of docker-compose, follow the
       `docker-compose installation guide`_ if your package repository doesn't
       include it.

       .. _Docker installation guide: https://docs.docker.com/engine/installation/
       .. _docker-compose installation guide: https://docs.docker.com/compose/install/

3. Create a copy of ``docker-compose.yml.example`` as ``docker-compose.yml``
   and a copy of ``docker-compose.env.example`` as ``docker-compose.env``.
   You'll be editing both these files: taking a copy ensures that you can
   ``git pull`` to receive updates without risking merge conflicts with your
   modified versions of the configuration files.
4. Modify ``docker-compose.yml`` to your preferences, following the
   instructions in comments in the file. The only change that is a hard
   requirement is to specify where the consumption directory should
   mount.[#dockercomposeyml]_
5. Modify ``docker-compose.env`` and adapt the following environment variables:

   ``PAPERLESS_PASSPHRASE``
     This is the passphrase Paperless uses to encrypt/decrypt the original
     document.  If you aren't planning on using GPG encryption, you can just
     leave this undefined.

   ``PAPERLESS_OCR_THREADS``
     This is the number of threads the OCR process will spawn to process
     document pages in parallel. If the variable is not set, Python determines
     the core-count of your CPU and uses that value.

   ``PAPERLESS_OCR_LANGUAGES``
     If you want the OCR to recognize other languages in addition to the
     default English, set this parameter to a space separated list of
     three-letter language-codes after `ISO 639-2/T`_. For a list of available
     languages -- including their three letter codes -- see the
     `Alpine packagelist`_.

   ``USERMAP_UID`` and ``USERMAP_GID``
     If you want to mount the consumption volume (directory ``/consume`` within
     the containers) to a host-directory -- which you probably want to do --
     access rights might be an issue. The default user and group ``paperless``
     in the containers have an id of 1000. The containers will enforce that the
     owning group of the consumption directory will be ``paperless`` to be able
     to delete consumed documents. If your host-system has a group with an ID
     of 1000 and you don't want this group to have access rights to the
     consumption directory, you can use ``USERMAP_GID`` to change the id in the
     container and thus the one of the consumption directory. Furthermore, you
     can change the id of the default user as well using ``USERMAP_UID``.

6. Run ``docker-compose up -d``. This will create and start the necessary
   containers.
7. To be able to login, you will need a super user. To create it, execute the
   following command:

   .. code-block:: shell-session

       $ docker-compose run --rm webserver createsuperuser

   This will prompt you to set a username (default ``paperless``), an optional
   e-mail address and finally a password.
8. The default ``docker-compose.yml`` exports the webserver on your local port
   8000. If you haven't adapted this, you should now be able to visit your
   `Paperless webserver`_ at ``http://127.0.0.1:8000``. You can login with the
   user and password you just created.
9. Add files to consumption directory the way you prefer to. Following are two
   possible options:

   1. Mount the consumption directory to a local host path by modifying your
      ``docker-compose.yml``:

      .. code-block:: diff

         diff --git a/docker-compose.yml b/docker-compose.yml
         --- a/docker-compose.yml
         +++ b/docker-compose.yml
         @@ -17,9 +18,8 @@ services:
                  volumes:
                      - paperless-data:/usr/src/paperless/data
                      - paperless-media:/usr/src/paperless/media
         -            - /consume
         +            - /local/path/you/choose:/consume

      .. danger::

          While the consumption container will ensure at startup that it can
          **delete** a consumed file from a host-mounted directory, it might
          not be able to **read** the document in the first place if the access
          rights to the file are incorrect.

          Make sure that the documents you put into the consumption directory
          will either be readable by everyone (``chmod o+r file.pdf``) or
          readable by the default user or group id 1000 (or the one you have
          set with ``USERMAP_UID`` or ``USERMAP_GID`` respectively).

   2. Use ``docker cp`` to copy your files directly into the container:

      .. code-block:: shell-session

         $ # Identify your containers
         $ docker-compose ps
                 Name                       Command                State     Ports
         -------------------------------------------------------------------------
         paperless_consumer_1    /sbin/docker-entrypoint.sh ...   Exit 0
         paperless_webserver_1   /sbin/docker-entrypoint.sh ...   Exit 0

         $ docker cp /path/to/your/file.pdf paperless_consumer_1:/consume

      ``docker cp`` is a one-shot-command, just like ``cp``. This means that
      every time you want to consume a new document, you will have to execute
      ``docker cp`` again. You can of course automate this process, but option
      1 is generally the preferred one.

      .. danger::

          ``docker cp`` will change the owning user and group of a copied file
          to the acting user at the destination, which will be ``root``.

          You therefore need to ensure that the documents you want to copy into
          the container are readable by everyone (``chmod o+r file.pdf``)
          before copying them.


.. _Docker: https://www.docker.com/
.. _docker-compose: https://docs.docker.com/compose/install/
.. _ISO 639-2/T: https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
.. _Alpine packagelist: https://pkgs.alpinelinux.org/packages?name=tesseract-ocr-data*&arch=x86_64

.. [#compose] You of course don't have to use docker-compose, but it
   simplifies deployment immensely. If you know your way around Docker, feel
   free to tinker around without using compose!

.. [#dockercomposeyml] If you're upgrading your docker-compose images from
   version 1.1.0 or earlier, you might need to change in the
   ``docker-compose.yml`` file the ``image: pitkley/paperless`` directive in
   both the ``webserver`` and ``consumer`` sections to ``build: ./`` as per the
   newer ``docker-compose.yml.example`` file


.. _setup-installation-vagrant:

Vagrant Method
++++++++++++++

1. Install `Vagrant`_.  How you do that is really between you and your OS.
2. Run ``vagrant up``.  An instance will start up for you.  When it's ready and
   provisioned...
3. Run ``vagrant ssh`` and once inside your new vagrant box, edit
   ``/etc/paperless.conf`` and set the values for:

    * ``PAPERLESS_CONSUMPTION_DIR``: This is where your documents will be
      dumped to be consumed by Paperless.
    * ``PAPERLESS_PASSPHRASE``: This is the passphrase Paperless uses to
      encrypt/decrypt the original document.  It's only required if you want
      your original files to be encrypted, otherwise, just leave it unset.
    * ``PAPERLESS_EMAIL_SECRET``: this is the "magic word" used when consuming
      documents from mail or via the API.  If you don't use either, leaving it
      blank is just fine.

4. Exit the vagrant box and re-enter it with ``vagrant ssh`` again.  This
   updates the environment to make use of the changes you made to the config
   file.
5. Initialise the database with ``/opt/paperless/src/manage.py migrate``.
6. Still inside your vagrant box, create a user for your Paperless instance
   with ``/opt/paperless/src/manage.py createsuperuser``. Follow the prompts to
   create your user.
7. Start the webserver with
   ``/opt/paperless/src/manage.py runserver 0.0.0.0:8000``. You should now be
   able to visit your (empty) `Paperless webserver`_ at ``172.28.128.4:8000``.
   You can login with the user/pass you created in #6.
8. In a separate window, run ``vagrant ssh`` again, but this time once inside
   your vagrant instance, you should start the consumer script with
   ``/opt/paperless/src/manage.py document_consumer``.
9. Scan something.  Put it in the ``CONSUMPTION_DIR``.
10. Wait a few minutes
11. Visit the document list on your webserver, and it should be there, indexed
    and downloadable.

.. caution::

    This installation is not secure. Once everything is working head up to
    `Making things more permanent`_

.. _Vagrant: https://vagrantup.com/
.. _Paperless server: http://172.28.128.4:8000


.. _setup-permanent:

Making Things a Little more Permanent
-------------------------------------

Once you've tested things and are happy with the work flow, you should secure
the installation and automate the process of starting the webserver and
consumer.


.. _setup-permanent-webserver:

Using a Real Webserver
++++++++++++++++++++++

The default is to use Django's development server, as that's easy and does the
job well enough on a home network. However it is heavily discouraged to use
it for more than that.

If you want to do things right you should use a real webserver capable of
handling more than one thread. You will also have to let the webserver serve
the static files (CSS, JavaScript) from the directory configured in
``PAPERLESS_STATICDIR``.  The default static files directory is ``../static``.

For that you need to activate your virtual environment and collect the static
files with the command:

.. code:: bash

    $ cd <paperless directory>/src
    $ ./manage.py collectstatic


Apache
~~~~~~

This is a configuration supplied by `steckerhalter`_ on GitHub.  It uses Apache
and mod_wsgi, with a Paperless installation in ``/home/paperless/``:

.. code:: apache

    <VirtualHost *:80>
        ServerName example.com

        Alias /static/ /home/paperless/paperless/static/
        <Directory /home/paperless/paperless/static>
            Require all granted
        </Directory>

        WSGIScriptAlias / /home/paperless/paperless/src/paperless/wsgi.py
        WSGIDaemonProcess example.com user=paperless group=paperless threads=5 python-path=/home/paperless/paperless/src:/home/paperless/.env/lib/python3.4/site-packages
        WSGIProcessGroup example.com

        <Directory /home/paperless/paperless/src/paperless>
            <Files wsgi.py>
                Require all granted
            </Files>
        </Directory>
    </VirtualHost>

.. _steckerhalter: https://github.com/steckerhalter


Nginx + Gunicorn
~~~~~~~~~~~~~~~~

If you're using Nginx, the most common setup is to combine it with a
Python-based server like Gunicorn so that Nginx is acting as a proxy.  Below is
a copy of a simple Nginx configuration fragment making use of a gunicorn
instance listening on localhost port 8000.

.. code:: nginx

    server {
        listen 80;

        index index.html index.htm index.php;
        access_log /var/log/nginx/paperless_access.log;
        error_log /var/log/nginx/paperless_error.log;

        location /static {

            autoindex on;
            alias <path-to-paperless-static-directory>

        }

        location / {

            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_pass http://127.0.0.1:8000
        }
    }


The gunicorn server can be started with the command:

.. code-block:: shell

    $ <path-to-paperless-virtual-environment>/bin/gunicorn <path-to-paperless>/src/paperless.wsgi -w 2


.. _setup-permanent-standard-systemd:

Standard (Bare Metal + Systemd)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you're running on a bare metal system that's using Systemd, you can use the
service unit files in the ``scripts`` directory to set this up.

1. You'll need to create a group and user called ``paperless`` (without login)
2. Setup Paperless to be in a place that this new user can read and write to.
3. Ensure ``/etc/paperless`` is readable by the ``paperless`` user.
4. Copy the service file from the ``scripts`` directory to
   ``/etc/systemd/system``.

.. code-block:: bash

    $ cp /path/to/paperless/scripts/paperless-consumer.service /etc/systemd/system/
    $ cp /path/to/paperless/scripts/paperless-webserver.service /etc/systemd/system/

5. Edit the service file to point the ``ExecStart`` line to the proper location
   of your paperless install, referencing the appropriate Python binary. For
   example:
   ``ExecStart=/path/to/python3 /path/to/paperless/src/manage.py document_consumer``.
6. Start and enable (so they start on boot) the services.

.. code-block:: bash

    $ systemctl enable paperless-consumer
    $ systemctl enable paperless-webserver
    $ systemctl start paperless-consumer
    $ systemctl start paperless-webserver


.. _setup-permanent-standard-upstart:

Standard (Bare Metal + Upstart)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ubuntu 14.04 and earlier use the `Upstart`_ init system to start services
during the boot process. To configure Upstart to run Paperless automatically
after restarting your system:

1. Change to the directory where Upstart's configuration files are kept:
   ``cd /etc/init``
2. Create a new file: ``sudo nano paperless-server.conf``
3. In the newly-created file enter::

    start on (local-filesystems and net-device-up IFACE=eth0)
    stop on shutdown

    respawn
    respawn limit 10 5

    script
      exec <path to paperless virtual environment>/bin/gunicorn <path to parperless>/src/paperless.wsgi -w 2
    end script

   Note that you'll need to replace ``/srv/paperless/src/manage.py`` with the
   path to the ``manage.py`` script in your installation directory.

  If you are using a network interface other than ``eth0``, you will have to
  change ``IFACE=eth0``. For example, if you are connected via WiFi, you will
  likely need to replace ``eth0`` above with ``wlan0``. To see all interfaces,
  run ``ifconfig -a``.

  Save the file.

4. Create a new file: ``sudo nano paperless-consumer.conf``

5. In the newly-created file enter::

    start on (local-filesystems and net-device-up IFACE=eth0)
    stop on shutdown

    respawn
    respawn limit 10 5

    script
      exec <path to paperless virtual environment>/bin/python <path to parperless>/manage.py document_consumer
    end script

  Replace the path placeholder and ``eth0`` with the appropriate value and save the file.

These two configuration files together will start both the Paperless webserver
and document consumer processes when the file system and network interface
specified is available after boot. Furthermore, if either process ever exits
unexpectedly, Upstart will try to restart it a maximum of 10 times within a 5
second period.

.. _Upstart: http://upstart.ubuntu.com/


Vagrant
~~~~~~~

You may use the Ubuntu explanation above. Replace
``(local-filesystems and net-device-up IFACE=eth0)`` with ``vagrant-mounted``.


.. _setup-permanent-docker:

Docker
~~~~~~

If you're using Docker, you can set a restart-policy_ in the
``docker-compose.yml`` to have the containers automatically start with the
Docker daemon.

.. _restart-policy: https://docs.docker.com/engine/reference/commandline/run/#restart-policies-restart
