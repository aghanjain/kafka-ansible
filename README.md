Ansible Playbook for Apache Kafka Cluster Deployment This Ansible playbook automates the deployment and configuration of a high-availability Apache Kafka and ZooKeeper cluster on Linux servers. The playbook is designed to be universal, supporting multiple OS families, and interactive, allowing the user to select the desired Kafka version at runtime.

**Features **✨ Interactive Kafka Version Selection: Prompts the user to choose from a list of supported Kafka versions.

Multi-OS Support: Handles setup for both Debian-family (Ubuntu) and RedHat-family (CentOS, RHEL) distributions.

Dynamic Configuration: Uses Jinja2 templates to dynamically generate server.properties and zookeeper.properties based on your inventory.

Preflight Compatibility Checks: Validates that the selected Kafka version is compatible with the target server's operating system before starting the installation.

Automated Service Setup: Creates and manages systemd services for both Kafka and ZooKeeper, ensuring they start on boot and restart on failure.

Centralized Configuration: All server IPs and key variables are managed in easy-to-edit YAML files.

Prerequisites Before running this playbook, ensure you have the following:

Ansible Control Node: A machine with Ansible installed (version 2.12 or newer).

Target Servers: One or more Linux servers (VMs) that will host the Kafka and ZooKeeper nodes. These should be a supported OS (e.g., Ubuntu 22.04 or CentOS 9).

Network Connectivity:

The Ansible control node must have SSH access to all target servers. You should add the control node's public SSH key to the authorized_keys file on each target server.

All target servers must be able to communicate with each other over their private network on Kafka and ZooKeeper ports (e.g., 2181, 2888, 3888, 9092).

Configuration ⚙️ Follow these steps to configure the deployment for your environment.

Step 1: Update the Inventory File The inventory.yml file is where you define the servers that will be part of your cluster.

Open inventory.yml.

Under zookeeper_servers and kafka_servers, list your target nodes.

For each node, set the ansible_host to its private IP address.

Assign a unique zookeeper_myid (starting from 1) and kafka_broker_id (starting from 0) to each node.

Example inventory.yml:

YAML

all: children: zookeeper_servers: hosts: kafka-node-0: ansible_host: 10.0.0.50 # <-- Set private IP here zookeeper_myid: 1 kafka-node-1: ansible_host: 10.0.0.43 # <-- Set private IP here zookeeper_myid: 2

kafka_servers:
  hosts:
    kafka-node-0:
      ansible_host: 10.0.0.50 # <-- Set private IP here
      kafka_broker_id: 0
    kafka-node-1:
      ansible_host: 10.0.0.43 # <-- Set private IP here
      kafka_broker_id: 1
Step 2: Customize Installation and Kafka Variables Global variables for installation paths and Kafka's server.properties are located in group_vars/kafka_servers.yml.

Open group_vars/kafka_servers.yml.

Adjust variables like kafka_install_dir_linux, kafka_log_dir_linux, num_partitions, and log_retention_hours as needed. The defaults are generally sensible for a basic setup.

Example group_vars/kafka_servers.yml:

YAML

Installation variables
kafka_user: "kafka" kafka_install_dir_linux: "/opt/kafka" kafka_log_dir_linux: "/var/log/kafka"

Kafka Broker Configuration
kafka_port: 9092 num_partitions: 3 log_retention_hours: 72 Step 3: (Optional) Review Kafka Compatibility The list of supported Kafka versions and their compatible operating systems is defined in roles/kafka_universal/vars/main.yml. You can edit this file to add new Kafka versions or support additional OS versions.

Example roles/kafka_universal/vars/main.yml:

YAML

kafka_available_versions:

"3.7.0"
"3.6.2"
kafka_compatibility: "3.7.0": - { family: "Debian", versions: ["24.04", "22.04"] } - { family: "RedHat", versions: ["9", "8"] }

Usage ▶️ Once your configuration is complete, run the main playbook from your control node.

Navigate to the root directory of the project (/kafka-cluster).

Execute the playbook using the following command:

Bash

ansible-playbook kafka-playbook.yml -i inventory.yml The playbook will first run a local play to prompt you for the Kafka version you wish to install.

After you select a valid version, it will proceed to deploy and configure Kafka and ZooKeeper on all target nodes defined in your inventory.

Role Deep Dive 🛠️ The core logic is contained within the kafka_universal role. Here’s a summary of its execution flow:

preflight.yml: Checks if the chosen Kafka version and target OS are compatible based on the matrix in vars/main.yml.

main.yml: Acts as the entrypoint, including the correct OS-specific setup file (setup-debian.yml or setup-redhat.yml).

setup-debian.yml/setup-redhat.yml: Install OS-specific dependencies like Java.

setup-linux.yml: Contains the main, OS-agnostic installation steps:

Creates the kafka user and group.

Downloads and unarchives the specified Kafka version.

Creates necessary directories (log.dirs, dataDir).

Generates systemd service files for Kafka and ZooKeeper from templates.

Configuration Update: The final tasks in tasks/main.yml deploy the zookeeper.properties and server.properties files using the Jinja2 templates.

Service Restarts: If a configuration file is changed, the corresponding handler in handlers/main.yml is notified, which triggers a graceful restart of the zookeeper or kafka service.