# Deploying-Wordpress-with-AWS

A warm welcome of you all, Here I’ll gonna cover a “HOW WE CAN DEPLOY WORDPRESS OVER THE AWS EC2 INSTANCE AND RDS SERVICE OF THE AWS”. Integration of “AWS CLOUD” + “WORDPRESS” + “MYSQL DATABASE” + “ANSIBLE” + “WEB-SERVER”, trust me you gonna like this practical.


Let’s first see, what task we have to perform actually:-
TASK DESCRIPTION:-
🔅 Create an AWS EC2 instance
🔅 Configure the instance with Apache Webserver.
🔅 Download php application name “WordPress”.
🔅 As wordpress stores data at the backend in MySQL Database server. Therefore, you need to setup a MySQL server using AWS RDS service using Free Tier.
🔅 Provide the endpoint/connection string to the WordPress application to make it work.
In this blog, I have done all this Steps using Ansible. Just by running single Ansible-playbook, all this steps automatically configure. Stay Tuned..
Before directly jumping to the Practical Part, first go through the Pre-Requisite to understand this Task.
PRE — REQUISITE:-
To understand the complete steps of this Task, you should have some basic knowledge of AWS Cloud EC2 Instance, Webserver, AWS RDS service, Wordpress, Ansible ROLE.
For doing this task, I have used RHEL 8 Linux Operating system as my Controller Node of Ansible. To make it easier I have used Putty program to run my Operating System.
I have installed Ansible, boto and boto3 tools inside the Controller Node of ansible. Command:-
pip3 install ansible
pip3 install boto
pip3 install boto3
Make Sure you should have account on AWS Cloud.
PRACTICAL:-
Let’s start running the commands & writing the codes…
STEP-1:- Create one workspace or folder inside your Controller Node using this command:-
mkdir /task18
Now all the Files and Roles we will create inside this workspace.
STEP-2:- Inside this workspace, create one folder named as “/roles” , and inside this folder run following commands. This commands will create the Ansible Roles inside this directory-
cd /task18
mkdir /roles
cd /roles
ansible-galaxy init ec2-instance
ansible-galaxy init wordpress
ansible-galaxy init rds
I have Created three Ansible Roles. One for Launching AWS EC2-instance, Second for Configuring Wordpress over Instance and third one is for Creating the RDS (DataBase).
STEP-3:- Go to EC2->KEY-PAIRS service and create one new Key Pair with whatever the name you wanna give let’s say- “task19.pem” download it and put it on our workspace. And run this command:-
chmod 400 task19.pem


STEP-4:- Login to your AWS Account. Search for IAM Service of AWS, service is used to create the New User of our account. Click on Add User and give some limited power and create it. It will provide the Access Key and Secret Key of the user that we will gonna use to log in inside our account using ansible. Download the user credential file.


STEP-5:- We gonna create one local configuration file inside “task18” folder & whatever Ansible commands we want to run in future we will run on this folder. Because then only Ansible will be able to read this Local configuration file & can work accordingly.
Create “ansible.cfg” file inside the “task18” directory and Put this content on it:-
vim ansible.cfg


Here some common key-word you can see like “host_key_checking”, “roles_path”, “ask_pass”, etc. You should be familiar with this all common keywords as I already mentioned in Pre-Requisite.
Here is the “private_key_file” keyword is used for aws key pair. When Ansible gonna login to AWS instances to setup K8s via SSH, then it needs the private key file. Also the default remote user of EC2 Instance is “ec2-user”.
STEP-6:- Create one vault file of Ansible, where we gonna put all our user credentials like “access_key” and “secret_key”, that ansible use while login. Command:-
ansible-vault create cred.yml
It will ask you to create password, create it and put the credential like that:-
access_key: XXXXXXXXXX
secret_key: XXXXXXXXXXX
Now we are ready to work on Roles.
Your Managed Node looks like this:-


STEP-7:- Go to “/roles/ec2-instance/tasks” and start editing the “main.yml” file. Here we gonna put all our code for launching the EC2-Instance. Code:-


Have used many variables in this code, value of this variables I putted inside the “/ec2-instance/vars/main.yml”. Screen Shot of this file you will find at last of this step.
Here I have used “ec2” module of ansible to launch the EC2-INSTANCE. You might be familiar with all the properties that I used inside the ec2 module, if you worked in AWS Cloud before. So I skipping the explanation of this properties.
Here I used “register” property to store the Output of above module “ec2” in variable named as “instance”.
Also used “add_host” module of ansible. This module will dynamically create the Host Group while running the playbook with the name that we provided “groupname”. Used JSON parsing to fetch the IP of the instances that it launched in “host”.
Here I used “debug” module of ansible, just for printing the value of the variable. You can skip this part.
At last have used “wait_for” module, it stops the program till the Public DNS name of the Slave Instance will not come. As we know that the Instance Launching takes some time, that’s why I used this module. It will wait till the SSH comes up.
“cd /task18/roles/ec2-instance/vars”
vim main.yml


The value of this variable you will find in AWS Cloud Console.
STEP-8:- Go to “/role/wordpress/tasks” and start editing the “main.yml” file. Here we gonna put all the code for configuration of “Wordpress”, code:-


Here I have used “package” module of ansible, to install the Dependencies of Wordpress (mysql and httpd webserver).
Next we have to put wordpress files over the Apache Default directory (/var/www/html). Either we can download the particular file using “get_url” module of ansible over the directory. But here I’m using “unarchive” module of ansible, which first copy the file from Managed Node to the Target Node and then Untar it. The file present inside the “/wordpress/files” directory.
Next for working with this wordpress version, we need Php 7.4 version in particular OS. So for Enabling the 7.4 version, I have used this command. Using “command” module.
Using “package” module, I have installed all the Php software’s that require for working of wordpress.
Next using “service” module, starting the webserver service.
ALL ROLES AND FILES, YOU WILL FIND IN MY GIT HUB REPO. LINK IS PRESENT AT LAST OF THIS BLOG.
STEP-9:- Go to “/roles/rds/tasks” and start editing the “main.yml”. Here we gonna the code for creating the RDS. Code:-


Here I have used many variables, values of this variables are present inside “/rds/vars/main.yml” file.
Using “rds” module of Ansible, I have created the RDS Data Base. Here I’m using MySQL Database as my Engine name. I assume that from rest of the Key-words you are already familiar, as I already mentioned in Pre-requisite.
“/rds/vars/main.yml” -


STEP-10:- Now it’s time to create final playbook that will gonna run all our Roles. Create one playbook inside the workspace “task18”.
vim execute.yml


Using the “hostname” we gonna run all the roles.
STEP-11:- Command to run the playbook:-
ansible-playbook execute.yml — ask-vault-pass
If your Roles don’t have any error and all the Steps you configured properly, then you will get this output:-




Finally our playbook has run successfully, you can go inside the AWS Cloud and see the instance and RDS has launched or not. let’s go:-
STEP-12:- Here you can see that EC2-INSTANCE has launched and one RDS Database has created successfully:-




STEP-13:- Let’s go inside the wordpress instance, to see whether the wordpress is configured successfully or not:-


Here we can see that all this are configured successfully.
STEP-14:- Now using the Public Ip of the instance over the browser, we can see the wordpress site. URL:-
http://<PUBLIC_IP>/wordpress
First, you have to provide the credentials of Database that you wanna use. So put the DATABASE NAME, USERNAME, PASSWORD that we created inside the RDS configuration file. This credentials:-


Next, inside the LOCALHOST put the “ENDPOINT URL” of your RDS Database. That you will find inside the RDS console
