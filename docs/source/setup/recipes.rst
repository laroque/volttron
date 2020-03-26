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

Each of the available recipes is based around a single ansible playbook. The host configuration
recipe requires administrative access on the remote machines and must be executed using the ansible
command-line interface. All others, like volttron itself, execute exclusively in user-space and have
been integrated with the ``vctl`` command-line tool's ``deploy`` subcommand group.

As with any ansible-configured system, the recipies bring together a playbook, which detail a
sequence of tasks to be completed on each remote in order to achieve the desired state, with an
inventory describing various details about each system and defining that desired state.


Getting Started
~~~~~~~~~~~~~~~~

The recipes system is designed to be executed from a user workstation or other server with ssh
access to the hosts which will be running the VOLTTRON platforms being configured. In order to do
so, you require a python environment with both ansible and the core VOLTTRON components installed.
To achieve that, use the :ref:`Bootstrap-Options` and include the ``--recipes`` option. This will
install both the ``vctl`` command-line tool, as well as the ansible package with its command-line
tools.


Platform and angent configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first component of an ansible configuration is the inventory. This contains the information
about what hosts exist to be configured and allows various details of that configuration to be
selected. All VOLTTRON recipes use a standard ansible inventory file, which respects the standard
ansible rules allowing options to be applied globally, specified for a particular subset of systems,
or to be applied to a specific host. The inventory file includes options, described in the
:ref:`inventory-variables` table below, which configure the volttron platform itself on each host.


.. glossary::

   volttron_git_organization (string [volttron])
     the github organization to use when cloning onto remotes

   volttron_git_repo (string [volttron])
     the github repo name within the volttron_git_repo

   volttron_git_tag (string [master])
     git tag within the volttron_git_repo

   volttron_home (string-path [~/.volttron])
     directory path on the remote for the VOLTTRON_HOME

   volttron_root (string-path [~/volttron])
     directory path on the remote for the VOLTTRON_ROOT

   host_config (string-path [~/host.config.yml])
     path on the remote host where the platform configuration will be placed

   host_configs_dir (string-path [~/configs])
     path on the remote host where the agent configurations dir will be placed

   message_bus ((rmq or zmq) [zmq])
     selects the RabbitMQ or ZeroMQ message bus for the platform

   enable_web (bool [false])
     toggle web features in the platform and include dependencies when bootstrapping

   enable_drivers (bool [false])
     toggle installation of the driver capabilities when bootstrapping

   extra_requirements (list of strings [ [] ])
     list of extra packages to pip install into the platforms virtualenvironment

   http_proxy (string [''])
     http proxy to use on remote when using pip to install extra packages

   my_ansible_virtualenv (string-path [~/ansible_venv])
     path on remotes where a venv will be created with dependencies for remote task execution

   my_ansible_python_interpreter (string-path [~/ansible_venv/bin/python])
     path to the python interpreter to be used when executing volttron_instance module

   deployment_config_root (string-path [<inventory_dir>])
     local-system path to the directory containing configurations for all hosts

   deployment_host_config_dir (string-path [<deployment_config_root>/<inventory_hostname>])
     local-system path to the directory containing configuration for the specific remote host

   deployment_platform_config_dir (string-path [<deployment_host_config_dir>/configs])
     local-system path to the agent configurations directory specific to this remote host

   deployment_platform_config_file (string-path [<deployment_host_config_dir>/<inventory_hostname>.yml])
     local-system path to platform configuration file for this node
.. end of glossary of valid inventory varaibles


In addition to the platform configuration for each host, the recipes system allows for the agents
within each platform to be configured based on the contents a configurations directory. That
directory must have a configuration file for the platform, listing the complete set of agents, as
well as a supporting directory structure which may contain additional files used to configure those
agents.


.. TODO describe the agents-list file

.. TODO describe the associated agent dir files

Available recipes
~~~~~~~~~~~~~~~~~

Each recipe is based around an ansible playbook. With the exception of host configuration, recipes
may be executed by using a subcommand of ``vctl deploy`` as indicated in each of the following
paragraphs. In all cases the standard ``--help`` option may be used to print full set of required
and optional arguments and flags.


Host Configuration
^^^^^^^^^^^^^^^^^^

The host configuration recipe installs required system packages, as well as any other system
configuration changes required on the host. Currently this may include adding additional
repositories to the package manager, as well as creating directories in /etc/ansible with write
permissions enabled for the VOLTTRON user, used for customizing ansible facts. This recipe requires administrative access and must be run using the ansible commandline interface.

Assuming that your working directory is one level above the ``$VOLTTRON_ROOT`` directory, and that
your inventory file is in a subdirectory at path ``$PLATFORM_INVENTORY/hosts.yml``, that command
would look like:

.. code-block:: bash

  ansible-playbook -i $PLATFORM_INVENTORY/hosts.yml -K $VOLTTRON_ROOT/deployment/playbooks/host-config.yml

Please review the ansible documentation for detailed explanation of the options or other available
options.


Platform Installation
^^^^^^^^^^^^^^^^^^^^^

The platform installation recipe is run with the subcommand named ``init`` (and playbook defined in
``install-platform.yml``). It clones the specified VOLLTRON version on the remote platform and then
executes the standard bootstrapping procedures, including extra options as indicated by the options in
the inventory file. The platform is also configured based on those options.


Agent Installation
^^^^^^^^^^^^^^^^^^

The agent installation recipe is run with the subcommand named ``up`` (and playbook defined in
``update-agents.yml``). For each platform, It copies the local agent configuration files and
directories specified in the inventory file and then attempts to install each of the listed agents
into the platform.


Stop Platform
^^^^^^^^^^^^^

The stop platform recipe is run with the subcommand named ``down`` (and playbook defined in
``stop-volttron.yml``). It connects to each remote system and stops the running platform.


Check Platform Status
^^^^^^^^^^^^^^^^^^^^^

The platform status recipe is run with the subcommand named ``status`` (and playbook in
``status.yml``). It connects to each remote platform and collects collects the current status
information, printing it as the final task.


Remove Deployed VOLTTRON Components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The destroy platform recipe is run with the subcommand named ``destroy`` (and playbook
``remove-volttron.yml``). It stops any running platform and then attempts to remove all of the
user-space files created by the other recipes. Note that this does *not* uninstall any system-level
packages which may have been added, nor does it remove extra package repositories.


Recipes deployment example
--------------------------

A set of example configuration files are available within the source repository, located at
``$VOLTTRON_ROOT/examples/deployment/``. There a ``Vagrantfile`` is provided which can be used to
provision three virtual machines for use in this example. There is also an inventory file
(``hosts.yml``) and a configuration directory for each of the platforms.

.. note::
   The network relationship between a base operating system and virtual machines provisioned via
   Vagrant can be sensitive to a number of details specific to the host sytem. The example makes
   assumptions about what local IP addresses are available and that the provider (typically
   VirtualBox) will configure the network so that the VMs may be reached via ssh to their IP
   addresses. If that is incorrect, you may need to either adjust the inventory file to specify
   different connection details, or your ssh client configuration. In either case, you will need to
   consult documentation for the particular tool you are attempting to configure. To ensure ansible
   is able to connect to VMs.

After starting the VMs (with ``vagrant up``), you can proceed with executing each of the recipes in
turn. Assuming your working directory is ``$VOLTTRON_ROOT/examples/deployment``, and that you've
activated a virtualenvironment bootstrapped to support recipes. Those steps would each look like the
following (where the output is not shown). You're encouraged to pause after running each to connect
to the VMs and observe the changes.

.. code-block:: bash

   # Install system-level dependencies for volttron
   ansible-playbook -i hosts.yml -K ../../deployment/playbooks/host-config.yml

   # Create the volttron virtual environment, install runtime dependencies and VOLTTRON, configure
   # platform on each VM
   vctl deploy int -i hosts.yml

   # Install and configure VOLTTRON agents
   vctl deploy up -i hosts.yml

   # Inspect the status of all deployed platforms
   vctl deply status -i hosts.yml

   # Shutdown all platforms
   vctl deploy down -i hosts.yml

   # Remove all userspace artifacts on all VMs
   vctl deploy destroy -i hosts.yml
