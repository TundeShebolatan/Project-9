# Project 9: TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Step 0 - This project is a continuation of Project-8, so all configured NFS, web servers and Mysql db, server instances must be up and running as shown on the EC2 dashboard below;

![server instances running](image/instances.PNG)

Step 1 – Install Jenkins server

Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

Install JDK (since Jenkins is a Java-based application)

`sudo apt update`

`sudo apt install default-jdk-headless`

![spin up instance for ubuntu server and name it Jenkins](image/spin-up-ubuntu-server-Jenkins.PNG)

![update Jenkins server](image/update-Jenkins-server.PNG)

![Install default-jdk-headless](image/install-jdk-on-Jenkins-server.PNG)

Install Jenkins

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add - sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \ /etc/apt/sources.list.d/jenkins.list'`

`sudo apt update`

`sudo apt-get install jenkins`

![Install Jenkins](image/Install-Jenkins.PNG)

Make sure Jenkins is up and running

`sudo systemctl status jenkins`

![Jenkins status](image/Jenkins-active-and-running.PNG)

By default Jenkins server uses TCP port 8080 – create a new Inbound Rule in your EC2 Security Group

![Jenkins server uses TCP port 8080](image/Edit-inbound-rules-Jenkins-server.PNG)

Perform initial Jenkins setup.
From your browser access  

`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

![Create first Admin user](image/create-first-admin-user-Jenkins.PNG)

![Setting up Jenkins](image/Instance-Configuration-Jenkins.PNG)

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Install suggested plug-ins

![Install suggested plug-ins](image/installing-suggested-pluggings-Jenkins.PNG)

![Jenkins set up complete](image/Jenkins-is-ready.PNG)

![Jenkins Dashboard](image/Jenkins-dashboard.PNG)

Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks

Enable webhooks in your GitHub repository settings

![Enable webhooks in your GitHub](image/adding-webhhook-in-github.PNG)

Click "New Item" and create a "Freestyle project" on the Jenkins web console

![create a Freestyle project](image/freestyle-project9.PNG)

In configuration of Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![Update SCM and credentials](image/udpate-SCM-and-credentials.PNG)

Save the configuration and try to run the build

![Run fist build](image/first-build.PNG)

Open the build and check in "Console Output" if it has run successfully

![Fist build successful](image/first-build-successful.PNG)

Click "Configure" your job/project and add these two configurations:

Configure triggering the job from GitHub webhook

![Github hook trigger for GITScm polling](image/Configure-build-triggers.PNG)

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts"

![Post-build Actions](image/Post-build-Actions.PNG)

Make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch

![Edit README file in GitHub repository](image/make-changes-github-repo.PNG)

A new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server

![Second build successful](image/second-build-successful.PNG)

One have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub)

By default, the artifacts are stored on Jenkins server locally

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH:

Now the artifacts are saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Install "Publish Over SSH" plugin

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item. On "Available" tab search for "Publish Over SSH" plugin and install it

![Install Publish Over SSH extension](image/Publish-Over-SSH.PNG)

Configure the job/project to copy artifacts over to NFS server

On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server

![Input SSH server credentials](image/input-ssh-server-credentials.PNG)

![Input the source files](image/input-source-files-ssh-server-jenkins.PNG)

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections

The test turned out unsuccessful due an error

![Test unsuccessful](image/error.PNG)

The error was solved by creating password for the NFS server as a root user in the home directory, and then input the password in the SSH server configuration, and rerun the test.

![Create password for the NFS server](image/NFS-Sever.PNG)

![Install Publish Over SSH extension successful](image/Publish-Over-SSH-successful.PNG)

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.

![Send all files produced by the build into our previously define remote directory](image/Post-build-Actions.PNG)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

![Edit README file](image/make-changes-github-repo.PNG)

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

SSH: Transferred 25 file(s)
Finished: SUCCESS

![successfully copied artifacts over to NFS server](image/Files-successully-copied-to-NFS-via-SSH-Jenkins.PNG)

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file

`cat /mnt/apps/README.md`

![README.MD file updated in NFS server](image/copied-readme-files-from-github-on-NFS.PNG)
