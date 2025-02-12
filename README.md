# JenkinsCI-CDDocker
The objective is to implement the automation of the build and release process for their product
# The Problems faced
In this project a ticket booking company is facing problems due to using a waterfall method An entertainment company like BookMyShow where users book their tickets have multiple users accessing their web app. Due to less infrastructure availability, they use less machines and provide the required structure. This method includes many weaknesses such as:

Developers must wait till the complete software development for the test results.
There is a huge possibility of bugs in the test results.
Delivery process of the software is slow.
The quality of software is a concern due to continuous feedback referring to things like coding or architectural issues, build failures, test conditions, and file release uploads.
# The solution
The objective is to implement the automation of the build and release process for their product. This was achieved by

Setting up the Jenkins server in master or slave architecture
Using the Jenkins plugins to perform the computation part on the Docker containers
Creating Jenkins pipeline script
Using the GIT web hook to schedule the job on check-in or poll SCM
Building an image using the artifacts and deploy them on containers
Removing the container stack after completing the job

# steps that were involved 

# creating an ansible controller instance which runs an ansible-playbook to deploy a kubernetes cluster with one master and 2 node
# run the playbook to configure the master and nodes
# this cluster has a master and 2 nodes
# the master has jenkins installed on it,
# jenkins uses docker and maven to  build an image and deploys it to docker hub
# the pipeline also creates a container from this image and deploys that to the tomcat server on the worker node
# every time there is a github commit a new image is build and new container is deployed 
# after everything is done the container image and the containers should be destroyed


set up ec2 amazon linux instances
# controller
# t2 micro is sufficient
inbound ports
80
8080
443
22
# master
# has to have atleast 2 cpus t2 medium atleast
inbound ports
8080
TCP	Inbound	6443*	Kubernetes API server	All
TCP	Inbound	2379-2380	etcd server client API	kube-apiserver, etcd
TCP	Inbound	10250	kubelet API	Self, Control plane
TCP	Inbound	10251	kube-scheduler	Self
TCP	Inbound	10252	kube-controller-manager	Self
# Worker node(s)
# has to have atleast 2 cpus t2 medium atleast
inbound ports
8080
TCP	Inbound	10250	kubelet API	Self, Control plane
TCP	Inbound	30000-32767	NodePort Services†	All

# change hosts and permissions
sudo vi /etc/hosts
add all the hosts master and node
[master]
54.164.166.210 
[ansadmin]
54.164.166.212 
[worker]
54.164.166.211
[all]
54.164.166.210 
54.164.166.211

( change to ip address of different servers, ip r to get the address)

sudo vi /etc/ssh/sshd_config ( if you want to use password)
change #PermitRootLogin prohibit-password to Permitrootlogin yes
challengeresponseauthenitication no
change #passwordauthentication no to passwordauthentication yes esc :wq

sudo systemctl restart sshd

# from ansible controller

sudo su - 
adduser ansadmin 
passwd ansadmin 
give new password (for example welcome123)
sudo usermod -aG root ansadmin
visudo (to go into /etc/sudoers)
add under root
ansadmin ALL=(ALL) ALL
su - ansadmin
generate ssh keys
ssh-keygen (enter for all the questions)
cd .ssh
now you will see id_rsa.pub and id_rsa keys
# in the master
sudo su -
adduser master 
passwd master
give new password
sudo usermod -aG root master ( in case of ubuntu it is group sudo, in centos it is wheel)
visudo (to go into /etc/sudoers)(under root add)
master ALL=(ALL) ALL
su - master
# in the worker
sudo su -
adduser worker 
passwd worker
give new password
sudo usermod -aG root worker
visudo (to go into /etc/sudoers)(under root add)
worker ALL=(ALL) ALL
su - worker

# REMEMBER THE PASSWORDS

copy the public key to the hosts server
ssh-copy-id -i id_rsa.pub master@172.31.23.115 ( to copy to the master server)
ssh-copy-id -i id_rsa master@172.31.23.115 ( to copy private ip to the master server)

ssh-copy-id -i id_rsa.pub worker@172.31.23.115 ( to copy the tomcat server)
( to copy the files to the deploy server change it to your deploy pvt ip address )

# if you get an error on the password)
(
go to the server
go to root 
passwd master
change the password
and try again

or you can 
chmod 640 /etc/shadow
and change passwd
)

press yes enter password welcome123 key will be added
  
# try to ssh to see if you have connection
even ssh deploy@172.31.23.115 should work
ssh -i id_rsa deploy@172.31.23.115 ( to see if you can log in to the server change it your deploy nodes pvt ip address)
ssh -i id_rsa tomcat@172.31.23.115 ( to see if you can log in to the server change it your tomcat nodes pvt ip address)
yeah if you did !!

# now you can check if the authorized_keys exist in the .ssh folder here
cd .ssh
ls
exit (to go back to the ansible server VM)

# install ansible
sudo whoami
enter the password
sudo yum update
sudo amazon-linux-extras install epel
sudo yum install ansible
sudo vi /etc/ansible/hosts
add hosts
# check if you can ssh to the host file of /etc/ansible/hosts 
[master]
172.31.30.227 ansible_pass=master123 ansible_user=master

[worker]
172.31.19.177 ansible_pass=worker123 ansible_user=worker

# git install and clone
sudo yum install git 
git init
git clone https://github.com/nalapatt/ansible-k8s-setup.git


cd ansible-k8s-setup
vi hosts
- change the hosts
- [master]
172.31.30.227 ansible_pass=master123 ansible_user=master

[worker]
172.31.19.177 ansible_pass=worker123 ansible_user=worker
~                           
pwd
vi ansible.cfg
change and insert the path
inventory : pwd/hosts
esc :wq

# check ansible lists and connection
ansible master --list 
ansible worker --list 
ansible all --list 
ansible all -m ping 
(and ping if successful)
(should should show the respective hosts)

sudo passwd root
change the password for root the same as master and worker
so will work for all

# change the files so that it works on your system
ll - shows all the files
vi k8s-pkg.yml 
remove the firewall disable
remove the disable se linux
ansible-playbook k8s-pkg.yml --syntax-check
ansible-playbook k8s-pkg.yml  --extra-vars "ansible_sudo_pass=ansible123" ( set all three passwords to this)
if done YEAH!!!

# go into the respective terminals and in root add user to docker group

sudo usermod -aG docker master
sudo usermod -aG docker worker

# change yaml files 
vi k8s-master.yml
edit masters to master
change to your ip address of your master in apiserver-address=ipaddressof master
ansible-playbook k8s-master.yml --syntax-check
check the syntax if alright
then
ansible-playbook k8s-master.yml --extra-vars "ansible_sudo_pass=ansible123" 
if alright YEAAAAH!!!

# check if master is configured
kubectl get nodes (should see the master)
if error (localhost:8080 refused connection) then do this is root

mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
- y to overwrite config

# get the join command
edit the k8s-workers.yaml
change the hosts to master and ip address in hostsvars[]
ansible-playbook k8s-workers.yaml --extra-vars "ansible_sudo_pass=ansible123" 
if this is done 

# check the cluster nodes
kubectl get nodes
should show all the nodes

# # YEAH YOU HAVE DONE creating the kubernetes cluster

# now install jdk and jenkins on the master
# jdk
sudo yum update
su -c "yum install java-1.8.0-openjdk"
# jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum upgrade
sudo yum install chkconfig java-devel
sudo yum install jenkins

# give user docker permissions
sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo chkconfig docker on
sudo service docker start

# login to the jenkins UI
jenkins UIipaddressofjenkins:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
go to jenkins terminal
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
enter
the password that you see below copy and paste in browser
continue
update suggested plugins
update password
url suggestions default
save and finish
start using jenkins

# manage plugins
make sure these are installed
check in available plugins
git parameters plugin
ssh plugin
ssh agent
ssh build agent
ssh credentials
ssh pipeline
ssh connection
install without restart
# configure global
add maven
mymaven 
install automatically
git
default
/bin/git
save

# set up container server
# configure tomcat
sudo yum update
su -c "yum install java-1.8.0-openjdk"

sudo su -
adduser tomcat
passwd
enter password (remember this )
su - tomcat
# in master change the hosts file to include tomcat
vi /etc/hosts
[tomcat]
172.31.23.115
# copy the public key to tomcat user
cd .ssh from the ansible controller
ssh-copy-id -i id_rsa.pub tomcat@172.31.5.18

# check and change configurations for the tomcat UI
go to the tomcat server UI
tomcat 8 tar.gz right click copy link address ( https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz)
wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz
tar -xvzf apache-tomcat-8.5.66.tar.gz (the gz file)
mv apache-tomcat-8.5.66 tomcat
cd tomcat/conf
vi server.xml
change 8080 port to 8090
cd ../bin
./startup.sh
public ipaddress of node:8090
you should see the tomcat server
click manager app
you will see an error message
go back to the terminal
exit
find / -name context.xml
it shows locations of the files with that name ( for example for me i got
/root/tomcat/conf/context.xml
/root/tomcat/webapps/manager/META-INF/context.xml
/root/tomcat/webapps/examples/META-INF/context.xml
/root/tomcat/webapps/host-manager/META-INF/context.xml ) so i did
( vi /root/tomcat/conf/context.xml
vi /root/tomcat/webapps/manager/META-INF/context.xml
vi /root/tomcat/webapps/examples/META-INF/context.xml
vi /root/tomcat/webapps/host-manager/META-INF/context.xml )
cd to all the directories mentioned here one by one
if you see a valve line comment it out
vi context.xml
comment the lines
!
so '<!-- '
in the beginning
and in the end
' -->!'
this was found only in the 2nd and 4th file)
esc :wq
after that save and quit ! esc :wq
su - tomcat
cd ../conf
vi tomcat-users.xml
go to the role section and add
"<role rolename="tomcat"/>
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <role rolename="manager-script"/>
  <user username="admin" password="admin" roles="tomcat,manager-gui,admin-gui,manager-script"/> "
  without the " "
  
- save esc :wq - manager app sign in with admin and admin you are signed in
go to the tomcat UI manager app should show a sign in
yeah done


# add credentials for github https://github.com/nalapatt/dockeransiblejenkins.git
add credentials
username with password
username (github username)
password ( github password)
id github
description github
# add credentials for docker hub
username with secret text
secret ( docker hub password)
id docker-hub
description docker-hub

# add credentials for container deploy
ssh username with private key
dev-slave
dev-slave
worker
add private key from master
# configure jenkins ssh keys credentials
- manage jenkins 
- manage credentials
- Stores scoped to Jenkins
-  go to the global drop down box
-   and add credentials 
-   ssh username with private key 
-   id dev-server
-    description dev-server
-     username deploy 
-     private key enter directly 
-     add
-      go back to the terminal 
-      sudo su - 
-      su - jenkins
-       cd .ssh
-        cat id_rsa (get the private key copy and paste in the UI )
-        ok
-   id tomcat-server
-    description tomcat-server
-     username tomcat 
-     private key enter directly 
-     add
-      go back to the terminal 
-      sudo su - 
-      su - jenkins
-       cd .ssh
-        cat id_rsa (get the private key copy and paste in the UI )
-        ok
-        
# Create a new pipeline project
new item pipeline

add pipeline script ( JenkinsFile of https://github.com/nalapatt/JenkinsCI-CDDocker.git)

Poll SCM ( H/5* * * * )every 5 mins for new commit

Build now

If all the stages are done then success

check public ip:8090/dockeransible

should show the index.html

commit github changes and check if there is a new build

check the html again

YEAH PIPELINE DONE






# Step 1
# Create Instances
Create master and slave EC2 instances in AWS
- select Ubuntu 18.04 and default settings and launch instances

# Configure Master 
configure jenkins and docker in jenkins server
# Install docker in jenkins server
- sudo apt-get update 
- sudo apt install docker.io 
- docker --version

# start docker in jenkins
- sudo service docker start

# install jenkins and jdk in jenkins
- sudo apt-get update 
- sudo apt install openjdk-8* 
- sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
-  sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list' 
- sudo apt-get update 
- sudo apt-get install jenkins

# give user docker permissions
- sudo usermod -a -G docker jenkins
-  sudo service jenkins restart 
-  sudo service docker start

# configure ansible in ansible ec2 instance
- sudo apt-get update 
- sudo apt-get install software-properties-common 
- sudo apt-add-repository ppa:ansible/ansible 
- sudo apt-get update
-  sudo apt-get install ansible 
-  ansible --version (get the executable location /usr/bin/ansible ,
-   this will be used in the UI as /usr/bin/)


# change hosts and permissions
- sudo vi /etc/ansible/hosts 
- add 
-  54.164.166.210 ( change to ip address of deploy node, ip r to get the address)

- sudo vi /etc/ssh/sshd_config ( if you want to use password)
- change #PermitRootLogin prohibit-password to Permitrootlogin yes
- change #passwordauthentication no to passwordauthentication yes esc :wq
- sudo systemctl restart sshd

# login to the jenkins UI
- jenkins UIipaddressofjenkins:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
- go to jenkins terminal
-  sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
-  enter
-  the password that you see below copy and paste in browser 
-  continue 
-  update suggested plugins 
-  update password 
-  url suggestions default
-  save and finish
-  start using jenkins

# manage plugins
# make sure these are installed
check in available plugins
- ansible plugin
- ssh plugin
-  ssh agent 
-  ssh build agent
-  ssh credentials
-  ssh pipeline
-  ssh connection 
- install without restart

# set up container server
# configure tomcat 
- sudo apt-get update 
- sudo apt install openjdk-8*
- sudo su - 
- adduser tomcat
- passwd
- enter password (remember this )
- su - tomcat
# go to the tomcat server UI 
- tomcat 8 tar.gz right click copy link address ( https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz) 
- wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz 
- tar -xvzf apache-tomcat-8.5.66.tar.gz (the gz file)
- mv apache-tomcat-8.5.66 tomcat
- cd tomcat/conf
- vi server.xml
- change 8080 port to 8090
- cd ../bin
- ./startup.sh
- public ipaddress of node:8090 
- you should see the tomcat server 
- click manager app 
- you will see an error message
- go back to the terminal
- exit 
- find / -name context.xml 
- it shows locations of the files with that name
( for example for me i got
- /root/tomcat/conf/context.xml
- /root/tomcat/webapps/manager/META-INF/context.xml
- /root/tomcat/webapps/examples/META-INF/context.xml
- /root/tomcat/webapps/host-manager/META-INF/context.xml
)
so i did
- ( vi /root/tomcat/conf/context.xml
- vi /root/tomcat/webapps/manager/META-INF/context.xml
- vi /root/tomcat/webapps/examples/META-INF/context.xml
- vi /root/tomcat/webapps/host-manager/META-INF/context.xml
)
- cd to all the  directories mentioned here one by one 
- if you see a valve line comment it out
- vi context.xml
- comment the lines
-  <!--   <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->!
- so '<!-- '
-  in the beginning 
-  and in the end 
-  ' -->!'
- this was found only in the 2nd and 4th file)
- esc :wq
- after that save and quit ! esc :wq
- su - tomcat
- cd ../conf
- vi tomcat-users.xml
- go to the role section and add
- 
<role rolename="tomcat"/>
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <role rolename="manager-script"/>
  <user username="admin" password="admin" roles="tomcat,manager-gui,admin-gui,manager-script"/>
- save esc :wq
- manager app sign in with admin and admin you are signed in

- go to the tomcat UI manager app should show a sign in
- yeah done

# Install docker 
- exit
- sudo apt-get update 
- sudo apt install docker.io 
- docker --version

# start docker 
- sudo service docker start

# install jenkins and jdk in jenkins
- sudo apt-get update 
- sudo apt install openjdk-8* 
  
# set up user in container server

- sudo su - 
- adduser deploy 
-  passwd
-  welcome123 (remember this password) enter for all and press Y for correct again please remember your username and password
- sudo vi /etc/ssh/sshd_config ( if you want to use password)
- change #PermitRootLogin prohibit-password to Permitrootlogin yes
- change #passwordauthentication no to passwordauthentication yes esc :wq
- sudo systemctl restart sshd
# give user docker permissions
- sudo usermod -a -G docker deploy
-  sudo service docker start

# in jenkins UI set up config to accept password and hosts file to configure hosts
- configure system to check ssh connection with password
-  select configure system
-  go to add ssh remote hosts 
-  hostname - pvt ip address of deploy node ( you can get this with ip r)
-   port 22
-    add jenkins 
-    credentials
-     username with password 
-     username - deploy (the deploy server username you set up ) 
-    password - welcome123 ( the password that you gave for that user) 
-    id - deploy 
-    description - deploy 
-    add 
-    select deploy in drop down 
-    leave the other boxes blank 
-    test connection 
-    if passed then connection works from the jenkins user

# from jenkins user ssh keygen
- sudo su - 
- adduser jenkins ( already exists) 
- passwd 
- give new password (for example welcome123)
- su - jenkins

# generate ssh keys
- ssh-keygen (enter for all the questions)
- cd .ssh
-  now you will see id_rsa.pub and id_rsa keys

# copy the public key to the hosts server
- ssh-copy-id -i id_rsa.pub deploy@172.31.23.115 ( to copy to the deploy server)
- ssh-copy-id -i id_rsa.pub tomcat@172.31.23.115 ( to copy the tomcat server)
- ( to copy the files to the deploy server change it to your deploy pvt ip address ) 
- press yes enter password welcome123 key will be added

# check if you can ssh to the host
- ssh -i id_rsa deploy@172.31.23.115 ( to see if you can log in to the server change it your deploy nodes pvt ip address)
- ssh -i id_rsa tomcat@172.31.23.115 ( to see if you can log in to the server change it your tomcat nodes pvt ip address)

# yeah if you did !!
# now you can check if the authorized_keys exist in the .ssh folder here 
- cd .ssh
- ls
- exit (to go back to the jenkins server VM) 

# configure jenkins ssh keys credentials
- manage jenkins 
- manage credentials
- Stores scoped to Jenkins
-  go to the global drop down box
-   and add credentials 
-   ssh username with private key 
-   id dev-server
-    description dev-server
-     username deploy 
-     private key enter directly 
-     add
-      go back to the terminal 
-      sudo su - 
-      su - jenkins
-       cd .ssh
-        cat id_rsa (get the private key copy and paste in the UI )
-        ok
-   id tomcat-server
-    description tomcat-server
-     username tomcat 
-     private key enter directly 
-     add
-      go back to the terminal 
-      sudo su - 
-      su - jenkins
-       cd .ssh
-        cat id_rsa (get the private key copy and paste in the UI )
-        ok
# add credentials for github
- add credentials 
- username with password 
- username (github username)
-  password ( github password)
-   id github
-    description github

# add credentials for docker hub
- username with secret text
-  secret ( docker hub password)
-   id docker-hub 
-   description docker-hub

# Create a new pipeline project
- new item pipeline
- add pipeline script

- Poll SCM ( H/5* * * * )every 5 mins for new commit
- Build now
- If all the stages are done then success
- check public ip:8090/dockeransible
- should show the index.html
- commit github changes and check if there is a new build
- check the html again

YEAH PIPELINE DONE

