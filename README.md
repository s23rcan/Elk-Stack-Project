## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Diagrams/Week_13_ELK_Stack_Project_v1.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Azure Cloud Environment file may be used to install only certain pieces of it, such as Filebeat.

  [Filebeat Playbook](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Ansible/filebeat_playbook.txt)

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly reliable, in addition to restricting access to the network.
- Load balancers can reduce the load on your web servers and optimize traffic for a better user experience. Load Balancing plays an important security role as computing moves evermore to the cloud. The off-loading function of a load balancer defends an organization against distributed denial-of-service (DDoS) attacks. It does this by shifting attack traffic from the corporate server to a public cloud provider. The advantage of using a jump box is that limiting the access to one administator and allowing us to have access to other VMs and Elk server.

The configuration details of each machine may be found below:

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system traffic.
- What does Filebeat watch for? Installed as an agent on your servers, Filebeat monitors the log files or locations that are specified, collects the log events, and forwards them either to Elasticsearch or Logstash for indexing. Filebeat is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.

- What does Metricbeat record? It periodically collect metrics from the operating system and from services running on the server. The information collected is sent to the output that you specify, such as Elasticsearch or Logstash.

The configuration details of each machine may be found below.

```
| Name                 | Function         | IP Address | Operating System |
|----------------------|------------------|------------|------------------|
| Jump-Box-Provisioner | Gateway          | 10.0.0.4   | Linux            |
| Web-VM-1             | DVWA Containers  | 10.0.0.5   | Linux            |
| Web-VM-2             | DVWA Containers  | 10.0.0.6   | Linux            |
| Web-VM-3             | DVWA Containers  | 10.0.0.8   | Linux            |
| ELK-VM-1             | Configuration VM | 10.2.0.4   | Linux            |

```
### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- (0.0.0.0) My Personal Machine's IP Adress

Machines within the network can only be accessed by Jump Box Provisioner VM , SSH.
- My Personal Machine can access to the ELK-Server from the Jump Box 
- The DVWA's from the Jumpbox 

A summary of the access policies in place can be found in the table below.

```
| Name      | Publicly Accessible  | Allowed IP Addresses                         |
|-----------|----------------------|----------------------------------------------|
| Jump Box  | Yes                  | 10.0.0.4 - Personal IP Adress                |
| Web-VM-1  | No                   | 10.0.0.4                                     |
| Web-VM-2  | No                   | 10.0.0.4                                     |
| Web-VM-3  | No                   | 10.0.0.4                                     |  
| Elk-Server| No                   | 10.0.0.4 - Jump Box - Personal IP\Port 5601  |

```

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...

- Free: Ansible is an open-source tool.
- Very simple to set up and use: No special coding skills are necessary to use Ansible’s playbooks (more on playbooks later).
- Powerful: Ansible lets you model even highly complex IT workflows. 
- Flexible: You can orchestrate the entire application environment no matter where it’s deployed. You can also customize it based on your needs.
- Agentless: You don’t need to install any other software or firewall ports on the client systems you want to automate. You also don’t have to set up a separate management structure.
- Efficient: Because you don’t need to install any extra software, there’s more room for application resources on your server.


The playbook implements the following tasks:
**In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc.** 
- Install docker.io
- Install pip3
- Install Docker python module
- Increase virtual memory
- Creates Elk Container
- Specificies open ports

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/elk_docker.PNG)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-VM-1   10.0.0.5
- Web-VM-2   10.0.0.6
- Web-VM-3   10.0.0.8

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat watches for log files/locations and collect log events. (Filebeat: Lightweight Log Analysis & Elasticsearch)

![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/filebeat.PNG)

- Metricbeat records metrics and statistical data from the operating system and from services running on the server (Metricbeat: Lightweight Shipper for Metrics)


![](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Images/metricbeat.png)


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

- Copy the ![filebeat-playbook.yml](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Ansible/filebeat_playbook.txt) file to /etc/ansible/roles
- Update the /etc/ansible/hosts file to include the IP address of the Elk Server VM and webservers.
- Run the playbook, and navigate to Kibana page at with http://ELKServerPublicIP:5601/app/kibana# to check that the installation worked as expected.




-Which file is the playbook?
  - [ELK-Playbook](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Ansible/elk_playbook.txt) - used to install ELK Server
  - [Filebeat-Playbook](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Ansible/filebeat_playbook.txt) - Used to install and configure Filebeat on Elk Server and DVWA servers
  - [Metricbeat-Playbook](https://github.com/s23rcan/Elk-Stack-Project/blob/main/Ansible/metric_playbook.txt) - Used to install and configure Metricbeat on Elk Server and DVWA servers
  
-Where do you copy it?  
  - /etc/ansible/
  
-Which file do you update to make Ansible run the playbook on a specific machine? 
  - /etc/ansible/hosts.cfg

-How do I specify which machine to install the ELK server on versus which to install Filebeat on?
   - /etc/ansible/hosts.cfg ensure you you have input your IP address for the specific machine you;d like to install the ELK server versus the Filebeats.
   
-Which URL do you navigate to in order to check that the ELK server is running?
  - http://ELKServerPublicIP:5601
  
  
  
  
  
  
  
  
 
