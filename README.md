# PG DO - Configuration Management with Chef, Puppet and Ansible
CI/CD Deployment Using Ansible CM Tool

## DESCRIPTION

You are a DevOps engineer at XYZ Ltd. Your company is working on a Java application and wants to automate WAR file artifact deployment so that they don’t have to perform WAR deployment on Tomcat/Jetty web containers. Automate Ansible integration with Jenkins CI server so that we can run and execute playbooks to deploy custom WAR files to a web container and then perform restart for the web container.

## Steps to Perform:

1. Configure Jenkins server as Ansible provisioning machine
2. Install Ansible plugins in Jenkins CI server
3. Prepare Ansible playbook to run Maven build on Jenkins CI server
4. Prepare Ansible playbook to execute deployment steps on the remote web container with restart of the web container post deployment

## Procedure:

Created two AWS EC2 Instances
- One for Jenkins/Ansible
- One for Tomcat

copy .pem file into vm for running ansible
`scp -i aws.pem aws.pem ubuntu@[aws ip address]:~/`

change permissions of .pem file
`chmod 600 aws.pem`

### **Install Java and Maven**
```
sudo apt -y update
sudo apt install default-jre
sudo apt install maven
java -version
mvn --version
```

### **Install Ansible**
```
sudo apt -y update
sudo apt -y install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt update
sudo apt -y install ansible```
```
### **Install AWS Ansible Plugin**
`ansible-galaxy collection install amazon.aws`
- Edit `/etc/ansible/ansible.cfg`
```
[defaults]
enable_plugins = aws_ec2
host_key_checking = False
pipelining = True
remote_user = ec2-user
private_key_file=~/aws.pem
```

### **Provision Tomcat EC2**
Replace IP address and .pem filename below

Use `tomcat_provision.yaml` from this repo
Create an inventory file:
```
sudo nano inventory.ini

[aws]
3.94.116.75

[aws:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/aws.pem
```
Then run: `ansible-playbook tomcat_provision.yaml -i host.ini` to install and configure Tomcat on other VM

### **Install Jenkins**
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins

### **Start and Configure Jenkins**
sudo systemctl start jenkins
sudo systemctl status jenkins

Go to `hostname:8080` to set up jenkins

Get the password and copy it to the webpage:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Go to "Manage Plugins" and install the following plugins:
- SSH
- Ansible

Go to "Manage Credentials":
- Add a global credential with:
   - **Kind**: SSH Username with private key
   - **ID**: aws2
   - **Description**: SSH credentials to communicate with Tomcat EC2
   - **Username**: ubuntu
   - Enter private key directly. Copy past .pem file contents into the text area.

Go to "Configure System":
- Under "SSH remote hosts" click on "Add" button
  - **Hostname**: Amazon EC2 IP
  - **Port**: 22
  - **Credentials**: ubuntu (SSH credentials to communicate with Tomcat EC2)
  - **serverAliveInterval**: 0
  - **timeout**: 0

Go to Global Tool Configuration:
- Add JDK
  - **Name**: jdk-11
  - **JAVA_HOME**: `/use/lib/jvm/java-11-openjdk-amd64/bin/java`

- Add Maven
  - **Name**: local-mvn
  - **MAVEN_HOME**: `/usr/bin/mvn`

Create Pipeline project
- Choose Pipeline script from SCM
- **SCM**: Git
- **Repo URL**: `https://github.com/jdvalera/devops-project-4`
- **Branch**: `*/main`

Build Project
