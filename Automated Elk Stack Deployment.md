# Project1
Elk Project
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/pbordner/Project1/blob/main/Diagram/Azure%20Resource%20Group%20RedTeam.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the elk.yml file may be used to install only certain pieces of it, such as Filebeat.

  - [Elk install](https://github.com/pbordner/Project1/blob/main/Ansible/elk.yml.txt)
  - [Metricbeat playbook](https://github.com/pbordner/Project1/blob/main/Ansible/metricbeat-playbook.yml.txt)
  - [Filebeat playbook](https://github.com/pbordner/Project1/blob/main/Ansible/filebeat-playbook.yml.txt)


This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
Load Balancing plays an important security role as computing moves evermore to the cloud.
The off-loading function of a load balancer defends an organization against distributed denial-of-service (DDoS) attacks.
It does this by shifting attack traffic from the corporate server to a public cloud provider. 

Jump box servers are installed in such a way that they are placed between a secure zone and a DMZ to provide transparent management on devices on the DMZ once we establish a session.
The jump server acts as an audit for traffic and a single point where we can manage user accounts.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.
- Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash. 
Metricbeat helps you monitor your servers by collecting metrics from the system and services running on the server, such as: Apache.

The configuration details of each machine may be found below.

| Name       | Function  | IP Address | Operating System |
|------------|-----------|------------|------------------|
| Jump Box   | Gateway   | 10.0.0.4   | Linux (Ubuntu)   |
| ElkServer  | Log Server| 10.2.0.4   | Linux (Ubuntu)   |
| Web1       | Server    | 10.0.0.5   | Linux (Ubuntu)   |
| Web2       | Server    | 10.0.0.6   | Linux (Ubuntu)   |
| Web3       | Server    | 10.0.0.9   | Linux (Ubuntu)   |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
 -Personal IP address

Machines within the network can only be accessed by the Jump Box VM.
The Elk Machine can have access from personal IP address through port 5601.

A summary of the access policies in place can be found in the table below.

| Name          | Publicly Accessible | Allowed IP Addresses |
|---------------|---------------------|----------------------|
| Jump Box      | Yes                 | Personal             |
| Load Balancer | Yes                 | Open                 |
| Web1          | No                  | 10.0.0.5             |
| Web2          | No                  | 10.0.0.6             |
| Web3          | No                  | 10.0.0.9             |
| ElkServer     | No                  | 10.2.0.4             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because servcies running can be limited, system installation and update can be streamlined, and processes become more replicable.
The main advantage of Ansible is it's free, Very simple to set up and use: No special coding skills are necessary to use Ansible's playbooks,
Powerful: Ansible lets you model even highly complex IT workflows.

The playbook implements the following tasks:
- installs docker.io, pip3, and the docker module.
```bash
  # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

  # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

  # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```   
- increases the virtual memory (for the virtual machine we will use to run the ELK server)
```bash
  # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
```
- uses sysctl module
```bash
  # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```
- downloads and launches the docker container for elk server 
```bash
# Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![](https://github.com/pbordner/Project1/blob/main/Images/Elk%20Server.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web1 (10.0.0.5)
- Web2 (10.0.0.6)
- Web3 (10.0.0.9)

We have installed the following Beats on these machines:
- Filebeat
- Metric Beat

These Beats allow us to collect the following information from each machine:
- Filebeat is a log data shipper for local files. Installed as an agent on your servers, Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing. An examle of such are the logs produced from the MySQL database supporting our application.
- Metricbeat collects metrics and statistics on the system. An example of such is cpu usage, which can be used to monitor the system health.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file from your Ansible container to your web VM's.
- Update the /etc/ansible/hosts file to include the IP address of the Elk Server VM and webservers.
- Run the playbook, and navigate to http://[Elk_VM_Public_IP]:5601/app/kibana to check that the installation worked as expected.


- Which file is the playbook? 
The Filebeat-configuration

- Where do you copy it?
copy /etc/ansible/filebeat-config.yml to /etc/ansible/filebeat.yml
- Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
update filebeat-config.yml -- specify which machine to install by updating the host files with ip addresses of web/elk servers and selecting which group to run on in ansible.

- Which URL do you navigate to in order to check that the ELK server is running?
http://[your.ELK-VM.External.IP]:5601/app/kibana

filebeats
```bash
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

```

metricbeats
```bash
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup
