Ansible Playbook for Kafka and ZooKeeper Deployment
This repository contains an Ansible playbook and a universal role designed to automate the deployment of a scalable Apache Kafka and ZooKeeper cluster. The automation is built to be idempotent, robust, and compatible with multiple Linux distributions.

Key Features
Automated Kafka & ZooKeeper Setup: Deploys a multi-node Kafka and ZooKeeper cluster from scratch.

Multi-OS Support: Automatically detects the target OS family (Debian, Red Hat, SUSE) and uses the correct package manager.

Version Compatibility Checks: Includes pre-flight checks to ensure that the chosen Kafka version is compatible with the target operating system, preventing failed deployments.

Idempotent: The playbook can be run multiple times without causing unintended changes, ensuring a consistent state.

Dynamic Configuration: Uses Jinja2 templates to dynamically generate configuration files (server.properties, zookeeper.properties) based on your inventory.

Service Management: Configures and manages systemd services for both Kafka and ZooKeeper, ensuring they start on boot.

Prerequisites
Before running this playbook, ensure you have the following:

Ansible Controller: A machine with Ansible installed (version 2.9 or newer).

Target Servers: One or more servers (bare metal or VMs) with a supported Linux distribution (e.g., Ubuntu, CentOS, SUSE).

SSH Access: The Ansible controller must have passwordless SSH access to all target servers.

Python: Python 3 must be installed on all target servers, as it is required by most Ansible modules.

How to Use
1. Configure the Inventory
First, define your servers in an inventory file (e.g., inventory.yml). You need to specify which hosts will run ZooKeeper and which will run Kafka. You also need to assign a unique ID to each node.

Example inventory.yml:

YAML

all:
  children:
    zookeeper_servers:
      hosts:
        kafka-node-0:
          ansible_host: 10.0.0.23
          zookeeper_myid: 1
        kafka-node-1:
          ansible_host: 10.0.0.26
          zookeeper_myid: 2
        kafka-node-2:
          ansible_host: 10.0.0.30
          zookeeper_myid: 3

    kafka_servers:
      hosts:
        kafka-node-0:
          ansible_host: 10.0.0.23
          kafka_broker_id: 0
        kafka-node-1:
          ansible_host: 10.0.0.26
          kafka_broker_id: 1
        kafka-node-2:
          ansible_host: 10.0.0.30
          kafka_broker_id: 2
  vars:
    ansible_user: "root"
2. Customize Variables (Optional)
You can change the Kafka version, installation directories, and other settings by editing the variables file at roles/kafka_universal/vars/main.yml.

3. Run the Playbook
Execute the main playbook to start the deployment. If you have an interactive playbook like kafka-playbook.yml, run that.

Bash

ansible-playbook -i inventory.yml kafka-playbook.yml
How It Works: The Automation Workflow
The deployment is handled by the kafka_universal role, which follows a logical sequence of tasks:

Pre-flight Checks (preflight.yml):

The role first validates that the kafka_version you've selected is defined in its compatibility matrix.

It then checks if the target OS is listed as compatible with that Kafka version.

If any check fails, the playbook stops immediately with a clear error message.

OS-Specific Setup (setup-debian.yml, setup-redhat.yml, etc.):

The role uses the ansible_os_family fact to determine the host's operating system.

It then runs the appropriate task file to install dependencies using the native package manager (apt for Debian, dnf for Red Hat, etc.).

Common Linux Setup (setup-linux.yml):

After OS-specific tasks, a common set of tasks is executed for all Linux systems:

Creates the kafka group and user.

Downloads and unarchives the specified Kafka version from the official Apache archives.

Copies the binaries to the installation directory (/opt/kafka).

Creates the necessary data and log directories.

Generates the myid file required by each ZooKeeper node.

Deploys systemd service files for both kafka.service and zookeeper.service.

Starts and enables the services.

Configuration Templating:

The main task file uses template modules to generate the zookeeper.properties and server.properties files.

These templates are dynamically populated with data from your inventory, such as broker IDs and the list of ZooKeeper servers.

If a configuration file is changed, the notify directive triggers the corresponding handler to restart the service, ensuring changes are applied.

Service Restarts (Handlers):

The handlers in handlers/main.yml are responsible for restarting the Kafka and ZooKeeper services. They are only triggered when their respective configuration files are modified, making the playbook efficient on subsequent runs.

Role Structure
roles/kafka_universal/
├── handlers/         # Tasks for restarting services
│   └── main.yml
├── tasks/            # Main logic for the role
│   ├── main.yml      # Main entrypoint, dispatches to other files
│   ├── preflight.yml # Compatibility and environment checks
│   ├── setup-debian.yml # Tasks for Debian/Ubuntu
│   ├── setup-redhat.yml # Tasks for RHEL/CentOS
│   └── setup-linux.yml  # Common tasks for all Linux systems
├── templates/        # Jinja2 templates for configuration files
│   ├── server.properties.j2
│   └── zookeeper.properties.j2
└── vars/             # Default variables for the role
    └── main.yml
