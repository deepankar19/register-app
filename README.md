# Deploy to Kubernetes Using Jenkins | End to End DevOps Project | CICD

#Git
#AWS
#Jenkins
#SonarQube
#Ansible
#Docker
#Kubernetes
#ArgoCD

# prerequisites aws, dockerhub, github account credentials
# Install and Configure the Jenkins-Master and Jenkins-Agent
# Integrate Maven to Jenkins and Add GitHub Credentials to Jenkins
# Create Pipeline Script(Jenkinsfile) for Build & Test Artifacts and Create CI Job on Jenkins
# Install and Configure the SonarQube
# Integrate SonarQube with Jenkins
# Build and Push Docker Image using Pipeline Script
# Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl
# ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD
# Configure ArgoCD to Deploy Pods on EKS and Automate ArgoCD Deployment Job using GitOps GitHub Repository
# DONE - Verify CI/CD Pipeline by Doing Test Commit on GitHub Application Repo
#=====================================================================================================================

# Install and Configure the Jenkins-Master and Jenkins-Agent

- Jenkins-Master
- Jenkins-Agent

# install and configure Jenkins-Master

> > sudo apt update and sudo apt upgrade -y
> > sudo nano /etc/hostname

    xx.xx.xx.xx ---> Jenkins-Master

> > sudo init 6 {Reboot}
> > sudo apt install openjdk-17-jre -y
> > java --version
> > sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
> >  https://pkg.jenkins.io/debian/jenkins.io-2023.key
> > echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
> >  https://pkg.jenkins.io/debian binary/ | sudo tee \
> >  /etc/apt/sources.list.d/jenkins.list > /dev/null
> > sudo apt-get update
> > sudo apt-get install jenkins

> > sudo systemctl enable jenkins {start jenkins server every reboot}
> > sudo systemctl start jenkins
> > sudo systemctl status jenkins {show jenkins info details}

# generate ssh and add in jenkins agent

> > sudo nano /etc/ssh/sshd_config

- pubkeyauthentication yes {uncomment it}
- Authorized keyfile .ssh/authorizedkey {uncomment it}
  > > sudo service sshd reload {Reload ssh service}
  > > ssh-key -t ed25519
  > > cd .ssh/ > cat id_rsa.pub > copy id_rsa.pub > JENKINS-AGENT > /home/ubuntu/.ssh/ > paste to authorized_keys

# "add port 8080 in outbonds for access jenkins"

> > jenkins-dashboard > Manage jenkins > node >BUilt-in Node >> configure

- Number of executors: 0
  > > save  
  > > jenkins-dashboard > Manage jenkins > node > New node
- Node name: Jenkins-Agent
- Type: Permanent Agent
  > > create
- Name: Jenkins-Agent
- Description: Jenkins-Agent
- Number of executors: 2
- Remote root Directory: /home/ubuntu
- labels: Jenkins-Agent
- Usage: Use this node as much as possible
- launch method: launch agents via SSH
- Host: <private-ip-Jenkins-Agent>
- Credentials: ubuntu(Jenkins-Agent)
  add > jenkins credentials provider:jenkins
  - Kind: SSH Username with private key
  - ID: Jenkins-Agent
  - Description: Jenkins-agent
  - Username: ubuntu
  - private Key: copy & paste Jenkins-Master Node .ssh/id_rsa private key
    > > Add
- HostKey Verfication Strategy: Non verification Strategy
  > > SAVE

create a new job {test pipeline} and add piple script {hello-world}
for checking connectivity between master node and agent is successfull

---

# install and configture Jenkins-Agent

> > sudo apt update && sudo apt upgrade -y
> > sudo nano /etc/hostname

    xx.xx.xx.xx ---> Jenkins-Agent

> > sudo init 6 {Reboot}
> > sudo apt install openjdk-17-jre -y {install java}
> > java --version

# install docker in jenkins agent

> > sudo apt install docker.io -y
> > sudo usermod -aG docker $USER
> > sudo inti 6 {reboot}

# generate ssh and add in jenkins agent

> > sudo nano /etc/ssh/sshd_config

- pubkeyauthentication yes {uncomment it}
- Authorized keyfile .ssh/authorizedkey {uncomment it}
  > > sudo service sshd reload {Reload ssh service}

---

## Integrate Maven to Jenkins and Add GitHub Credentials to Jenkins

> > Manage Jenkins > plugins > available plugins

- Maven Integartion
- Pipline Maven Integration
- Eclipse Termurin installer

# configture these plugins

> > Manage Jenkins > Tools

- Maven installations
- Name: Maven3
  - install automatically
- JDK installations
- JDK: Java17
- install automatically: install from adoptium.net > version > jdk-17.0.5+8
  => add github credentails to jenkins
  => Manage Jenkins > Credentials > Add credentails
  _ Kind: username with password
  _ username: <github*username>
  * password: <github*persornal_access_token>
  => how to get github_personal_access_token
  => github account > settings > Developer settings > Personal access token > Token(classic) > generate token > copy it
  * ID: github \* Description: github
  > > create

---

## Create Pipeline Script(Jenkinsfile) for Build & Test Artifacts and Create CI Job on Jenkins

> > Github repo{register-app} > create Jenkinsfile

     pipeline {
      agent { label 'Jenkins-Agent' }
       tools {
            jdk 'Java17'
            maven 'Maven3'
            }
        stages{
            stage("Cleanup Workspace"){
                    steps {
                    cleanWs()
                    }
            }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/<github-username>/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
    }

}

----------------------------------------------------------------------------------------------------

## Install and Configure the SonarQube

> > sudo apt update && sudo apt upgrade -y

# Create the file repository configuration:

> > sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:

> > wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null

# Update the package lists:

> > sudo apt-get update

# Install the latest version of PostgreSQL.

# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':

> > sudo apt-get -y install postgresql postgresql-contrib

> > sudo systemctl enable postqresql
> > sudo passwd postgres
> > su - postgres
> > createuser sonar
> > psql
> > ALTER USER sonar WITH ENCRYPTED password 'sonar';
> > CREATE DATABASE sonarqube OWNER sonar;
> > grant all privileges on DATABASE sonarqube to sonar;
> > \q
> > exit

# install Adoptium libary in sonar machine

> > sudo bash
> > apt install -y wget apt-transport-https gpg
> > wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor | tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null
> > echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

# install java in sonar machine

> > apt update
> > apt install temurin-17-jdk
> > update-alternatives --config java
> > /usr/bin/java --version

# linux terminal tuning

# increase the limit

> > sudo vim /etc/security/limits.conf/
> > end of file -> add lines
> > sonarqube - nofile 65536
> > sonarqube - nproc 4096

# increase the mapped memory regions

> > sudo vim /etc/sysctl.conf
> > end of file -> add lines
> > vm.max_map_count = 262144
> > sudo init 6 {Reboot}

# Add port 9000 to inbound rules for sonarqube access

> > sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.065466.zip
> > sudo apt install unzip
> > sudo unzip sonarqube-9.9.0.65466.zip -d /opt
> > sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube

# create sonarqube user and set permission for user

> > sudo groupadd sonar
> > sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
> > sudo chown sonar:sonar /opt/sonarqube -R

# update the sonarqube properties with database credentials

> > sudo vim /opt/sonarqube/conf/sonar.properties

# uncomment line

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar

sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

# create service for sonar file

> > sudo vim /etc/systemd/system/sonar.service
> > [Unit]
> > Description=SonarQube service
> > After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

# start the sonarqube & enable the service

> > sudo systemctl start sonar
> > sudo systemctl enable sonar
> > sudo systemctl status sonar

# watch log and status of system

> > sudo tail -f /opt/sonarqube/logs/sonar.log

http://<sonarqube-public-ip>:9000
user:- admin
pass:- admin

---

## Integrate SonarQube with Jenkins

> > sonarqube homepage > account > security > Generate Tokens

- Name: jenkins-sonarqube-token
- Type: Global Analysis Token
- Expires in: No expiration
  > > click on generate & copy generate token to local system
  > > Goto jenkins dashboard > manage jenkins > credentials > Add credentials >
  - Kind: Secret text
  - Secret: "paste recent cretaed token"
  - ID: jenkin-sonarqube-token
  - Description: jenkins-sonarqube-token
    > > > click on create

# install and configure plugins for sonarqube

> > Manage Jenkins > plugins > Avaliable plugins

       * SonarQube Scanner
       * Sonar Quality Gates
       * Quality Gates
       >> install and restart jenkins
    >> Manage Jenkins > System > SonarQube installations
       * Name: sonarqube-server
       * Server URL: http://<sonar-private-ip>:9000
       * Server authentication token: select jenkins-sonarqube-token
       >> apply&save
    # add sonarqube scanner into jenkins
      >> Manage Jenkins > Tools > SonarQube Scanner installations
       * Name: sonarqube-scanner
       * install automatically
       >> apply&save
    => add stage to (github > jenkinsfile) pipline
    => after stage("Test Application")
    => add stage("SonarQube Analysis")
       * credentialsId: 'jenkins-sonarqube-token'
    # sonarqube webhook
     >> sonarqube dsahboard > Administartion > configration > webhooks
       => create
       * Name: sonarqube-webhook
       * URL: http://<jenkins-private-ip>:8080/sonarqube-webhook/
        >> create
    # pipeline for quality gates
    => add stage("Quality Gate")
       * credentialsId: 'jenkins-sonarqube-token'

## Build and Push Docker Image using Pipeline Script
