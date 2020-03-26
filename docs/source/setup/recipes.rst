.. _recipes:

VOLTTRON Deployment Recipes
===========================

Beginning with version 7.0, VOLTTRON introduces the concept of recpies.  This system leverages
`ansible <https://docs.ansible.com/ansible/latest/index.html>`_ to orchestrate the deployment and
configuration process for both the VOLTTRON platform, and installed agents, on remote systems.

The following sections describe the recipes which are currently available individually. At the end
we go through an exmaple of deploying three platforms, each on a local virtual machine configured
using `vagrant <https://www.vagrantup.com/docs/index.html>`_. All of the configuration files
required for the example are distributed with the VOLTTRON source repository.

Recpies component descriptions
------------------------------


Getting Started
~~~~~~~~~~~~~~~~

The recipes system is designed to be executed from a user workstation or other server with ssh
access to the hosts which will be running the VOLTTRON platforms being configured. In order to do
so, you require a python environment with both ansible and the core VOLTTRON components installed.
To achieve that, use the :ref:`Bootstrap-Options` and include the ``--recipes`` option.

Platform and angent configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



Available recipes
~~~~~~~~~~~~~~~~~

.. TODO A sub sub sub section for each of the recpies

.. TODO host config

.. TODO init

.. TODO up

.. TODO down

.. TODO status

.. TODO destroy

Recipes deployment example
--------------------------

.. TODO: Here I should walk through the vagrant-based example
