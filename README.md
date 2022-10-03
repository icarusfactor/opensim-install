# opensim-install
Steps to setup a full opensim system. NOTE: Server Operating system is CentOS7.9 

**SSH :**
 Setup SSH keys for login and quick access to *screen to control
administration actions not done by web interface. 
If you wish to have a key setup follow the below instructions. 

https://www.ssh.com/academy/ssh/keygen

**Screen:**
 We will have to share a screen with the user that is running the opensim binary
so screen will need to be used, tmux is standard in newer OS’s but does not have 
screen sharing.  

**HOWTO INSTALL SCREEN:**

```
yum update -y
yum install screen -y
screen -v
Screen version 4.01.00devel (GNU) 2-May-06
```

**Apache 2.4  :**
 A standard Apache install will work for what we want. Nothing special
 will be required. We will use it for the opensim users to easily create and
 manage their avatar and profile.
We will also use a custom interface for managing the opensim configuration
files. 

**HOWTO INSTALL APACHE:**

```
yum update -y
yum install httpd
systemctl start httpd
systemctl enable httpd.service
```


**PHP version 7.4 :** 
 This is the stable version and will be used for Wordpress and OSWebManager
to help manage and use Opensim. The only non-default module needed is mysql

```
yum install php php-mysql
systemctl restart httpd.service
```


**Mysql 10.3 :**
 The is the stable version of mysql and will work with opensim and OSWM.
 No special setups need to be made so a basic install will work. 

**HOWTO INSTALL MYSQL:**
```
rpm --import https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
yum install MariaDB-server galera MariaDB-client MariaDB-shared MariaDB-backup MariaDB-common
systemctl enable mariadb
systemctl start mariadb
mysql_secure_installation 
 And answer to questions in wizard:
 Switch to unix_socket authentication [Y/n] Y
 Change the root password? [Y/n] Y
 New password: Type new root password
 Re-enter new password: Re-enter new root password
 Remove anonymous users? [Y/n] Y
 Disallow root login remotely? [Y/n] Y
 Remove test database and access to it? [Y/n] Y
 Reload privilege tables now? [Y/n] Y
```

**Mono version. 6.12:**
 This is the system and libraries that OpenSim uses to run. 

```
rpmkeys --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"

yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
su -c 'curl https://download.mono-project.com/repo/centos7-stable.repo | tee /etc/yum.repos.d/mono-centos7-stable.repo'
yum clean all
yum makecache
yum install -y mono-complete
```

**SystemD:**
 Will use systemd to setup screen and automate starting and stopping
 progmatically while being able to attach to it as root or user. You can
 change the username from dyount which is my username to your.

```
cd /etc/systemd/system/

[root@host dyount]# vim ./opensim.service
# Systemd Service Unit for OpenSimulator 
#
[Unit]
Description=OpenSimulator service
After=syslog.target network.target
 
[Service]
User=root
#Group=dyount
Type=simple
LimitSTACK=1048576
TimeoutStopSec=60
WorkingDirectory=/home/dyount/opensim.spotcheckit.org/opensim/bin

ExecStart=/bin/screen -S Opensim_1 -dm bash -c "ulimit -s 1048576;/bin/mono --desktop -O=all /home/dyount/opensim.spotcheckit.org/opensim/bin/OpenSim.exe;/bin/screen -S Opensim_1 -X multiuser on;/bin/screen -S Opensim_1 -X acladd dyount"
ExecStop=/usr/bin/screen -d -r Opensim_1 -X stuff "shutdown" ; /usr/bin/screen -d -r Opensim_1 -X eval "stuff ^M"
KillMode=none

 
[Install]
WantedBy=multi-user.target

```

**OpenSim:**
 This is the base component written in Mono and will need 
configuration and can be configured and we will install it inside the opensim users account.  

http://opensimulator.org/wiki/Download
http://opensimulator.org/dist/opensim-0.9.2.1.tar.gz

Change directory to opensim user directory example in my case /home/dyount/opensim.spotcheckit.org/
```
mkdir opensim
cd opensim 
tar -zxvf opensim-0.9.2.1.tar.gz
The binaries and startup files are located in the /bin directory. 
```

**OSWM:**
This user-friendly web interface will access the remote XML RPC port to change
 configurations on the opensim setup. 

Original code: This only works with php 5.x I will upload my version that works with 7.x
but making sure  the ways it gets to the configuration files works cleanly, I had to patch 
the default directory, see if I do not need to do this. 

https://github.com/Nino85Whitman/OpenSim-Manager-Web-V5


**Wordpress:**
The base CMS will create a basic layout for working with the opensim interface. 

**Wordpress plugin:**
w4os – OpenSimulator Web Interface :
This will create an interface for users to create profiles and avatars 

https://wordpress.org/plugins/w4os-opensimulator-web-interface/

**Cool VL Viewer**
This is the client side program that users run to control their avatars. Others exist,this one works well for personal grids.
http://sldev.free.fr/
http://sldev.free.fr/binaries/CoolVLViewer-1.30.0.18-Linux-x86_64-Setup

**LETS BEGIN THE SETUP**

Now the we have all of the basic programs needed for the our system setup, first
we will setup the opensim binary to get the base system up and running.

Change directory to the Apache users DOCROOT to start from, but not install
Wordpress yet, in here we will untar the file as the user as follows
"tar -zxvf opensim-0.9.2.1.tar.gz". We will also change the name to its simpler
form without version. "mv opensim-0.9.2.1.tar.gz ./opensim". Now change directory
to the opensim directory and then into the bin directory.  

**Files that need to be changed from default settings:**

The inital configuration file. 
```
BASEPATH/opensim/bin/OpenSim.ini
```

Change then name of the constant host name prompt display, I used the IP address.
Enable saving the history of the console file and how many lines of it and if it was timestampped. I disabled crashes , dont need to enable this until problems start occuring and need to debug the issue and I have enabled the PID file for the main binary. No limit set by default, but always good to set a small and large prime size limit to limit any crashes from excessively large prims or prims you can not find in your map. Also allow 
prims to be bound by the physics engine or not.  

Changes in the [Const] section.
```
BaseHostname = "69.167.171.208"
```

Changes in the [Startup] section.
```
ConsolePrompt = "Region (\R) "
ConsoleHistoryFileEnabled = true
ConsoleHistoryFileLines = 100
ConsoleHistoryTimeStamp = true
save_crashes = false
PIDFile = "/tmp/OpenSim.exe.pid"
PhysicalPrimMin = 0.01
PhysicalPrimMax = 64
physical_prim = true
```

Personal settings for the Estate creditials and the UUID if left as zeroed will be randonly generated. 

Changes in the [Estates] section. 
```
DefaultEstateName = My Estate
DefaultEstateOwnerName = FirstName LastName
DefaultEstateOwnerUUID = 00000000-0000-0000-0000-000000000000
DefaultEstateOwnerEMail = factor@userspace.org
DefaultEstateOwnerPassword = password
```

Changes in the [Messaging] section. While this is optional it will enable Offline messaging service and uses mysql for storage.  
```
  OfflineMessageModule = "Offline Message Module V2"
  StorageProvider = OpenSim.Data.MySQL.dll
  MuteListModule = MuteListModule
  ForwardOfflineGroupMessages = true
```

Changes in the [RemoteAdmin] section. This will be needed for our custom web interface to manage the grids configuration files from it and even restart the service.  
```
 enabled = true
 port = 0
 access_password = "pAs$w0Rd"
```

Changes in the [Groups] section. This is part of and used by the messaging service. 
```
Enabled = true
LevelGroupCreate = 0
HomeURI = "http://69.167.171.208:9000"
MessagingEnabled = true
MessagingModule = "Groups Messaging Module V2"
DebugEnabled = false
```

This is the default setting so no change is needed , just to make aware of
how it connected to the rest of the system.
```
Include-Architecture = "config-include/Standalone.ini"
```

We will be using the mysql database instead of the default sqlite. No changes need to be made yet.
```
BASEPATH/opensim/bin/config-include/Standalone.ini
```

Not changed but default ,to show its connection to the next. This is the end of the file.
```
Include-Common = "config-include/Standalone.ini"
```



The changes here will reflect the switch from default SQLite to mysql.
By selecting Standalone.ini it will also need StandaloneCommon.ini
says do not hcange any option in Standalone.ini but I ended up chaning the null to mysql since we were using mysql.
```
BASEPATH/opensim/bin/config-include/Standalone.ini
```

Enabling the file Standalone.ini also enables the below file. 
```
BASEPATH/opensim/bin/config-include/StandaloneCommon.ini
```

Changes made to [DatabaseService] section. Comment out the below line to disable SQLite as we will be using mysql instead.
```
 ;Include-Storage = "config-include/storage/SQLiteStandalone.ini";
```

Uncomment the below lines and setup the database credtials for opensim. 
```
 StorageProvider = "OpenSim.Data.MySQL.dll"
 ConnectionString = "Data Source=localhost;Database=dyount_opensim;User ID=dyount_opensim;Password=PaSsW0Rd;Old Guids=true;SslMode=None;"
```
 
Changes made to [GridService] section. Comment out the NULL storage and uncomment the mysql source.   
```
;StorageProvider = "OpenSim.Data.Null.dll:NullRegionData"
Uncomment or add:
StorageProvider = "OpenSim.Data.MySQL.dll:MySqlRegionData"
```

Changes made in [GridInfoService] section. Give the grid a name and nick name.
```
gridname = "spotcheckit.org"
gridnick = "spottygrid"
```


Now that we have setup the base opensim system we need to make sure the SystemD file is in place to start the server side program and an empty database is setup that uses the credtials in the [DatabaseService] section of the configuration file. 

Start the SystemD service and enter the inital database settings.
```
service opensim start
screen -r Opensim_1
```

Attaching the opensim screen session for the first time you should see in the Console.
```
=====================================

We are now going to ask a couple of questions about your region.

You can press 'enter' without typing anything to use the default

the default is displayed between [ ] brackets.

=====================================

New region name []: shutdown
RegionUUID [c4508773-b09c-4e8a-98f2-41f560f504e8]: 
```

Press enter for the random UUID and then for region location and internal IP and internal port and to reoslve hostname.Then enter the IP address of the server. 
```
Region Location [1000,1000]:
Internal IP address [0.0.0.0]:
Internal port [9000]:
Resolve hostname to IP on start (for running inside Docker) [False]:
External host name [SYSTEMIP]: 69.167.171.208
```

After entering the IP address it should finish starting up the service and filling the database that was created for this service. Next we will create the user on the console and setup the client binary to create a login for the grid. One of the users wa screate already by the Estate section. To exit the sessionto the main console press CTRL+A+D


**Creating Opensim User**
If you had left the screen session be sure to get back into it, so we can create the first user for the grid from the opensim console. We will run through some of the basic grid commands to see what the settings are currently for the grid. 
```
screen -r Opensim_1
```

First we will run the show users command.
```
Region (shutdown) # show users
Root agents in region shutdown: 0 (root 0, child 0)
```
Next we will see if logins are enabled
```
Region (shutdown) # login status
Login in shutdown are enabled
```

Then we will check the name of the Estate
```
Region (shutdown) # estate show 
Estate information for region shutdown
Estate Name          ID      Owner               
My Estate            101     Daniel Yount
```

To view the basic data settings for the region.
```
Region (shutdown) # show region
Region information for shutdown
Region ID                  : c4508773-b09c-4e8a-98f2-41f560f504e8
Region handle              : 1099511628032000
Region location            : 1000,1000
Region size                : 256x256
Maturity                   : 0
Region address             : http://69.167.171.208:9000/
From region file           : ./Regions/Regions.ini
External endpoint          : 69.167.171.208:9000
```

Now we will create the second user which will be a normal user. The first user was created by the Estate section.
```
Region (shutdown) # create user Joe Smith
Password:     <- Enter actual password
Email []: factor@userspace.org
User ID (enter for random) []: 
Model name []: 
18:51:16 - [AUTHENTICATION DB]: Set password for principalID c1ab1be8-cb43-4550-bb14-2e51c10468ec
18:51:16 - [GRID SERVICE]: GetDefaultRegions returning 0 regions
18:51:16 - [USER ACCOUNT SERVICE]: Unable to set home for account Joe Smith.
18:51:16 - [USER ACCOUNT SERVICE]: Created user inventory for Joe Smith
18:51:16 - [USER ACCOUNT SERVICE]: Creating default appearance items for c1ab1be8-cb43-4550-bb14-2e51c10468ec
18:51:16 - [USER ACCOUNT SERVICE]: Creating default avatar entries for c1ab1be8-cb43-4550-bb14-2e51c10468ec
18:51:17 - [USER ACCOUNT SERVICE]: Account Joe Smith c1ab1be8-cb43-4550-bb14-2e51c10468ec created successfully
```

Now we will exit the screen session CTRL+A+D and install the Cool VL viewer as the client to access our new Grid. We downloaded the binary to our desktop, now we will run the binary so it installs. Change directorty to the viewer and run the binary file. 
```
bash ./CoolVLViewer-1.30.0.13-Linux-x86_64-Setup  
cd CoolVLViewer
./cool_vl_viewer
```

Once the viewer starts.
  Click on the menu "Edit" option and then select "prefrences". 
  
  Now make sure the tab "grids list" is highlighted.
  
  In the Login URI enter the IP address with port http://69.167.171.208:9000
  
  Then click "get paramaters". It may try to change the port to 8002 , change it back to 9000 or it wont work.
  
  Then if everything works you should see the grid name show up in the "Grid Name:" section.
  
  Then click over into the list and select "Add New Grid" it should be in the list at this time and able to pick and log in as a user to.
  
  Fill out the "First name:" and "Last name:" and password of the user. Click login.
  
  When you first log in, errors dialog boxes will pop up about no inventory and region data. Just click through these as it will create this initial data and you wont have to click through them again.
  
  Now you have your avatar enabled and active to walk/fly around the region. 

**Terrain changes water levels**

Back to the server side within the screen command we will download and install custom terrains. We will also change the default water level. 
```
screen -r 43305.Opensim_1
```
From the opensim console we will change the default water level wihich is 20m to 25m.  
While logged into the opensim server with our user avatar when the command is issued you will immedialy see the level of water change. 
```
set water height 25
```

Now we will create a directory for our custom terrains. Then download a few free terrians and load them while we are in the viewer.  
In the DOCROOT of your opensim account. Not within the opensim directory. 
```
mkdir terrain
cd terrain
```

Now we will download and test three free terrain files from. 
https://terrains.dynamicworldz.com/Terrain-Size-256/
```
wget https://terrains.dynamicworldz.com/gallery/dw-flat-starter.zip
wget https://terrains.dynamicworldz.com/gallery/dw-island1x1-7.zip
wget https://terrains.dynamicworldz.com/gallery/dw-island1x1-9.zip
unzip *.zip
```

Now we will load all three of these custom terrains.Lets log back into the opensim Console and type below with full path to the terrain files.  
```
screen -r 43305.Opensim_1
terrain load /home/dyount/opensim.spotcheckit.org/terrains/dw-island1x1-7.r32
terrain load /home/dyount/opensim.spotcheckit.org/terrains/dw-flat-starter.r32
terrain load /home/dyount/opensim.spotcheckit.org/terrains/dw-island1x1-9.raw
```
After loading each you can move your avatar around to see it was immediately change around the avatar. 

**Backup and Restore regions** 

Now we will create a directory for our backups of opensim. 
```
mkdir backups
```
To save the current opensim setup type:
```
Region (shutdown) # save oar /home/dyount/opensim.spotcheckit.org/backups/region.oar
20:13:19 - [ARCHIVER]: Writing archive for region shutdown to /home/dyount/opensim.spotcheckit.org/backups/region.oar
20:13:19 - [ARCHIVER]: Creating version 0.8 OAR
20:13:19 - [ARCHIVER]: Added control file to archive.
20:13:19 - [ARCHIVER]: Writing region shutdown
20:13:19 - [ARCHIVER]: 0 region scene objects to save reference 0 possible assets
20:13:19 - [ARCHIVER]: Adding region settings to archive.
20:13:19 - [ARCHIVER]: Adding parcel settings to archive.
20:13:19 - [ARCHIVER]: Adding terrain information to archive.
20:13:19 - [ARCHIVER]: Adding scene objects to archive.
20:13:19 - [ARCHIVER]: Saving 0 assets
20:13:19 - [ARCHIVER]: Finished writing out OAR for shutdown
```

Now that we have our region saved, lets change it by loading another terrain and see if we can activaley load the saved OAR file. 
```
terrain load /home/dyount/opensim.spotcheckit.org/terrains/dw-flat-starter.r32
```
Now after you view the change terrain happen, load up the previous state. 
```
load oar /home/dyount/opensim.spotcheckit.org/backups/region.oar
```

After running the above command the previous state will be activated again. Now you know how to save and restore your world. 


**Installing OSMW Opensim Manager Web Interface**

While managing your world with the Opensim Console is fine, having a browser based manager will make it much easier to keep it managed and you can immedialty see textures and maps  not just the data. 

I have upgraded the original OSMW as it was limited to the End Of Life PHP5.x and made it work with PHP7.x and maybe even 8.x. As account of user change directory to the Opensim DOCROOT of the account. In here we will use git clone to grab OSMW. 
```
 git clone https://github.com/icarusfactor/OpenSim-Manager-Web-V5.git
 ln -s /home/dyount/opensim.spotcheckit.org/OpenSim-Manager-Web-V5 osmw
```

Now that the code is in place we need to install the additional database tables that will be used by OSMW into the opensim standard database as root user.  
```
cd /osmw/docs/sql
mysql dyount_opensim  < ./database.sql 
mysql dyount_opensim < ./database_NPC.sql
```

Six new tables will be created in the database called config,moteurs,users and with the experimental npc tables gestionnaire,inventaire,npc. 

Now we can proceed to the installation of OSMW by going to the URL of the install. 
```
https://opensim.spotcheckit.org/osmw/
```

It should come up with an error as the install has not been done yet, click the link to start the install. Now you should see the title "Open Simulator Manager Web Installer".
Enter the 5 pieces of data. the database will be located on the server itself. 
```
Database Host : 127.0.0.1 
Database User : dyount_opensim
Database Password : PaS$W0Rd
Database Name : dyount_opensim
DNS Name Server : opensim.spotcheckit.org
```

Click the install button and then you should see the message.
```
Creation of effected configuration file with success ...
```

If so,then go to the follow URL to log into OSMW for the first time and login using the inital password from the opensim database user,first name is Super,Last name is Admin and the password is password. Ths can be changed after login by going to right hand side Super Admin drop down and modify your account replace the password , or other credtials if you wish, but at least the password.   
```
https://opensim.spotcheckit.org/osmw/
Firstname: Super
Lastname: Admin
Password: password
```
Click the Home icon and then below click "Administrator Section" as we still need to configure file locations and basic naming conventions of your Opensim GRID. Click on "Management simulators" and in here enter your grid name. 
```
Name: SpottyGrid
Version: Opensim 0.9.2.1
Path: /home/dyount/opensim.spotcheckit.org/opensim/
HG url: opensim.spotcheckit.org:80
Database: dyount_opensim
```
Then click Update. Now the name and version of your current selected grid should be showing your grids name. Next in "Administrator Section" click "Configuring OSMW" . The first option is "/osmw/" and we created a symlink for this default options so no change needed. The second option is the only one needed to be changed for now tol your email. Then next will be in "Administrator Section" again "Editing configuration files" , once you click on this section you may see red notifications saying files do not exist. It will show the files it found and the configurations files can be edited and updated in here. 

Create config files not set yet in Opensim as account user so the manager will find and use and be able to save to them.
```
cd /home/dyount/opensim.spotcheckit.org/opensim/bin
mv startup_commands.txt.example startup_commands.txt
mv shutdown_commands.txt.example shutdown_commands.txt
```

You will still see two other files in notifications but we dont need them for now but will be used for a connected GRID instead of the Standalone GRID that we are setting up now.


This works except for the PHP Send commands as its https not http as my http gets redirected to https and these commands wont fit in a https call. Will work on instructions for settings up XMLRPC access locally as it does work I ran the curl command to shutdown my opensim service on port 9000.  

**Example OpenSim CURL command to call XMLRPC**
```
curl --header "Content-Type: application/xml" \
  --request POST \
  --data "<methodCall>
       <methodName>admin_console_command</methodName>
        <params><param>
         <value>
           <struct>
            <member>
              <name>password</name>
                <value><string>PaS$w0Rd</string></value>
            </member>
            <member>
               <name>command</name>
               <value>quit</value>
            </member>
           </struct>
          </value>
        </param>
        </params>
</methodCall>" http://opensim.spotcheckit.org:9000
```

I have now modified OSMW to work with modern XMLRPC by using cUrl opensim wiki had a method for PHP7 fsocket, but I would rather use CURL its more robust and would like to use oAuthV2 , but that would require add it to the opensim core binary , maybe a module, would like it to be a module which you could still use the older XMLRPC method or enable the Oauth V2 token method , which is more secure. I also had to figure out how to send the console output to a file without overriding the CPU of the system. I could not just do a simple STDOUT/STDERR redirect Mono was not liking it at all. So I had switched to using screen log file method instead. Will have to integrate this new method into the systemd format.  

Currently to start opensim in a detached multi-user screen I do the following. 
```
screen -S Opensim_1 -dm bash -c "mono --desktop -O=all OpenSim.exe" -L;screen -S Opensim_1 -X multiuser on;screen -S Opensim_1 -X acladd dyount
```
Then to set the screen sessions log file for OSMW to read that the XMLRPC command did its job
```
screen -S 85541.Opensim_1 -X logfile /home/dyount/opensim.spotcheckit.org/opensim/bin/OpenSim.Console.log
```
Finally to enable the screen log session to start logging. 
```
screen -S 85541.Opensim_1 -X log
```

This will let the opensim program run while using very little processing power. The standard redirect was slamming one of the cores all of the time. It was in no way practical. This setup is. Next I will need to modify OSMW to have a page for sending and receiving commands and to get its http files and cache them so they can be distrubuted as https in the manager. New browsers really do not like http any more and has to be mitigated. Finishing this setup will let me have the option to run opensim with a gui or from comandline or both with little effort. Just figuring out and updating the code is a lot of effort to finish this tutorial.  

  ... MORE TO COME LATER 
