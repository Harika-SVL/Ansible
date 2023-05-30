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

* Now the public key to linuxmachine 'ssh-copy-id username@ipaddess'

![Alt text](shots/12.PNG)

* connect to the machine using private key 'ssh -i <path-to-private key> username@ipaddress'
* Generally private keys created will have extension of .pem i.e we create a Service account public and private key. 
* Copy the service account public key to all the servers and also disable the password based authentication

## Setting up sudo permissions
* We need to add devops user to the sudoers group (Wheel in Redhat server)
* We use 'sudo visudo' command

![Alt text](shots/13.PNG)

## Evironment setup (Installing and configuring) for Ansible

* Start two linux machines(vms) and rename them as Ansible control node and node(s)
* Create a service account/user('sudo adduser') as devops in all machines with sudo permissions('sudo visudo')
* Create a key pair ('ssh-keygen') in Ansible control node and copy the public key into the nodes from the control node ('ssh-copy-id username@ipaddress')

[ Optional: as both machines are in the same network we can also use private ipaddress of other machine to connect instead of public ipaddress ]
* Install Ansible on the control node using commands :
-> sudo apt update
-> sudo apt install software-properties-common -y
-> sudo add-apt-repository --yes --update ppa:ansible/ansible
-> sudo apt install ansible -y

[ Optional: Disable password based authentication ]
* Verify ansible version ('ansible --version')
* Adding inventory - Create a file named hosts ( 'vi hosts')with one entry <private ipaddress of the node>
* Check connectivity by executing 'ansible -m ping -i hosts all'






