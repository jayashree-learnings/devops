### jenkins-ansible-tomcat  

In this project,  a .war file in jenkins is built by integrating the code from github and building it using jenkins. Using jenkins job ( jenkins dashboard), two things are done  
1) the built .war file is copied to ansible and   
2) an ansible playbook is executed.  

In the ansible playbook, the instruction is to copy the .war file from ansible to tomcat so that when the jenkins executes the playbook,the .war file will be copied from the ansible server to tomcat server.  

### Step 1) jenkins and tomcat set up  

Do the basic setup of jenkins and tomcat as in the first project by launching 2 Azure VMs and doing the required basic configurations.  

### Step 2) ansible server installation 

For the ansible server, launch a anew Azure VM(CentOS-8-free). It runs on port number 22 itself.So no need of adjusting its default network security group.Do the required installations  
1) yum install python3  
2) yum install python3-pip  
3) pip install ansible.check its version  

### Step3) Configuration of ansible server  

1)Create a user "ansadmin" and set password for that user.  

2)Use the command visudo to edit the sudoers file and give "anasadmin ALL=(ALL) NOPASSWD:ALL". This user can execute commands like a root user using the sudo command and without being prompted for the password.
##### ![01a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/01a_visudofile.PNG)   

3)In /etc/ssh/sshd_config, enable "PasswordAUthentication yes" and disable "PasswordAUthentication no".Else it throws 
error , while trying to copy the private key into another server.reload sshd after this.
##### ![01b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/01b_EnablePasswdAuthentication.PNG)   

4)In tomcat server also,create ansadmin user amd modify his credentials in the same manner. Both the servers now have this common user.  

4)In the ansible-server,login as the ansadmin.generate the key by using the command ssh-keygen and copy it to the tomcatserver.It will
prompt for the ansadmin password.The user  "ansadmin" could login into the tomcat server now.
##### ![01c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/01c_CopyKey.PNG)   

5)Exit from tomcatserver and in /etc/ansible/hosts, give the groupname as [webservers] and provide the private ip of tomcat.
##### ![02a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/02a-ansibleHosts-giveTomcatPrivateIp.PNG)   

6)Test the connection between the two servers using the command, "ansible all  -m ping"
##### ![02b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/02b-pingTomcat.PNG)   

7)Write the playbook(/opt/playbooks/copyfile.yml) to copy the war file from ansible to tomcat
##### ![02c](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/02c-Ansibleplaybook.PNG)  

### Step4)Configuration of jenkins  

1)Install the plugin "publish over ssh".  

2)In manage jenkins-configuration system, in the SSH servers, for the ansible-server use password based authentication and give the credentials of the user ansadmin.Test the configuration from the dashboard itself.
##### ![03a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/03a_ansibleServerConfigurationINJenkinsDashboard.PNG)  

3)Create a new free style job and given maven goal as clean install. In post build action,   
1) choose "send files/execute commands over ssh". For the file transfer,specify the source and destination  

2) Take one more window for the the post build action and give exec command to execute the playbook "ansible-playbook /opt/playbooks/ccopyfile.yml".  

Both post build actions are shown below. copying to ansible 
##### ![03b1](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/03b1_postBuild-CopyToAnsible.PNG)  

executing the playbook
##### ![03b2](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/03b2_postBuild-executePlaybook.PNG)   

  3) Apply,save and build the job.  
If executed properly without errors, the "success"  message can be seen in the jenkins dashboard. 
##### ![04a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/04a-FileToAnsibleServer.PNG)
##### ![04b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/04b-executeAnsiblePlayBook.PNG)  

First the .war files get copied from jenkins to ansible-server(/opt/playbooks/webapp/target/webapp.war. jenkins will create the folders "webapp/target/webapp.war") in the ansible server as shown below.  
##### ![05a](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/05a-FileInAnsibleServer.PNG)  

The playbook copyfile.yml will be executed then and hence this .war file now present in ansible will be copied to /opt/apache-tomcat-9.0.65/webapps folder of the tomcat server as shown below.  
##### ![05b](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/05b-warFileCopiedToTomcatServer.PNG)    

  4) On the successful completion of jenkins job, the web page can be accessed by using the public ip of tomcat server as shown.
##### ![06](https://github.com/jayashree-learnings/devops/blob/main/00_includes/02-jenkinsAnsibleTomcat/06-DisplayWebPage.PNG)























