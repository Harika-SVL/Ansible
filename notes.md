## Introduction

* Used for the applications which is accessed globally .
* Written once and applicable over various os flavours (servers too) .
* When frequent updates to be made into all the systems used in an organisation .

## Deployment options 

![Alt text](shots/1.PNG)

* We can have mainly two deployment approaches :
1. Declarative approach
2. Procedural approach

![Alt text](shots/2.PNG)

* Configuration management is the declarative deployment which ensures :
1. Idempotence : Run this once or n times you will have same result 
2. Desired state : We express configuration to acheive a desired state 
3. Reusable

## Configuration Management (CM)

* There are two types :
1. PULL based configuration
2. PUSH based configuration

### PULL based Configuration Management :

* Communication direction => Node to CM server

![Alt text](shots/3.PNG)

* Required : 
=> Agent needs to be installed with necessary credentials to connect to CM Server

![Alt text](shots/4.PNG)

* Popular tools :
=> chef
=> puppet

![Alt text](shots/5.PNG)

### PUSH based Configuration Management :

* Communication direction => CM Server to Node

![Alt text](shots/6.PNG)

* Required :
=> List of nodes (inventory)
=> Credentials to login into node

![Alt text](shots/7.PNG)

* Popular tools :
=> Ansible
=> SaltStack

![Alt text](shots/8.PNG)

# Ansible
* Referrence : https://docs.ansible.com/ 

## Architecture and Workflow

* Basic workflow

![Alt text](shots/9.PNG)

* In this it is a push based configuration management
* Here the control node can execute desired state on nodes using
=> adhoc commands
=> playbooks (YAML files)

## How organisations work over multiple sever scenerio

* They have many a servers and admins
* And creating individual logins for each of them is not possible/feasible
* So, they create a service account for the admins to login and perform administration

![Alt text](shots/10.PNG)

* Let's our service account’s name  be devops
* Having username and password is not a sensible way so we use keypair based authentication

## Key pair authetication using RSA (Rivest, Shamir, Adleman)

* In this we generate two keys (i.e., public and private) of some mathematical prime number calulations

=> Public key : It comprises of two numbers, in which one number is the result of the product of two large prime numbers. This key is provided to all the users.

=> Private key : It is derived from the two prime numbers involved in public key and it always remains private. 


* To create a keypair use : 'ssh-keygen' command

![Alt text](shots/11.PNG)

* Now the public key to linux machine 'ssh-copy-id < username >@ipaddess'

![Alt text](shots/12.PNG)

* Connect to the machine using private key 'ssh -i < path-to-private key > username@ipaddress'
* Generally private keys created will have extension of .pem (i.e we create a Service account public and private key). 
* Copy the service account public key to all the servers and also disable the password based authentication

## Setting up sudo permissions

* We need to add devops user to the sudoers group (Wheel in Redhat server)
* We use 'sudo visudo' command

![Alt text](shots/13.PNG)

## Evironment setup (Installing and configuring) for Ansible

* Start two linux machines(vms) and rename them as Ansible control node and node(s)

![Alt text](shots/14.PNG)

[ Note: (This is done in both the machines)
To enable password authentications edit config 'sudo vi /etc/ssh/sshd_config' and set 'PasswordAuthentication to yes' ] 

![Alt text](shots/15.PNG)
![Alt text](shots/16.PNG)

=> Now need to restart the service, 'sudo service sshd restart'

* Create a service account/user('sudo adduser devops') in all machines with sudo permissions('sudo visudo') (This is to be done on both the machines)

![Alt text](shots/17.PNG)
![Alt text](shots/18.PNG)

[ Note : Login into both the machines as devops user using 'ssh devops@< same machine public_IPaddress >']

![Alt text](shots/19.PNG)
![Alt text](shots/20.PNG)

[ Optional: as both machines are in the same network we can also use private_IPaddress of other machine to connect instead of public_IPaddress ]

* Install Ansible on the control node using commands only on the control node :

-> sudo apt update
-> sudo apt install software-properties-common -y
-> sudo add-apt-repository --yes --update ppa:ansible/ansible
-> sudo apt install ansible -y
-> ansible --version

![Alt text](shots/21.PNG)

[ Optional: Disable password based authentication ]

* Create a keypair using 'ssh-keygen' and copy the ssh-key to the node(private_ip also works) machine using 'ssh-copy-id devops@< node machine public_IPaddress >'

![Alt text](shots/22.PNG)
![Alt text](shots/23.PNG)

* Try logging into the node machine from the control node machine using 'ssh < other machine IPaddress >' (private IP address also works) and then logout 'exit'

![Alt text](shots/24.PNG)
![Alt text](shots/25.PNG)

* Adding inventory ('mkdir inventory') - Create a file named hosts ( 'vi inventory/hosts') with entry (public-IP of the node) (private-IP works when machines in same network)
* Check connectivity by executing 'ansible -i inventory/hosts -m ping all'

![Alt text](shots/26.PNG)

## Communication types

* Ansible can communicate with nodes by using two approaches :

1. adhoc commands : (build command for desired state)

* command uses the /usr/bin/ansible command-line tool to automate a single task on one or more managed nodes.
* These are quick and easy, but they are not reusable.
* These demonstrate the simplicity and power of Ansible. The concepts will port over directly to the playbook language.
* These are great for tasks you repeat rarely.

Syntax : ansible [pattern] -m [module] -a "[module options]"

E.g : For Gathering facts
      'ansible all -m ansible.builtin.setup'

2. Playbook : (create file for desired state)

* These offer a repeatable, re-usable, simple configuration management and multi-machine deployment system (well suited to deploying complex applications).
* If you need to execute a task with Ansible more than once, write a playbook and put it under source control,then you can use the playbook to push out new configuration or confirm the configuration of remote systems.
* Playbooks with multiple ‘plays’ can orchestrate multi-machine deployments, running one play on your webservers, then another play on your database servers, then a third play on your network infrastructure, and so on
* Here smallest unit of work is done by module 

![Alt text](shots/27.PNG)

Syntax : YAML format 

 * Running a playbook 
    ('ansible-playbook -i < inventory-path > < playbook-path >')
* YAML supports yes-no for true-false unlike JASON

![Alt text](shots/28.PNG)

## Sample playbook execution
 
* Complete the entire Ansible control node - node setup
* Create inventory and add node hosts
  * mkdir inventory
  * vi inventory/hosts
* Create a directory and add a yaml file
  * mkdir playbooks
  * vi playbooks/hello.yml

 Playbook : playbooks/hello.yml 

![Alt text](shots/39.PNG)
![Alt text](shots/29.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/hello.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/hello.yml
* ansible-playbook -i inventory/hosts playbooks/hello.yml

![Alt text](shots/30.PNG)

* Check for output on node machine 

![Alt text](shots/31.PNG)

## WOW (Ways Of Working)

1. list down all the manual steps
2. Ensure all the steps are working
3. For each step find a module and express the desired state

## Sample-1 : Install apache server

 Manual steps :

* sudo apt update
* sudo apt install apache2 -y
* Verify installation 'http://public-ip' 

![Alt text](shots/32.PNG)
![Alt text](shots/33.PNG)

Playbook steps :

* Complete the entire Ansible control node - node setup
* Create inventory and add node hosts
  -> mkdir inventory
  -> vi inventory/hosts
* create a directory and add a yaml file
  -> mkdir playbooks
  -> vi playbooks/apache2.yml

 Playbook : playbooks/apache2.yml 

![Alt text](shots/38.PNG)
![Alt text](shots/34.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/apache2.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/apache2.yml
* ansible-playbook -i inventory/hosts playbooks/apache2.yml

![Alt text](shots/35.PNG)

* Check status on the node 'sudo systemctl status apache2'
* Expose using node IPaddress

![Alt text](shots/36.PNG)
![Alt text](shots/37.PNG)

## Sample-2 : Install LAMP server on ubuntu
   ( skipping mysql installation )

 Manual steps :

* sudo apt update
* sudo apt install apache2 -y
* sudo apt install php libapache2-mod-php php-mysql -y
 
 [# Create a file called as /var/www/html/info.php with below content
#(?php phpinfo(); ?) ]

* sudo -i
* echo '<?php phpinfo(); ?>' > /var/www/html/info.php
* exit
* sudo systemctl restart apache2
* Verify installation 'http://public-ip/info.php' 

![Alt text](shots/40.PNG)
![Alt text](shots/41.PNG)
![Alt text](shots/42.PNG)

Playbook steps :

* Complete the entire Ansible control node - node setup
* Create inventory and add node hosts
  -> mkdir inventory
  -> vi inventory/hosts
* create a directory and add a yaml file
  -> mkdir playbooks
  -> vi playbooks/lamp-ubuntu.yml
* For writing playbook we first search for module as 'apt in ansible'
* Now select parameters

 Playbook : playbooks/lamp-ubuntu.yml

![Alt text](shots/46.PNG)

playbook/info.php

* <?php phpinfo(); ?>

![Alt text](shots/45.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/lamp-ubuntu.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/lamp-ubuntu.yml
* ansible-playbook -i inventory/hosts playbooks/lamp-ubuntu.yml

![Alt text](shots/43.PNG)
![Alt text](shots/44.PNG)

* Check status on node and expose with node IPaddress

![Alt text](shots/47.PNG)
![Alt text](shots/48.PNG)
![Alt text](shots/49.PNG)

## Sample-3 : Install LAMP stack on Redhat9
   ( skipping mysql installation )

 Manual steps :

* sudo yum install httpd -y
* sudo systemctl enable httpd
* sudo systemctl start httpd
* sudo yum install php -y
* sudo -i
* echo '<?php phpinfo(); ?>' > /var/www/html/info.php
* exit
* sudo systemctl restart httpd
* Verify installation 'http://public-ip/info.php' 

Playbook steps :

* Complete the entire Ansible control node - node setup
* Create inventory and add node hosts
  -> mkdir inventory
  -> vi inventory/hosts
* create a directory and add a yaml file
  -> mkdir playbooks
  -> vi playbooks/lamp-redhat.yml

 Playbook : playbooks/lamp-redhat.yml

![Alt text](shots/60.PNG)

playbooks/info.php

* <?php phpinfo(); ?>

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/lamp-redhat.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/lamp-redhat.yml
* ansible-playbook -i inventory/hosts playbooks/lamp-redhat.yml


## METRIC BEATS INSTALLATION

 Manual steps (using apt package):
       
* wget -q0 – https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
* sudo apt-get install apt-transport-https
* echo “deb https://artifacts.elastic.co/packages/8.x/apt stable main” | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
* sudo apt-get update && sudo apt-get install metricbeat
* sudo systemctl enable metricbeat
* sudo systemctl start metricbeat
* sudo systemctl status metricbeat
 
![Alt text](shots/93.PNG)
![Alt text](shots/94.PNG)

playbook : playbooks/metricbeat.yml

![Alt text](shots/61.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/metricbeat.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/metricbeat.yml
* ansible-playbook -i inventory/hosts playbooks/metricbeat.yml  

![Alt text](shots/93.PNG)
![Alt text](shots/94.PNG)

## Handlers

* Sometimes you want a task to run only when a change is made on a machine. 
* For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged.Ansible uses handlers to address this use case. 
* Handlers are tasks that only run when notified.

### E.g.1 : playbooks/redhat.yaml



Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/redhat.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/redhat.yml
* ansible-playbook -i inventory/hosts playbooks/redhat.yml

### E.g.2 : playbooks/ubuntu.yaml

![Alt text](shots/55.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/ubuntu.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/ubuntu.yml
* ansible-playbook -i inventory/hosts playbooks/ubuntu.yml   

![Alt text](shots/56.PNG)
![Alt text](shots/57.PNG)

* Check the output using node IPaddress and also add /info.php and reload

![Alt text](shots/58.PNG)
![Alt text](shots/59.PNG)

## Inventory

* Inventory in Ansible represents the hosts which we need to connect to.
* Ansible inventory is broadly classified into two types :
  (a) Static inventory: 
   * where we mention the list of nodes to connect to (in a file)
  (b) Dynamic inventory: 
    * where we mention some script/plugin which will dynamically find out the nodes to connect to

## Static inventory

* Static inventory can be mentioned in two formats :

  (a) INI format

  * It is a configuration file that consists of a text-based content with a structure and syntax comprising key–value pairs for properties and sections that organize the properties
  * The headings in brackets are group names, which are used in classifying hosts and deciding what hosts you are controlling at what times and for what purpose. 

E.g : INI format - hosts.ini
  
        [ubuntu]
        172.31.27.136

        [redhat]
        172.31.23.22

        [webserver]
        172.31.27.136
        172.31.23.22

  (b) YAML format

  * It is a configuration file that consists of a text-based content with a structure and syntax comprising key–value pairs for properties and sections that organize the properties

E.g : YAML format - hosts.yml
  

        all:
          children:
            ubuntu:
              hosts:
                172.31.27.136:
            redhat:
              hosts:
                172.31.23.22:
            webserver:
              hosts:
                172.31.27.136:
                172.31.23.22:
 
 ## Facts

 * Ansible collects information about the node on which it is executing by the help of module called as 'setup'
* Playbook by default collects information about nodes where it is executing, we can use this with the help of variables
* To display the information we use 'ansible -m 'setup' -i 'localhost, ' all' command and get a JSON format info
* Collecting information can be disabled as well

![Alt text](shots/50.PNG)

* In the playbook the facts will be collected and will be available in a special variables ansible_facts

playbook : playbooks/distribution.yml

![Alt text](shots/83.PNG)
![Alt text](shots/85.PNG)

* The statement ansible_facts['os_family'] represents accessing os family from the facts collected
* From facts the variables can be accessed with full names ansible_default_ipv4 or ansible_facts['default_ipv4']

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/distribution.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/distribution.yml
* ansible-playbook -i inventory/hosts playbooks/distribution.yml        

![Alt text](shots/84.PNG)


playbook : playbooks/ipv4.yml

![Alt text](shots/86.PNG)
![Alt text](shots/87.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/ipv4.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/ipv4.yml
* ansible-playbook -i inventory/hosts playbooks/ipv4.yml        

![Alt text](shots/88.PNG)
![Alt text](shots/89.PNG)

## Conditionals

* In a playbook, you may want to execute different tasks, or have different goals, depending on the value of a fact (data about the remote system), a variable, or the result of a previous task. 
* You may want the value of some variables to depend on the value of other variables (or) you may want to create additional groups of hosts based on whether the hosts match other criteria. You can do all of these things with conditionals.

playbook : playbooks/factsdemo.yml

![Alt text](shots/90.PNG)
![Alt text](shots/91.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/factsdemo.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/factsdemo.yml
* ansible-playbook -i inventory/hosts playbooks/factsdemo.yml        

![Alt text](shots/92.PNG)

playbook : playbooks/combined.yml

---
- name: install lamp server on ubuntu
  hosts: redhat
  become: yes
  tasks:
    - name: install httpd server
      ansible.builtin.yum:
        name: httpd
        state: present
      when: ansible_facts['os_family'] == 'RedHat'
    - name: enable and start httpd
      ansible.builtin.systemd:
        name: httpd
        enabled: yes
        state: started
      when: ansible_facts['os_family'] == 'RedHat'
    - name: install httpd php server
      ansible.builtin.yum:
        name: php
        state: present
      notify:
        - restart apache
      when: ansible_facts['os_family'] == 'RedHat'
    - name: copy the info.php file in httpd
      ansible.builtin.copy:
        src: info.php
        dest: /var/www/html/info.php
      notify:
        - restart apache
      when: ansible_facts['os_family'] == 'RedHat'
    - name: update packages and install apache
      ansible.builtin.apt:
        name: apache2
        update_cache: yes
        state: present
      when: ansible_facts['os_family'] == 'Ubuntu'
    - name: install php packages
      ansible.builtin.apt:
        name:
          - php
          - libapache2-mod-php
          - php-mysql
        state: present
      notify:
        - restart apache2
      when: ansible_facts['os_family'] == 'Ubuntu'
    - name: copy the info.php page
      ansible.builtin.copy:
        src: info.php
        dest: /var/www/html/info.php
      notify:
        - restart apache2
      when: ansible_facts['os_family'] == 'Ubuntu'
  handlers:
    - name: restart apache
      ansible.builtin.systemd:
        name: httpd
        state: restarted
    - name: restart apache2
      ansible.builtin.systemd:
        name: apache2
        state: restarted

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/combined.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/combined.yml
* ansible-playbook -i inventory/hosts playbooks/combined.yml        

## NOPCOMMERCE INSTALLATION

 Manual steps (using apt package):

 => Install dotnet core7

    * wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    * sudo dpkg -i packages-microsoft-prod.deb
    * rm packages-microsoft-prod.deb
    * sudo apt-get update && \
      sudo apt-get install -y dotnet-sdk-7.0

 => Downloading nopcommerce zip and unzip

    * sudo apt install unzip -y
    * sudo mkdir /usr/share/nopCommerce
    * cd /usr/share/nopCommerce/
    * sudo wget https://github.com/nopSolutions/nopCommerce/releases/download/release-4.60.3/nopCommerce_4.60. 3_NoSource_linux_x64.zip
    * sudo unzip nopCommerce_4.60.3_NoSource_linux_x64.zip
    * sudo mkdir bin
    * sudo mkdir logs
    * cd ~
 => Creating user nop and giving full permissions

    * sudo adduser nop
    * sudo chgrp -R nop /usr/share/nopCommerce/
    * sudo chown -R nop /usr/share/nopCommerce/

=> Adding a service file

   * sudo vi /etc/systemd/system/nopCommerce.service
       
  ![Alt text](shots/51.PNG)

   * sudo systemctl enable nopCommerce.service
   * sudo systemctl start nopCommerce.service
   * sudo systemctl status nopCommerce.service

  ![Alt text](shots/52.PNG)
  ![Alt text](shots/53.PNG)

* Expose the nopcommerce with the < public_ip:5000>

  ![Alt text](shots/54.PNG)
 
playbook : playbooks/nop.yml

![Alt text](shots/62.PNG)
![Alt text](shots/63.PNG)
![Alt text](shots/64.PNG)

service-file : playbooks/nop.service

![Alt text](shots/97.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/nop.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/nop.yml
* ansible-playbook -i inventory/hosts playbooks/nop.yml

![Alt text](shots/95.PNG)
![Alt text](shots/96.PNG)

* Expose the '< public_IPaddress of node >:5000'

![Alt text](shots/98.PNG)

## OPEN-MRS INSTALLATION

=> URL : https://www.linuxcloudvps.com/blog/how-to-install-openmrs-on-ubuntu-20-04/

 Manual steps (using apt package): ( skipping mysql installation)

 * Switch as root user
 * apt-get update -y

 => Install Java 8

* apt-get install openjdk-8-jdk -y
* java -version

![Alt text](shots/101.PNG)
![Alt text](shots/102.PNG)

=> Installing tomcat7

* groupadd tomcat
useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
* wget https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.109/bin/apache-tomcat-7.0.109.tar.gz
* mkdir /opt/tomcat
tar -xvzf apache-tomcat-7.0.109.tar.gz -C /opt/tomcat/ --strip-components=1
* cd /opt/tomcat
chgrp -R tomcat /opt/tomcat
chmod -R g+r conf
chmod g+x conf
chown -R tomcat webapps/ work/ temp/ logs/

=> Create a Systemd Service File for Tomcat

* nano /etc/systemd/system/tomcat.service

![Alt text](shots/103.PNG)

* systemctl daemon-reload
* systemctl start tomcat
* systemctl status tomcat

![Alt text](shots/104.PNG)

* At this point, Tomcat is started and listens on port 8080

![Alt text](shots/105.PNG)

=> Install OpenMRS

* mkdir /var/lib/OpenMRS
chown -R tomcat:tomcat /var/lib/OpenMRS
* wget https://sourceforge.net/projects/openmrs/files/releases/OpenMRS_Platform_2.5.0/openmrs.war
* cp openmrs.war /opt/tomcat/webapps/
* chown -R tomcat:tomcat /opt/tomcat/webapps/openmrs.war
* Access OpenMRS using : 'http://your-server-ip:8080/openmrs'

![Alt text](shots/106.PNG)
![Alt text](shots/107.PNG)

 playbook : playbooks/openmrs.yml

![Alt text](shots/118.PNG)
![Alt text](shots/119.PNG)
![Alt text](shots/120.PNG)
![Alt text](shots/121.PNG)

 service-file : playbooks/tomcat.service

![Alt text](shots/122.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/openmrs.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/openmrs.yml
* ansible-playbook -i inventory/hosts playbooks/openmrs.yml

![Alt text](shots/116.PNG)
![Alt text](shots/117.PNG)

* Expose the application '<public_IPaddress:8080>'

![Alt text](shots/123.PNG)

     
## BROADLEAF INSTALLATION

=> URL :  https://github.com/BroadleafCommerce

 Manual steps (using apt package):

* sudo apt update
* git clone https://github.com/BroadleafCommerce/DemoSite.git

![Alt text](shots/108.PNG)

* sudo -i
* apt install maven -y
* mvn -version
* exit

![Alt text](shots/109.PNG)

* cd DemoSite/
* mvn clean install
* cd site/
* mvn spring-boot:run

![Alt text](shots/112.PNG)

* Expose the broadleaf with the '< public_IPaddress>:8080'

![Alt text](shots/110.PNG)
![Alt text](shots/111.PNG)

playbook : playbooks/broadleaf.yml

![Alt text](shots/113.PNG)
![Alt text](shots/114.PNG)

Jinja-service-file : playbooks/broadleaf.service.j2

![Alt text](shots/115.PNG)

Commands to execute :

* ansible-playbook -i inventory/hosts --syntax-check playbooks/broadleaf.yml
* ansible-playbook -i inventory/hosts --list-hosts playbooks/broadleaf.yml
* ansible-playbook -i inventory/hosts playbooks/broadleaf.yml

![Alt text](shots/125.PNG)


* Expose the '< public_IPaddress of node >:8080'



## VARIABLES

* Ansible uses variables to manage differences between systems. 
* With Ansible, you can execute tasks and playbooks on multiple different systems with a single command. 
* To represent the variations among those different systems, you can create variables with standard YAML syntax, including lists and dictionaries. 
* You can define these variables in your playbooks, in your inventory, in re-usable files or roles, or at the command line. 
* Not all strings are valid Ansible variable names. A variable name can only include letters, numbers, and underscores.

=> Let's take example of the 'combined.yml' file

1. Add variables in inventory and other in playbook

![Alt text](shots/126.PNG)
![Alt text](shots/127.PNG)
![Alt text](shots/128.PNG)

## ansible.builtin.package module – Generic OS package manager

* In most cases, you can use the short module name package even without specifying the collections: keyword. 

2. Using packaging module (Generic package manager) instead of using apt/yum packages for apache

* Using package instead of apt/yum for installing apache

![Alt text](shots/129.PNG)
![Alt text](shots/130.PNG)

## Loops

* Ansible offers the loop, with_<lookup>, and until keywords to execute a task multiple times. 
* Examples of commonly-used loops include changing ownership on several files and/or directories with the file module, creating multiple users with the user module, and repeating a polling step until a certain result is reached.

3. with loops and using loops from variables

![Alt text](shots/131.PNG)
![Alt text](shots/132.PNG)
![Alt text](shots/133.PNG)

=> Variables at the inventory level need not be in inventory file

4. with host_vars and group_vars

![Alt text](shots/134.PNG)
![Alt text](shots/135.PNG)
![Alt text](shots/136.PNG)
![Alt text](shots/137.PNG)
![Alt text](shots/138.PNG)

5.  to fail playbook explicitly on unsupported os

![Alt text](shots/139.PNG)

6. Debug messages

![Alt text](shots/140.PNG)
![Alt text](shots/141.PNG)