.. _upgrade_psmdb:

================================================================================
Upgrading |PSMDB|
================================================================================

An in-place upgrade is done by keeping the existing data in the server. It involves changing out the MongoDB binaries. Generally speaking, the upgrade steps include:

1. Stopping the ``mongod`` service 
#. Removing the old binaries
#. Installing the new server version binaries 
#. Restarting the ``mongod`` service with the same ``dbpath`` data directory. 
  
An in-place upgrade is suitable for most environments except the ones that use ephemeral storage and/or host addresses. 

This document provides upgrade instructions for the following use cases:

* :ref:`upgrade_from_mongodb`;
* :ref:`minor_upgrade`


.. _upgrade_from_mongodb:

Upgrading from |mongodb-ce|
================================================================================

.. note::

   MongoDB creates a user that belongs to two groups, which is a potential
   security risk.  This is fixed in |PSMDB|: the user is included only in the
   ``mongod`` group.  To avoid problems with current MongoDB setups, existing
   user group membership is not changed when you migrate to |PSMDB|.  Instead, a
   new ``mongod`` user is created during installation, and it belongs to the
   ``mongod`` group.

This section describes an in-place upgrade of a ``mongod`` instance. If you are using data at rest encryption, refer to the :ref:`upgrade_encryption` section. 

.. contents::
   :local: 

Prerequisites
-------------

Before you start the upgrade, update the |mongodb| configuration file
(:file:`/etc/mongod.conf`) to contain the following settings.  

.. code-block:: yaml

   processManagement:
      fork: true
      pidFilePath: /var/run/mongod.pid

Troubleshooting tip: The ``pidFilePath`` setting in :file:`mongod.conf` must  match the ``PIDFile`` option in the ``systemd mongod`` service unit. Otherwise, the service will kill the ``mongod`` process after a timeout.

.. warning::

   Before starting the upgrade, we recommend to perform a full
   backup of your data.  

.. tabs::

   .. tab:: Upgrading on Debian or Ubuntu

      
      1. Stop the ``mongod`` service: 

         .. code-block:: bash

            $ sudo systemctl stop mongod

      #. Check for installed packages: 
         
         .. code-block:: bash

            $ sudo dpkg -l | grep mongod

         .. admonition:: Output

            .. code-block:: text

               ii  mongodb-org            4.4.0    amd64      MongoDB document-oriented database system (metapackage)
               ii  mongodb-org-mongos     4.4.0    amd64      MongoDB sharded cluster query router
               ii  mongodb-org-server     4.4.0    amd64      MongoDB database server
               ii  mongodb-org-shell      4.4.0    amd64      MongoDB shell client
               ii  mongodb-org-tools      4.4.0    amd64      MongoDB tools

      #. Remove the installed packages:

         .. code-block:: bash

            $ apt remove mongodb-org mongodb-org-mongos mongodb-org-server \
            $ mongodb-org-shell mongodb-org-tools


      #. Remove log files: 

         .. code-block:: bash

            $ sudo rm -r /var/log/mongodb

      #. Install |PSMDB| :ref:`using apt <apt>`.

      #. Verify that the configuration file includes the correct options. For example, |PSMDB| stores data files in :file:`/var/lib/mongodb` by default. If you used another ``dbPath`` data directory, edit the configuration file accordingly
         
      #. Start the ``mongod`` service: 

         .. code-block:: bash

            $ sudo systemctl mongod start

   .. tab:: Upgrading on Red Hat Enterprise Linux or CentOS

      
      1. Stop the ``mongod`` service: 
         
         .. code-block:: bash

            $ sudo systemctl stop mongod

      #. Check for installed packages: 
         
         .. code-block:: bash
         
            $ sudo rpm -qa | grep mongo

         .. admonition:: Output

            .. code-block:: text

               mongodb-org-mongos-4.4.0-1.el6.x86_64
               mongodb-org-shell-4.4.0-1.el6.x86_64
               mongodb-org-server-4.4.0-1.el6.x86_64
               mongodb-org-tools-4.4.0-1.el6.x86_64
               mongodb-org-4.4.0-1.el6.x86_64

      #. Remove the installed packages:

         .. code-block:: bash

            $ yum remove \
            mongodb-org-mongos-4.4.0-1.el6.x86_64 \
            mongodb-org-shell-4.4.0-1.el6.x86_64 \
            mongodb-org-server-4.4.0-1.el6.x86_64 \
            mongodb-org-tools-4.4.0-1.el6.x86_64 \
            mongodb-org-4.4.0-1.el6.x86_64

      #. Remove log files: 

         .. code-block:: bash

            $ sudo rm -r /var/log/mongodb

      #. Install Percona Server for MongoDB :ref:`using yum <yum>`.

      .. note::

         When you remove old packages, your existing configuration file is saved as
         :file:`/etc/mongod.conf.rpmsave`.  If you want to use this configuration with
         the new version, replace the default :file:`/etc/mongod.conf` file.  For
         example, existing data may not be compatible with the default WiredTiger
         storage engine.

      # Start the ``mongod`` service:

        .. code-block:: bash

            $ sudo systemctl start mongod


To upgrade a replica set or a sharded cluster, use the :term:`rolling restart <Rolling restart>` method. It allows you to perform the upgrade with minimum downtime. You upgrade the nodes one by one, while the whole cluster / replica set remains operational. 

.. seealso::

   |mongodb| Documentation: 
      - `Upgrade a Replica Set <https://docs.mongodb.com/manual/release-notes/4.4-upgrade-replica-set/>`_
      - `Upgrade a Sharded Cluster <https://docs.mongodb.com/manual/release-notes/4.4-upgrade-sharded-cluster/>`_

.. _minor_upgrade:

Minor upgrade of |PSMDB|
========================

To upgrade |PSMDB| to the latest version, follow these steps:

1. Stop the `mongod` service:
   
   .. code-block:: bash

      $ sudo systemctl stop mongod

#. Install the latest version packages. Use the command relevant to your operating system.

   .. tabs:: 

      .. tab:: On Debian and Ubuntu:
      
         .. code-block:: bash

            $ sudo apt install percona-server-mongodb

      .. tab:: On Red Hat Enterprise Linux or CentOS:
      
         .. code-block:: bash

            $ sudo yum install percona-server-mongodb

#. Start the `mongod` service:
   
   .. code-block:: bash

      $ sudo systemctl start mongod

To upgrade a replica set or a sharded cluster, use the :term:`rolling restart <Rolling restart>` method. It allows you to perform the upgrade with minimum downtime. You upgrade the nodes one by one, while the whole cluster / replica set remains operational.


.. _upgrade_encryption:

Upgrading to |PSMDB| with data at rest encryption enabled
=========================================================

Steps to upgrade from |mongodb-ce| with data encryption enabled to |PSMDB| are different. ``mongod`` requires an empty ``dbPath`` data directory because it cannot encrypt data files in place. It must receive data from other replica set members during the initial sync. Please refer to the :ref:`switch_storage_engines` for more information on migration of encrypted data. `Contact us <https://www.percona.com/about-percona/contact#us>`_ for working at the detailed migration steps, if further assistance is needed.


.. include:: ../.res/replace.txt
.. include:: ../.res/replace.program.txt
