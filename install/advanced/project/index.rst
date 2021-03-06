.. _geonode-project:

===============
GeoNode Project
===============

Overview
========

The following steps will guide you to a new setup of GeoNode Project. All guides will first install and configure the system to run it in ``DEBUG`` mode (also known as ``DEVELOPMENT`` mode) and then by configuring an HTTPD server to serve GeoNode through the standard ``HTTP`` (``80``) port.

Those guides **are not** meant to be used on a production system. There will be dedicated chapters that will show you some *hints* to optimize GeoNode for a production-ready machine. In any case, we strongly suggest to task an experienced *DevOp* or *System Administrator* before exposing your server to the ``WEB``.

Ubuntu 18.04
============

This part of the documentation describes the complete setup process for GeoNode on an Ubuntu 18.04 64-bit clean environment (Desktop or Server). All examples use shell commands that you must enter on a local terminal or a remote shell.
- If you have a graphical desktop environment you can open the terminal application after login;
- if you are working on a remote server the provider or sysadmin should has given you access through an ssh client.

.. _install_dep_proj:

Install the dependencies
^^^^^^^^^^^^^^^^^^^^^^^^

In this section, we are going to install all the basic packages and tools needed for a complete GeoNode installation. To follow this guide, a piece of basic knowledge about Ubuntu Server configuration and working with a shell is required. This guide uses ``vim`` as the editor; fill free to use ``nano``, ``gedit`` or others.

Upgrade system packages
.......................

Check that your system is already up-to-date with the repository running the following commands:

.. code-block:: shell

   sudo apt update
   sudo apt upgrade


Create a Dedicated User
.......................

In the following steps a User named ``geonode`` is used: to run installation commands the user must be in the ``sudo`` group.

Create User ``geonode`` **if not present**:

.. code-block:: shell

  # Follow the prompts to set the new user's information.
  # It is fine to accept the defaults to leave all of this information blank.
  sudo adduser geonode

  # The following command adds the user geonode to group sudo
  sudo usermod -aG sudo geonode

  # make sure the newly created user is allowed to login by ssh
  # (out of the scope of this documentation) and switch to User geonode
  su geonode

Packages Installation
.....................

.. note:: You don't need to install the **system packages** if you want to run the project using Docker

First, we are going to install all the **system packages** needed for the GeoNode setup.

.. code-block:: shell

  # Install packages from GeoNode core
  sudo apt install -y gdal-bin
  sudo apt install -y python3-pip python3-dev python3-virtualenv python3-venv virtualenvwrapper
  sudo apt install -y libxml2 libxml2-dev gettext
  sudo apt install -y libxslt1-dev libjpeg-dev libpng-dev libpq-dev libgdal-dev
  sudo apt install -y software-properties-common build-essential
  sudo apt install -y git unzip gcc zlib1g-dev libgeos-dev libproj-dev
  sudo apt install -y sqlite3 spatialite-bin libsqlite3-mod-spatialite

  # If the following does not work, you can skip it
  sudo apt install -y libgdal20

  # Install Openjdk
  sudo -i apt update
  sudo apt install openjdk-8-jdk-headless default-jdk-headless -y
  sudo update-java-alternatives --jre-headless --jre --set java-1.8.0-openjdk-amd64

  sudo apt update -y
  sudo apt upgrade -y
  sudo apt autoremove -y
  sudo apt autoclean -y
  sudo apt purge -y
  sudo apt clean -y

  # Install Packages for Virtual environment management
  sudo apt install -y virtualenv virtualenvwrapper

  # Install text editor
  sudo apt install -y vim

Geonode Project Installation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Geonode project is the proper way to run a customized installation of Geonode. The repository of geonode-project contains a minimal set of files following the structure of a django-project. Geonode itself will be installed as a requirement of your project.
Inside the project structure is possible to extend, replace or modify all geonode componentse (e.g. css and other static files, templates, models..) and even register new django apps **without touching the original Geonode code**.


.. note:: You can call your geonode project whatever you like following the naming conventions for python packages (generally lower case with underscores (_). In the examples below, replace ``my_geonode`` with whatever you would like to name your project.

See also the `README <https://github.com/GeoNode/geonode-project/blob/master/README.md>`_ file on geonode-project repository

First of all we need to prepare a new Python Virtual Environment

Prepare the environment

.. code-block:: shell

  sudo mkdir -p /opt/geonode_custom/
  sudo usermod -a -G www-data geonode
  sudo chown -Rf geonode:www-data /opt/geonode_custom/
  sudo chmod -Rf 775 /opt/geonode_custom/

Clone the source code

.. code-block:: shell

  cd /opt/geonode_custom/
  git clone https://github.com/GeoNode/geonode-project.git

Make an instance out of the ``Django Template``

.. note:: We will call our instance ``my_geonode``. You can change the name at your convenience.

.. code-block:: shell

  vim ~/.bashrc
  # add the following line to the bottom
  source /usr/share/virtualenvwrapper/virtualenvwrapper.sh

.. code-block:: shell

  source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
  mkvirtualenv --python=/usr/bin/python3 my_geonode

  Alterantively you can also create the virtual env like below
  python3.8 -m venv /home/geonode/dev/.venvs/my_geonode
  source /home/geonode/dev/.venvs/my_geonode/bin/activate

  pip install Django==2.2.12

  django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile my_geonode

  # Install the Python packages
  cd /opt/geonode_custom/my_geonode
  pip install -r requirements.txt --upgrade --no-cache --no-cache-dir
  pip install -e . --upgrade

  # Install GDAL Utilities for Python
  pip install pygdal=="`gdal-config --version`.*"

  # Dev scripts
  mv .override_dev_env.sample .override_dev_env
  mv manage_dev.sh.sample manage_dev.sh
  mv paver_dev.sh.sample paver_dev.sh


Run GeoNode Project for the first time in DEBUG Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::

  Be sure you have successfully completed all the steps of the section :ref:`install_dep_proj`.

This command will run both GeoNode and GeoServer locally after having prepared the SQLite database. The server will start in ``DEBUG`` (or ``DEVELOPMENT``) mode, and it will start the following services:

#. GeoNode on ``http://localhost:8000/``
#. GeoServer on ``http://localhost:8080/geoserver/``

This modality is beneficial to debug issues and/or develop new features, but it cannot be used on a production system.

.. code-block:: shell

  # Prepare the GeoNode SQLite database (the first time only)
  ./paver_dev.sh setup
  ./paver_dev.sh sync

.. note::

  In case you want to start again from a clean situation, just run

  .. code:: shell

    ./paver_dev.sh reset_hard

.. warning:: This will blow up completely your ``local_settings``, delete the SQLlite database and remove the GeoServer data dir.

.. code-block:: shell

  # Run the server in DEBUG mode
  ./paver_dev.sh start

Once the server has finished the initialization and prints on the console the sentence ``GeoNode is now available.``, you can open a browser and go to::

  http://localhost:8000/

Sign-in with::

  user: admin
  password: admin

From now on, everything already said for GeoNode Core (please refer to the section :ref:`configure_dbs_core` and following), applies to a
GeoNode Project.

**Be careful** to use the **new** paths and names everywhere:

* Everytime you'll find the keyword ``geonode``, you'll need to use your geonode custom name instead (in this example ``my_geonode``).

* Everytime you'll find paths pointing to ``/opt/geonode/``, you'll need to update them to point to your custom project instead (in this example ``/opt/geonode_custom/my_geonode``).

Docker
======

.. warning:: Before moving with this section, you should have read and clearly understood the ``INSTALLATION > GeoNode Core`` sections, and in particular the ``Docker`` one. Everything said for the GeoNode Core Vanilla applies here too, except that the Docker container names will be slightly different. As an instance if you named your project ``my_geonode``, your containers will be called:

  .. code-block:: shell

    'django4my_geonode' instead of 'django4geonode' and so on...

Deploy an instance of a geonode-project Django template 3.0 with Docker on localhost
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prepare the environment

.. code-block:: shell

  sudo mkdir -p /opt/geonode_custom/
  sudo usermod -a -G www-data geonode
  sudo chown -Rf geonode:www-data /opt/geonode_custom/
  sudo chmod -Rf 775 /opt/geonode_custom/

Clone the source code

.. code-block:: shell

  cd /opt/geonode_custom/
  git clone https://github.com/GeoNode/geonode-project.git

Make an instance out of the ``Django Template``

.. note:: We will call our instance ``my_geonode``. You can change the name at your convenience.

.. code-block:: shell

  source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
  mkvirtualenv --python=/usr/bin/python3 my_geonode

  Alterantively you can also create the virtual env like below
  python3.8 -m venv /home/geonode/dev/.venvs/my_geonode
  source /home/geonode/dev/.venvs/my_geonode/bin/activate

  pip install Django==2.2.12

  django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile my_geonode
  cd /opt/geonode_custom/my_geonode

Modify the code and the templates and rebuild the Docker Containers

.. code-block:: shell

  docker-compose -f docker-compose.yml build --no-cache

Finally, run the containers

.. code-block:: shell

  docker-compose -f docker-compose.yml up -d

Deploy an instance of a geonode-project Django template 3.0 with Docker on a domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: We will use ``www.example.org`` as an example. You can change the name at your convenience.

Stop the containers

.. code-block:: shell

  cd /opt/geonode_custom/my_geonode

  docker-compose -f docker-compose.yml stop

Edit the ``ENV`` override file in order to deploy on ``www.example.org``

Replace everywhere ``localhost`` with ``www.example.org``

.. code-block:: shell

  vim .env

.. code-block:: shell

  # e.g.: :%s/localhost/www.example.org/g

.. note:: It is possible to override here even more variables to customize the GeoNode instance. See the ``GeoNode Settings`` section in order to get a list of the available options.

Run the containers in daemon mode

.. code-block:: shell

  docker-compose -f docker-compose.yml -f docker-compose.override.example-org.yml up --build -d

