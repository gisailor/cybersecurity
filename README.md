# cybersecurity
Home work for Cyber bootcamp project 1 
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[https://drive.google.com/file/d/13XFZrByF7DroQ4M_tUYjUZ7KdFRAr0fI/view?usp=sharing](Images/diagram)

These files have been tested and used to generate a live ELK deployment on Azure and DVWA Web servers behind a load balancer. They can be used to recreate the entire deployment pictured above. Alternatively, select portions of the config files and yaml files may be used to install only certain pieces of it, such as Filebeat.

  - DVWA Playbook (Yaml used to intall DVWA HTML on Web Servers) 
    ```YAML
    ---
    - name: Config Web VM with Docker and install DVWA Web site containers on Web VMs. 
      hosts: webservers
      become: true
      tasks:
      - name: docker.io
        apt:
          force_apt_get: yes
          update_cache: yes
          name: docker.io
          state: present
​
      - name: Install pip3
        apt:
          force_apt_get: yes
          name: python3-pip
          state: present
​
      - name: Install Docker python module
        pip:
          name: docker
          state: present
​
      - name: download and launch a docker web container
        docker_container:
          name: dvwa
          image: cyberxsecurity/dvwa
          state: started
          published_ports: 80:80
​
      - name: Enable docker service
        systemd:
          name: docker
          enabled: yes
   
  
  - Webservice configuration file changes
       [webservers]
        10.0.0.5 ansible_python_interpreter=/usr/bin/python3
	10.0.0.6 ansible_python_interpreter=/usr/bin/python3
	10.0.0.8 ansible_python_interpreter=/usr/bin/python3

	# default user to use for playbooks if user is not specified
    	# (/usr/bin/ansible will use current user as default)
    	remote_user = sysadmin

  - Filebeat Playbook
---
- name: installing and launching filebeat logging utlity 

  hosts: webservers
  become: yes
  tasks:
​
  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
 
  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
​
  - name: drop in filebeat.yml 
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
​
  - name: enable and configure system module
    command: filebeat modules enable system
​
  - name: setup filebeat
    command: filebeat setup
​
  - name: start filebeat service
    command: service filebeat start
 
  - Filebeat configuration file changes down loaded with curl command: curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > filebeat-config.yml
   line #1106 and replace the IP address with the IP address of ELK machine.
​   output.elasticsearch:
   hosts: ["10.1.0.4:9200"]
   username: "elastic"
   password: "changeme"
  
​   line #1806 replaced the IP address with the IP address of ELK machine.
​   setup.kibana:
   host: "10.1.0.4:5601"


  - Metricbeat Playbook used to install Metricbeat monitoring utlity on ELK stack server 
---
  - name: Install and Launch Metricbeat
    hosts: webservers
    become: yes
    tasks:
    # Use command module
    - name: Download metricbeat .deb metric
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb
    
    # Use command module
    - name: Install metricbeat .deb
      command: dpkg -i metricbeat-7.6.1-amd64.deb

    # Use copy module
    - name: Copy over and drop in metricbeat.yml
      copy:
        src: /etc/ansible/files/metricbeat-config.yml
        dest: /etc/metricbeat/metricbeat.yml
    
    # Use command module
    - name: Enable and Configure Docker Module
      command: metricbeat modules enable docker
    
    # Use command module
    - name: Setup metricbeat
      command: metricbeat setup
    
    # Use command module
    - name: Start metricbeat service
      command: service metricbeat start

  - Metricbeat configuration file changes:
    starting on line 61
    setup.kibana:
    host: "10.1.0.4:5601" 

    starting on line 96
    hosts: ["10.1.0.4:9200"]
    username: "elastic"
    password: "changeme"



This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly scaleable, in addition to restricting access to the network.
The Load Balancer off-loading function protects the system against denial-of-service (DDoS) attacks provides redunducey if servers go down or there is a high volume of traffic. 
This is accomlished by shifting traffic across several VMs. The system also has a jump box  that functions like a proxy server for the system. It provides isolated access to a private network internl network. 
The jumpbox is only accessible to computers on the internal network. The jump box is configured to allow acces to internl VMs. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the system logs, file logs etc. and system traffic.
The installed filebeat kibana utlity allows you to set up monitoring on select files and system componets and montiors selected activities in each log.  
Metricbeat montors and reports system operations metrics for services running on selected servers. 


The configuration details of each machine may be found below.
| Name     | Function | internal IP Address | Operating System     |
|----------|----------|---------------------|----------------------|
| Jump Box | Gateway  | 10.0.0.1            | Linux / ubuntu 18.04 |
| Web 1    | Webserver| 10.0.0.5            | Linux / ubuntu 18.04 |
| Web 2    | Webserver| 10.0.0.6            | Linux / ubuntu 18.04 | 
| Web 3    | Webserver| 10.0.0.8            | Linux / ubuntu 18.04 |
| ElkStack | Elkserver| 10.1.0.4            | Linux / ubuntu 18.04 |
                  |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:74.174.37.230 
(Non static host machine IP address must be verified prior to each connection attempt.)


Machines within the network can only be accessed by the jumb box VM.
The jump box also has access to the ELKstack VM via the interal ip 10.1.0.4 

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box |     No              | 74.174.37.230        |
| Web-1    |     No              | 10.0.0.5             |
| Web-2    |     No              | 10.0.0.6             |
| Web-1    |     No              | 10.0.0.8             |
| ELKStack |     No              | 10.1.0.4             |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it is repeatable and
configureable. This method supports rapid upgrades and deployments on multible servers. 

The playbook implements the following tasks:
- Installs Docker
- Installs python3
- Downloads and installs current kibana filebeats softare
- Downloads and installs current kibana metricbeat softare- 
- 

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1 : 10.0.0.5
- Web-2 : 10.0.0.6
- Web-3 : 10.0.0.8

We have installed the following Beats on these machines:
- Filebeats 7.4.0
- Metricbeat 7.6.1

These Beats allow us to collect the following information from each machine:
- Filebeat monitors selected system log files for changes and selected message severity levels. Metricbeats monitors system performance and provices alerts when 
  services have issues. For example, when a servers CPU is running at capacity or the RAM is being fully utlized.  

Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the anible.cfg  file to etc/ansible.
- Update the config file to include Host IP of server that contains the ELK stack and the config file for the webservers with all the web server IPs requiring configuration / deployment of Filebeat and Metirc beat.
- Run the playbook, and navigate to the ELK stack / Kibana server to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? 
-  How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

    Copy the install-elk.yml, filebeat-playbook.yml and files to /etc/ansible/host.
    Update the install-elk.yml and filebeat-playbook.yml file to include the machine you want use the playbooks on by changing the hosts name on the 3rd line.
    Run the playbook, then use a web broswer to navigate to http://[ELKStack IP]:5601/app/kibana to verify the installation is working as expected.

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
