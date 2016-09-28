#Digital Ocean - getting started

###What is Digital Ocean
- DigitalOcean offer what is known as an iaas platform (Infrastructure as a Service) 
  - An empty fresh (Linux) server
- More complex and more freedom than:
  - paas (Platform as a Service) where software is installed for us (e.g. Openshift)
- We can design the server exactly how we want it
- We have to learn some Linux in the process
- We will definetely feel some frustration in the process
- All the work will be done in a comando prompt



##Ready - Set - Go
- Use git bash (if not allready installed get it [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git))  




### Generate a SSH key
- Use git gui for this



###Sign up on Digital Ocean
Do it from [here](https://www.digitalocean.com)
1. Choose a size (5$/mo)
2. Set billing alerts
	- go to Settings > Billing > Billing Alerts 
3. Create Ubuntu droplet
4. Choose datacentre (Frankfurt or Amsterdam)
5. Add your SSH key (or more keys if more users share the droplet)



##Setup the Ubuntu server 
Based on this [guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)  



### Login with root
- `ssh root@138.68.84.251`
- This will work if you have setup the SSH key



### Create a new user with admin rights
- Do this because we should never use 'root'-user in normal activities
- Run this:
  - `adduser <username>` (Choose any username here)
  - `usermod -aG sudo <username>` (this adds root priveleges to the new account)



### Add Public Key Authentication for the New User
- copy the same public key (as used on D.O. website) into clipboard
- On the droplet write:
  - `sudo - <username>`  (We are now in the new users home directory)
- Create a new directory: (.ssh)
	- `mkdir ~/.ssh`
	- `chmod 700 ~/.ssh` (Grant all access for current user)
- Create a new file: authorized_keys
	- `nano ~/.ssh/authorized_keys` (Creates file and open with texteditor: nano)
	- Insert the public key and hit ctrl-x (press y and enter)
	- Add more keys if you need to (e.g. to have access from different computers)
- Restrict the permissions of the authorized_keys
	- `chmod 600 ~/.ssh/authorized_keys`
	- `exit` (to go back to root)
- Now from a different terminal try logging in with
	- `ssh <username>@<ip number>`
[Reference on ssh](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)



###Disable Password Authentication 
- With a text editor open file: sshd_config
	- `sudo nano /etc/ssh/sshd_config`
	- Find this line: #PasswordAuthentication 
	- Uncomment it and change its value to no.
	- The following default values must not be changed
		- `PubkeyAuthentication yes`
		- `ChallengeResponseAuthentication no`
	- Exit and safe (CTRL-x and press y plus enter)
- Reload SSH daemon
	- `sudo systemctl reload sshd`
- Open new terminal and test that login with SSH is still working


### Install java
Logged in as the new user:
- `sudo apt-get update`
- `sudo apt-get install default-jdk`
- `java -version` (to test that java is now installed)


##Tomcat installation
Tomcat is the webserver we use


### Install
- Change to root priveleges:
	- `sudo su `
	- `apt-get install tomcat8 tomcat8-admin`
	- `apt-get install haveged` (This saves time see [reference](http://www.issihosts.com/haveged/)


### Create tomcat admin user
- `sudo nano /etc/tomcat8/tomcat-users.xml`
- Insert at the end of the xml file:  
```
<role rolename="manager-gui"/>  <user name="admin" password="XXX" roles="manager-gui, manager-script"/>
```
- (remember to change the password)  
- Save the file (CTRL-x -> y -> enter) and restart Tomcat
	- `service tomcat8 restart`
- Tomcat is now running on http://<Server-IP>:8080/manager
- Check it from a browser!!


### Change Tomcat default port from 8080 to 80
- `sudo nano /etc/tomcat8/server.xml`
- Replace Connector port="8080" with Connector port="80" in line 71
- Save file
- `sudo nano /etc/default/tomcat8`
- Change `#AUTHBIND=no` to `AUTHBIND=yes` in line 46 (Remember to remove #)
- Save file
- Restart Tomcat
	- `service tomcat8 restart`



### Test deployment of new webapplication
- In browser go to 
	- `http://<yourIP>/manager`
	- find section: 'WAR file to deploy'
	- Deploy a war file from a working webapplication
	- Verify that it is added to the list of Applications
- Test the web application from a browser


### Deploy from Maven
- Add plugin to the POM.XML:  
```
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>  
    <artifactId>tomcat7-maven-plugin</artifactId>  
    <version>2.2</version>  
    <configuration>  
       <url>http://46.101.196.110/manager/text</url>  
       <path>/demo</path>    
       <username>admin</username>  
       <password>XXX</password>   
     </configuration>   
</plugin>
```  
- Execute the Maven goal
- [See more here](http://tomcat.apache.org/maven-plugin-2.0/tomcat7-maven-plugin/)


### Logfiles
- `sudo su`
- `cd /var/log/tomcat8`
- `ls -ar` (to see all files)
- `cat <filename>` to see content of files
- `tail -f catalina.out` (to see the continued output from tomcat log)
- From another terminal restart tomcat:
	- `service tomcat8 restart`
	- From this terminal: `clear` and `tail -f localhost.XXXXXXX.log` (XXXXX is the newest log)
- Looking at both the browser and the 2 terminal windows do this:
	- Run a webapplication that has 2 servlets:
		1. has a `system.out.println("Msg: A");`
		2. throws a runtime exception like `int x = 100 / 0;`
		- See what happens in the 2 log terminals


## Install MySQL server
