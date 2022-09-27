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
screen -r Opensim_1

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

bash ./CoolVLViewer-1.30.0.13-Linux-x86_64-Setup  
cd CoolVLViewer
./cool_vl_viewer
Once it starts up, click on the menu "Edit" option and then select "prefrences". 
Make sure the tab "grids list" is highlighted. In the Login URI enter the IP address with port http://69.167.171.208:9000/ Then click "get paramaters". It may try to change the port to 8002 , change it back to 9000 or it wont work. Then if everything works you should see the grid name show up in the "Grid Name:"" section. Then click over into the list and select "Add New Grid" it should be in the list at this time and able to pick and log in a user to. Fill out the First name: and Last name: and password of the user. When you first log in you will get errors about no inventory and region data. Just click through these as
it will create this data and you wont have to click through again. Now you have your avatar enabled and active to walk/fly around the region. 

Next we will download and load a specific terrian for the avatars to move around in and then save the region for backups and then move on to install the admin web based controls  with OSWM. 
  ... MORE TO COME LATER 
