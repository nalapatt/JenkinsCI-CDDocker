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
- find / -name context.xml 
- it shows locations of the files with that name
( for example for me i got
/root/tomcat/conf/context.xml
/root/tomcat/webapps/manager/META-INF/context.xml
/root/tomcat/webapps/examples/META-INF/context.xml
/root/tomcat/webapps/host-manager/META-INF/context.xml
)
so i did
( vi /root/tomcat/conf/context.xml
vi /root/tomcat/webapps/manager/META-INF/context.xml
vi /root/tomcat/webapps/examples/META-INF/context.xml
vi /root/tomcat/webapps/host-manager/META-INF/context.xml
)
- cd to all the  directories mentioned here one by one 
- if you see a valve line comment it out
- vi context.xml
- comment the lines
-  <!--   <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->!
- so <!--  in the beginning and in the end  -->!
- i found this only in the 2nd and 4th file)
- esc :wq
- after that save and quit ! esc :wq

- cd ../conf
- vi tomcat-users.xml
- go to the role section and add
- press i
- <role rolename="manager-gui"/>
- <user username="tomcat" password="s3cret" roles="manager-gui"/>

- <role rolename="manager-script"/>
- <user username="deployer" password="deployer" roles="manager-script"/>

- save esc :wq
- manager app sign in with tomcat and s3cret you are signed in

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
- ssh-copy-id -i id_rsa.pub deploy@172.31.23.115 
- ( to copy the files to the deploy server change it to your deploy pvt ip address ) 
- press yes enter password welcome123 key will be added

# check if you can ssh to the host
- ssh -i id_rsa deploy@172.31.23.115 ( to see if you can log in to the server change it your deploy nodes pvt ip address)

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
- Copy 
- pipeline{
    agent any
    tools {
      maven 'mymaven'
    }
    environment {
      DOCKER_TAG = 2.0
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/nalapatt/dockeransiblejenkins'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t nalapatt123/hariapp:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u nalapatt123 -p ${dockerHubPwd}"
                }
                
                sh "docker push nalapatt123/hariapp:${DOCKER_TAG} "
            }
        }
     
        stage('Docker Deploy'){
            steps{
                sshagent(['dev_server']) {
                     sh 'ssh -o StrictHostKeyChecking=no deploy@172.31.26.33 "docker run -p 8080:8080 -d --name my-app nalapatt123/hariapp:${DOCKER_TAG}"'
                }
                  }
        }
    }
}
- Poll SCM ( H/5* * * * )every 5 mins for new commit
- Build now
- If all the stages are done then success
- check public ip for index.html 
- commit github changes and check if there is a new build
- check the html again
- 
