## Azure Cloud Environment Design

1- First of all we need to create a Resource Group and call it Red-Team.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/1.PNG)


2- Create a Virtual Network, lets call it Red-Team like on the image below, we will connect our Virtual Machine to Red-Team Resource Group in the other steps

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/2.PNG)

3- Create a Network Security Group. This group will be where our Rules locating

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/3.PNG)

4- Adding Inbound Security Rules to our Network Security Group. Before adding this rule, we should go to []http://ip4.me adress and check our local machine's IP adress because we will add that IP adress to our first rule to allow connection to our VM from home IP adress only. This will allow us to connect to VM from our local machine but no other machine.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/4.PNG)


5- Before creating Virtual Machines, first we should create a SSH-Key for out local machine. This way we will not need passwords or on the other hand, to have SSH-Key will make our system more secure because only SSH Public Key can connect to that Virtual Machine.

  - For this step, we should use our Git Bash and run the comman
  ```
  ssh-key generate
  ```
  This command will create us two SSH keys. One for the private key and the other one is publick key. We will use the public key for setting up our Virtual Machine later.

6- Now we can create our first Virtual Machine, lets call it Jump-Box-Provisioner. 
  - select the Resource Group as Red-Team
  - On the Administrator Account, we will use our SSH public key that we generated in our local machine
  - After that we will jump to Network section and we will select the Static Public IP. This will allow the machine keep the same Public IP all the time. Becasue other way, our jump box machine's public IP will be lost everytime we shutdown the VM. 

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/6.PNG)
  
7- We will add another Inbound Security Rule now, just like the picture below. This step will make our system more secure and only allow the local machine can connect to the Jump Box-Provisoner via SSH which is Port 22. Also we can change the Destionation to Jump-Box-Provisioner's private IP also. 

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/7.PNG)


8- Create Virtual Machine and call Web-VM-1. After creating Jump-Box-Provisioner, now we can create our web server virtual machines. For this project we have created 3 of them and named them Web-Vm-1, Web-Vm-2, Web-Vm-3 and only Web-Vm-1 will have the public IP adress and the rest will have Private IP. This way we can use our Jump-Box-Provisioner to control our Web-Vms. And also only our local machine can connect to our Jump-Box-Provisioner VM so this way our system will be more secure.
  - Now lets connect to our jump-Box-Provisoner and creat SSH key here , after that we will use that SSH public key for our Web-Vms. This step will allow us only Jump-Box-Provisioner can connect to Web-Vms because of SSH public key.

9- After Creating Web-Vms and generating SSH key in the Jump-Box-Provisioner VM, we will change the username and password from the VM settings and add our Jump-Box VM's SSH public key here like the picture below.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/9.PNG)


10- Creating Docker Containers in Jump-Box VM
  - For this step, use commands below

```
sudo apt update && sudo apt install docker.io
systemctl status docker

sudo docker pull cyberxsecurity/ansible
sudo docker run -ti cyberxsecurity/ansible:latest bash


sudo docker ps -a

sudo docker start @container_name && sudo docker attach @container_name

```
  - These commands will install docker.io, create dokcer container and after that we will start to container and attach it. Which mean is we will connect to that container as root
  - For more info about Docker Container and how to use here is the [Cheat Sheet](https://phoenixnap.com/kb/list-of-docker-commands-cheat-sheet)


  - Right now we can check if we can connect to our Web-Vms from our container using SSH command 


11- Now we will change/edit our Ansible folder little bit to make sure our webservers IP adresses and Host file is correct. 
  - use the command cd /etc/ansible and jump into ansible folder, then lets edit the webserver IP part

```
    # It should live in /etc/ansible/hosts
    #
    #   - Comments begin with the '#' character
    #   - Blank lines are ignored
    #   - Groups of hosts are delimited by [header] elements
    #   - You can enter hostnames or ip addresses
    #   - A hostname/ip can be a member of multiple groups
    # Ex 1: Ungrouped hosts, specify before any group headers.

    ## green.example.com
    ## blue.example.com
    ## 192.168.100.1
    ## 192.168.100.10

    # Ex 2: A collection of hosts belonging to the 'webservers' group

    [webservers]
    ## alpha.example.org
    ## beta.example.org
    ## 192.168.1.100
    ## 192.168.1.110
    10.0.0.5 ansible_python_interpreter=/usr/bin/python3
		10.0.0.6 ansible_python_interpreter=/usr/bin/python3
		10.0.0.8 ansible_python_interpreter=/usr/bin/python3
```


- And after that lets edit the host file


```
# What flags to pass to sudo
# WARNING: leaving out the defaults might create unexpected behaviours
#sudo_flags = -H -S -n

# SSH timeout
#timeout = 10

# default user to use for playbooks if user is not specified
# (/usr/bin/ansible will use current user as default)
remote_user = sysadmin

# logging is off by default unless this path is defined
# if so defined, consider logrotate
#log_path = /var/log/ansible.log

# default module name for /usr/bin/ansible
#module_name = command

```

12- We will create our first playbook in /etc/ansible , here is the example of our first playbook


```
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yess
```

13- Add Inbound Security Rules

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/13.PNG)


14- Now time to create our Load Balancer and connect all Web-Vms to this Load Balancer group. We will create a Load Balancer and name it Red-Team.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/14.PNG)

15- After created the Load Balancer, now we will create our Health Probe and Backend Pools. Lets name the Healt Probe as RedTeamProbe and after that add our 3 Web-Vms to the backend pool.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/14a.PNG)
![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/14a.PNG)



16- On this point we will allow our Load Balancer to connect to Port 80 like the picture below

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/16.PNG)


17- Now time to create our Elk-VM-Machine. We will create our Elk machine with Stati Public IP just like our Jump-Box-Provisioner and Also will create another Security Source Group and will call it Red-Team-Vnet. Our Elk machine will be in different security group than other VMs because of different security setup.
	- Differences with Elk-Machine we should change the our Virtual Machine to #Ubuntu VM: 4GB minimum Ram size. 
	- Because Elk machine will need more RAM than our other VMs
	- Should change the time zone different than other Vms, just incase if we have free Azure account. Because Azure Free Subs only allow us to create 4 VMs in same time zone. 
	- Download and Configure the container
	- We will create playbook for the installation
	

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/elk_machine_vm.PNG)


18 - Finalize the ELK Machine
	- Connect to Jump-Box VM via SSH
	- Start and Attach to docker container
	- Go to /etc/ansible folder

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/docker_container_attach.PNG)


   - Edit host file
   - Edit ansible file
    
![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/elk_host.PNG)


- Create and playbook and name it Install-elk.yml and endit the file like below

```
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
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
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
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


19- On this step we will run the playbook and finalize the Elk container
	- Run Playbook with a command 
	``
	ansible-playbook install-elk.yml
	``
	Lets connect to our ELK machine via SSH 

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/elk_ssh.PNG)


20-	- Run docker ps -a 
	- Sudo docker start elk
	- We can connect our Elk container here also to see log files but for more Visual part lets use the Kibana. If we attach to elk to see log files, it will look like the picture below.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Azure_Cloud/docker_container_attach.PNG)



21-  lets Connect to Kibana on the browser using http://ElkMachinePublicIP:5601/app/kibana#/home

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Kibana/kibana_home.PNG)
	
22- Lets Install Filebeat on the DVWA Container
- Go to Kibana home page
- Add Log data
- System Logs
- Download the Filebeat
	
```
	curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
	sudo dpkg -i filebeat-7.6.1-amd64.deb
```
- Edit the configuration
- Modify /etc/filebeat/filebeat.yml
	
	```
	output.elasticsearch:
  	hosts: ["10.2.0.4:9200"]
  	username: "elastic"
  	password: "changeme"
  
	setup.kibana:
  	host: "10.2.0.4:5601"
  	```

- Create Filebeat Installation Playbook

```
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

```

- Verify the Installation and Playbook
- Click to System logs Dashbord

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Kibana/filebeat_check.PNG)

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/filebeat.PNG)


23- Lets Install metricbeat on the DVWA Container
	- Go to Kibana home page
	- Add Metric data
	- Docker Metrics
	- Download the Metricbeat
	
	```
	curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb
	sudo dpkg -i metricbeat-7.6.1-amd64.deb
	```
- Edit the configuration
- Modify /etc/metricbeat/metricbeat.yml

	```
	output.elasticsearch:
  	hosts: ["10.2.0.4:9200"]
  	username: "elastic"
  	password: "changeme"
  
	setup.kibana:
  	host: "10.2.0.4:5601"
	```

 - Create Metricbeat Installation Playbook
```	
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
 
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd$

  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

  - name: setup metric beat
    command: metricbeat setup

  - name: start metric beat
    command: service metricbeat start
 ```
 
 
![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/Kibana/metricbeat_check.PNG)

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/metricbeat.png)	
