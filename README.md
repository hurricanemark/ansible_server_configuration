# Configure Servers Using Ansible  


[__Ansible__](https://en.wikipedia.org/wiki/Ansible) is a server configuration tool.  It is an open source software that automates software provisioning, configuration management, and application deployment. Ansible connects via SSH, remote PowerShell or via other remote APIs. 

__This document describes steps used to have Ansible configured and controlled three separate OS environments (servers) via the use of SSH.__

For the purpose of demonstrating how Ansible is used to configure and provision remote servers via SSH, let's refer to the table below.  We have a local controller node (*ubuntu16*), and three remote nodes (*raspberrypi3*, *linux1rhel*, *azuresles12*).

| Ansible Controller | Servers        | IP_addr       | User          |
| ------------------ |:---------------| -------------:|---------------|
| *ubuntu16*         |                | 192.168.0.10  | hurricanemark |
|                    | *raspberrypi3* | 192.168.0.35  | hurricanemark |
|                    | *linux1rhel*   | 148.100.99.23 | jakeo         |
|                    | *azuresles12*  | 13.99.130.110 | hurricanemark |
 

#### Aliases for SSH:
 __Content of *~/.ssh/config* on *ubuntu16*__
 
	Host raspberrypi3
      HostName 192.168.0.35
      User hurricanemark
	Host linux1rhel
      HostName 148.100.99.23
      User jakeo
      IdentityFile ~/.ssh/identity_linux1_priv
 	Host azuresles12
      HostName 13.99.130.110
      User hurricanemark

You can now reference ip address 192.168.0.35 by its alias *raspberrypi3*

> hurricanemark@ubuntu16> **ssh raspberrypi3**


### Connect to servers with openssh
> hurricanemark@ubuntu16> **ssh-keygen**

> hurricanemark@ubuntu16> **ssh-copy-id markn@raspberrypi3**

> hurricanemark@ubuntu16> **ssh-copy-id markn@azuresles12**

> hurricanemark@ubuntu16> **ssh-copy-id -i ~/.ssh/identity_linux1_priv jakeo@linux1rhel**  

### Install ansible on the controller
> hurricanemark@ubuntu16#  **apt-add-repository ppa:ansible/ansible**

> hurricanemark@ubuntu16# **apt-get update**

> hurricanemark@ubuntu16# **apt install ansible**

> hurricanemark@ubuntu16# **ansible -version**


### Create ansible workspace
1. Create private workspace ( in your home directory).
> hurricanemark@ubuntu16> **cp -R /etc/ansible ~/myansible**

> hurricanemark@ubuntu16> **vi ~/myansible/ansible.cfg**

Modify this line

		inventory	= hosts
    
> hurricanemark@ubuntu16> **vi hosts**

Replace content with

		raspberrypi3
        azuresles12
        linux1rhel


2. Customize basic tasks.  Perhaps common requirement to your automated operation


* Create basic common tasks for ubuntu platforms
> hurricanemark@ubuntu16> **mkdir -p ~/myansible/roles/basic4ubuntu/tasks**
> hurricanemark@ubuntu16> **vi ~/myansible/roles/basic4ubuntu/tasks/main.yml**


		- name: "Installing python3"
    	   apt: pkg=python3 state=installed
        
* Create basic common tasks for ubuntu platforms
> hurricanemark@ubuntu16> **mkdir -p ~/myansible/roles/basic4rhel/tasks**
> hurricanemark@ubuntu16> **vi ~/myansible/roles/basic4rhel/tasks/main.yml**


		- name: "Installing python3"
    	   yum: pkg=python3 state=installed
        
        
* Create basic common tasks for ubuntu platforms
> hurricanemark@ubuntu16> **mkdir -p ~/myansible/roles/basic4sles/tasks**
> hurricanemark@ubuntu16> **vi ~/myansible/roles/basic4sles/tasks/main.yml**


		- name: "Installing python3"
    	  zypper: pkg=python3 state=installed



* Create ansible playbook for controller node
> hurricanemark@ubuntu16> **vi ~/myansible/playbook.yml**


		---
        - hosts: raspberrypi3
        	become: true
            roles:
            	- basic4ubuntu
                
        - hosts: linux1rhel
        	become: true
            roles:
            	- basic4rhel
                
        - hosts: azuresles12
        	become: true
            roles:
            	- basic4sles
                



### Ansible Ad-hoc commands

Let's test the interconnectivity we have set up thus far.
> hurricanemark@ubuntu16> **ansible -version**

> hurricanemark@ubuntu16> **ansible -m shell -a 'whoami' all**

> hurricanemark@ubuntu16> **ansible -m shell -a 'ping c1 www.google.com' all**

> hurricanemark@ubuntu16> **ansible -b -K -m user -a 'name=hurricanemark' all**


Now, run basic tasks on all three servers.

> hurricanemark@ubuntu16> **ansible-playbook -K ~/myansible/playbook.yml**






## Feedback

https://github.com/hurricanemark