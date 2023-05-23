###  Integrating a basic java application from github using Jenkins and Deploying to Tomcat Server  

In this exercise, a basic maven-based  java application is pulled from the github using jenkins(using the option source code management- git and give the url) and maven-build is done. The web page script is shown below
##### ![00a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/00a-theWebPageToBeDeployedInGithub.PNG) 

The built war file available in the jenkins server is deployed to tomcat server and the web page is accessed.  

Two azure virtual machines are launched. 
1)  to install and configure tomcat and   
2)  to install and configure jenkins. 
##### ![00b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/00b_twoServers.PNG)

In the tomcat server, appropriate modifications are done to access managerapp. Also required users and roles are created and provided to jenkins instance which authenticates jenkins to deploy the application to tomcat server.

In the jenkins server, both git and maven are installed. Maven is java based build tool allowing to run various targets and based on specified goals, it will compile, run some tests and build a war file. Git is a source code management tool from which the required java application codes are pulled so that it can be package into a war file using maven build and then deployed to tomcat server. 


### Step 1) Tomcat SetUp

1) Launch an Azure VM an open port number 8080 on it.The following commands are executed by switching to root use  

2) Install java-openjdk-11 version and check the version
##### ![03](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/03_javaforTomcat.PNG)  

3) In the /home/azureuser/,wget <url of tomcat zip file(apache-tomcat-9.0.65 is used)>  

4) unzip and untar using tar -xvzf 'zip file name' and rename to tomcat for convenience
##### ![04](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/04_wgetAndExtractTomcat.PNG)
##### ![05](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/05_RenameTomcat.PNG) 

5) go to tomcat/bin which contains startup.sh and shutdown.sh 

6) Start the service by giving ./startup.sh. 
##### ![06a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/06a_startTomcat.PNG)  

We can access tomcat by giving the public ip of server at 8080 as shown.  
##### ![06b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/06b-AccessTomcat.PNG)  

But when clicked on managerapp , it throws 403 permission denied error as shown 
##### ![06c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/06c-WithOUTConfiguring.PNG)  
a) By default, the Manager is only accessible from a web browser which runs on the same machine as Tomcat.Comment out the loop back address in both apache-tomcat-9.0.65/webapps/host-manager/META-INF/context.xml and apache-tomcat-9.0.65/webapps/manager/META-INF/context.xml as shown.
##### ![07a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/07a-edit-TwoContextFiles.PNG)  

Restart the server. Now it will not throw the error and sign in page appears as shown.  
##### ![07c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/07c-NoError-ManagerAppClick.PNG) 
The username and password required for sign in are configured in the next step. 

b) The credentials of tomcat-users need to be updated to login via mangerapp gui.So we need to edit conf/tomcat-users.xml as shown.


```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="admin" roles="manager-gui, manager-script"/>
<user username="deployer" password="deployer" roles="manager-script"/>
<user username="tomcat" password="s3cret" roles="manager-gui"/>
```
  
The roles manger-gui and manger-script are edited and users(tomcat,deployer,admin respectively) are assigned these roles.manger-script role allows programmatic access to tomcat server. Here it helps enables jenkins to copy files from jenkins server and deploy it to tomcat server. manager-gui is just for testing purpose to see if we an access the web gui properly.

7) Restart the server. From the web browser sign in as "tomcat" with the configured password . It opens the home page as shown below.
##### ![07f](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/07f-signinAStomcat.PNG)
##### ![07g](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/07g-SignIn%20AS-tomcatUser.PNG)  

### Step 2) Jenkins SetUp

1) Launch Azure VM (CentOS-8.5).Open ports 80 and also 8080 for jenkins in the securitygroup.  

2) Connect to it using putty or mobaxterm

3) check the linux distro name and version by cat /etc/os-release 

4) update and upgrade by sudo yum update and sudo yum upgrade(yum is the package manager)

5) for jenkins installation on centOS , follow the official documentation (https://pkg.jenkins.io/redhat-stable/)  

a)create repository named jenkins.repo in /etc/yum.repos.d  
b) edit this file and give the gpgkey = "https://pkg.jenkins.io/redhat-stable/jenkins.io.key". If this is not done, it throws 'error 14-gpgkey not found'.Else specify  --nogpgcheck while installing jenkins.
     (https://unix.stackexchange.com/questions/207907/how-to-fix-gpg-key-retrieval-failed-errno-14)  
c)install java-11-openjdk  
d)install jenkins  
e)start the service by sudo service jenkins start  
f)In the web browser , give the public ip of the server and port number(8080). Do the initial necessary pre-settings in the jenkins dashboard.  

6) To login into jenkins, use admin and the password obtained by cat /var/lib/jenkins/secrets/initialAdminPassword
##### ![02](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/02_JenkinsStart.PNG)

7) Install some of the basic plugins like GitHub Integration Plugin, Maven integration Plugin,publish over ssh etc.
 
### Step3)Configure jenkins server for integrating with git & maven and building a war file  

In the jenkins server, both git and maven are installed.  

1) yum install git -y and check version
2) To install the maven,  
     a)cd to /opt  

     b)wget 'url of the maven tar.gz' 

     c)extract it using tar -xvzf 'name of tar.gz package'  

     d)rename for convenience using mv apache-maven-3.8.3 maven  

     e)The command ./mvn -v(checking the maven version) works only if we are inside /maven/bin. The command wont work outside the bin directory.So set the path variable in the  .bash_profile  

     f)Give the paths for M2, M2_HOME and JAVA_HOME as shown  

         M2_HOME=/opt/maven (folder where the maven is unzipped) 
         M2=/opt/maven/bin (maven executables are in this path)
         JAVA_HOME=(the path of java in usr/lib/jvm/java-11-openjdk properly).Find it by the command "find / -name java-11*". Copy paste the path here and give all three values in the PATH variable as shown
     ##### ![08](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/08_PathinBashProfile.PNG)   

     g)Execute "source .bash_profile" so that the new path variable will be updated. Also this avoids logging out and re-logging into the server as root user which is otherwise required to update the new PATH variable. 

     h)In the global tool settings of jenkins dashboard, add maven and jdk and set the path for JAVA_HOME and MAVEN HOME(Get the respective paths by giving echo $JAVA_HOME and $M2).Apply and save. In the build section, the root pom.xml will be automatically prompted since it reads the pom.xml from the github. Specify goal as  'clean install'. Save and apply.  

3) The built war file can be seen in  /var/lib/jenkins/workspace/<jobname>/webapp/target in the jenkins server and in the jenkins dashboard as well as shown below.
##### ![09c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/09c-WarInWorkSpaceJenkins.PNG) 
##### ![09d](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/09d-warin%20JenkinsServer.PNG)   



### Step4) Deploy to tomcat Server  
1) Install "deploy to container" in jenkins dashboard.  

2) Select maven project. In the build section specify the goal as clean install.  

3) Configure the previous steps again.In the global credentials, add credentials with the option of username and password. It should match with those provided in the tomcat-users.xml file. In the post build actions,select deploy war/ear file to container and give the credentials properly(add credentials-username with password,given in the tomcat-users.xml file). 
Give the url of the server as shown.
##### ![09e](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/09e-credentialsAddedinJenkins.PNG)  

Save and apply.The jenkins job can be seen as successfully finished as shown.
##### ![10a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/10a_WarFileTransfertoTomcat.PNG)  

4) On the completion of deployment, we can see  the war file in the tomcat server and the folder webapp in the gui.
##### ![10b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/10b-warFileonTomcat.PNG)
##### ![10c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/10c-TomcatAdminPage.PNG)  

On clicking it, index page is displayed.
##### ![10c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/jenkinsTomcat/10d-appOnTomcat.PNG)
   













