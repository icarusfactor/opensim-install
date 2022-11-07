# opensim install
### version 9.2.1

***

### Step by step setup of a full Opensim system from server to client.

***

###### NOTE: Server Operating system is CentOS7.9 but should work on any Linux version.

***

*Programs Required:*
* Linux
* SSH
* Apache
* Mysql
* PHP 7.4
* Mono
* Opensim
* Screen
* Inotify-tools
* OSMW – OpenSim Manager Web(forked version)
* SystemD support scripts
* Cool VL Viewer
* Wordpress
* w4os – OpenSimulator Web Interface Plugin

***
*OpenSim Section Configuration:*
 * [Const] 
 * [Network]
 * [Startup]
 * [Estates]
 * [Messaging]
 * [RemoteAdmin]
 * [Groups]
 * [DatabaseService]
 * [GridService]
 * [GridInfoService
  
*Web Intro page:*

*OpenSim Setup:*

*Creating Opensim User:*

*Startup Client Viewer and Setup:*

*Terrain changes and water levels:*

*Backup and Restore regions:*

*Addiotnal Database tables added for OSMW and NPCs:*

*Enable extra files to opensim:*

*Start and Stop Opensim with Systemd:*

*Wordpress Database,Theme Setup:*

*Wordpress Opensim User/Avatar Plugin Setup:*

*Wordpress Customizing Theme & Welcome Page Setup:*

*Example CURL Opensim command to call XMLRPC:*

***

## **Programs Required**

***

**Linux :**

 Initally was trying to keep all of the suporting programs Opeating System independant. But due to security limitation and the in depth installtion,will be Linux only,but others could fork or modify options to work again on Microsoft Windows. With this tutorial any version of Linux should be able to install and setup these program, I will be doing this tutorial on CentOS7. For installation of the OS other resources can do this in a more in depth way than I can here.    
 
 http://isoredirect.centos.org/centos/7/isos/x86_64/

**SSH :**

 Setup of SSH keys for login and quick access to Screen to control administration actions, if not done by the forked OSMW web interface and used to access the SystemD management commands and for this install of the Opensim. If you wish to have a key based login setup, follow below instructions. 

 https://www.ssh.com/academy/ssh/keygen

**Apache 2.4 :**

 A standard Apache web server install will work for what we want. Nothing
 special will be required. We will use it for the Opensim users to easily create and
 manage their avatar and profile with a Wordpress plugin and admin control
 with the OSMW interface. 

```
yum update -y
yum install httpd
systemctl start httpd
systemctl enable httpd.service
```

**Mysql 10.3 :**

 The is the stable version of mysql and will work with opensim and OSWM.
 and recent versions of Wordpress No special setups need to be made so 
 a basic install will work. 

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

**PHP version 7.4 :** 

 This is the stable version and will be used for Wordpress and OSWebManager
to help manage and control Opensim. The only non-default module needed is mysql

```
yum install php php-mysql
systemctl restart httpd.service
```

**Mono version. 6.12:**

 Mono is the FOSS .NET Framework-compatible software. This is the system and binaries and libraries that OpenSim uses to run and can be compiled by using its master repo, if you choose to make modifications to the source.

```
rpmkeys --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"

yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
su -c 'curl https://download.mono-project.com/repo/centos7-stable.repo | tee /etc/yum.repos.d/mono-centos7-stable.repo'
yum clean all
yum makecache
yum install -y mono-complete
```

**OpenSim 9.2.1:**

 Now that we have Mono installed we can install the base component, as its written in Mono and will need a lot of configuration to get it setup, but will do this in another section. This binary will be installed as the web user in their DOCROOT. 

http://opensimulator.org/wiki/Download
http://opensimulator.org/dist/opensim-0.9.2.1.tar.gz

Change directory to Opensim users directory, my directory as example. 
```
cd /home/dyount/opensim.spotcheckit.org/
tar -zxvf opensim-0.9.2.1.tar.gz
```

The binaries and startup files are located in the newly created opensim/bin/ directory. We will symlink the opensim binary to a simpler directory. We will change the name to its simpler form without version. 

```
mv opensim-0.9.2.1.tar.gz ./opensim
cd ./opensim/bin/
```


**Screen:**

 Any recent version of screen should do but we will have to share a screen with the user that is running the Opensim binary so Screen will need to be used, tmux is the alternative in newer OS’s but does not have user based screen sharing.  

```
yum update -y
yum install screen -y
screen -v
Screen version 4.01.00devel (GNU) 2-May-06
```


**Inotify Tools:**

I've seen many tutorials where the install method is to chmod 777 the directory so that a web interface can use it. I really did'nt want to use this method, as it would lead to security issues. Enter the Inotfy tools package, using this method will deal with all of our permissions issues in a secure way and this will also be used to bypass PHP's exec() and system() functions. This setup will act as a file based trigger system for file system based binary execution and will watch for files created by Opensim running as root and change them instead to be owned by the users account so the web manager can get access to them. 

```
yum install epel-release
yum install inotify-tools
```

**OSWM - OpenSim Manager Web:**

This is a user-friendly web interface that will access the remote XMLRPC port of Opensim to change configurations on the fly or other basic opeations that will need to be restarted before being active within the Opensim environemnt.  

While managing your world with the Opensim Console is fine, having a browser based manager will make it much easier to keep it managed and configured and you can immedialty see textures and maps not just the data and makes organizing much easier. 

I have upgraded the original OSMW as it was limited to the End Of Life PHP5.x and patched and upgraded it to work with PHP7.x and maybe even 8.x. and is now only Linux, but could be made to work with Windows, but would need to be patched and alternate support files used.  

In the new version I have added a console control page to act as a read and write for the command line terminal interface to Opensim and I have also modified the map page to make it interavctive for adding regions and changing terrains for each region. As the account of user change directory to the Opensim DOCROOT of the account. In here we will use git clone to grab OSMW as user. 
```
 git clone https://github.com/icarusfactor/OpenSim-Manager-Web-V5.git
 ln -s /home/dyount/opensim.spotcheckit.org/OpenSim-Manager-Web-V5 osmw
```

Original Github code URL for historical purposes: 
https://github.com/Nino85Whitman/OpenSim-Manager-Web-V5

**SystemD :**

Now we have all of our support programs ready to run our SystemD script to automate starting and stopping Opensim from commandline next we will write the script to do it. 
Change the script to fit your install, in my case was the directory /home/dyount/opensim.spotcheckit.org/ but you will change tis to yours. 

``
cd /etc/systemd/system/
vim opensim.service
``

```
# Systemd Service Unit for OpenSimulator Inotify 
[Unit]
Description=OpenSimulator with Inotify services
After=syslog.target network.target
 
[Service]
User=root
Type=forking
PIDFILE=/run/opensim/screen.pid
LimitSTACK=1048576
TimeoutStopSec=60
WorkingDirectory=/home/dyount/opensim.spotcheckit.org/opensim/bin
ExecStart=/home/dyount/opensim.spotcheckit.org/opensim/bin/Opensim_start.sh
ExecStop=/home/dyount/opensim.spotcheckit.org/opensim/bin/Opensim_stop.sh
KillMode=none
 
[Install]
WantedBy=multi-user.target
```

Now we need to move some of the scripts from the OSMW install directory to the Opensim binary directory.
```
cd /home/dyount/opensim.spotcheckit.org/osmw
cp Opensim_start.sh Opensim_stop.sh inotfy-start-backend.sh inotify-change-ownership.sh inotify-exec-command.sh ../opensim/bin/
```

For inotify to work we need to setup the file system directory where the created files will be used as a trigger. 
```
cd /home/dyount/opensim.spotcheckit.org/
mkdir exec
```

**Cool VL Viewer :**

Now we will install the Cool VL viewer as the client to access the new Grid. Download the binary to our Linux or Windows PC desktop and run the binary to install it. MacOS version do exist as well.

http://sldev.free.fr/binaries/CoolVLViewer-1.30.0.24-Linux-x86_64-Setup
http://sldev.free.fr/binaries/CoolVLViewer-1.30.0.24-Windows-x86_64-Setup.zip

```
bash ./CoolVLViewer-1.30.0.13-Linux-x86_64-Setup  
cd CoolVLViewer
./cool_vl_viewer
```

**Wordpress :**

As account user we will download,uncompress and this will create a subdreictory that it will install to and setup latest version of Wordpress. It is best to download the latest version for security reason and will try to keep it standard to upgrades wont be a problem.

After downloading and installing we will need to create an .htaccess file to give access to Wordpress in the BASEDIR.   

```
cd BASEDIR/
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
vim .htaccess
```

To give access to the Wordpress install from the default directory that it creates once uncompressed we will create a basic .htaccess file in BASEDIR.
```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /wordpress
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /wordpress/index.php [L]
</IfModule>
```

Now we will change to the directory as user and verify that its installed and is active by using wp commandline client. We will need to create a database before we can do much more, but will stay in this subdirectory for now.
```
cd wordpress
wp core version
6.1
```

You can also test that Wordpress is working and ready to install by going to the domain plus the subdirectory. We will create a base index page later , but need to make sure the applications(opensim,osmw,wordpress and general saved files) are seprate and not all installed into the base directory and this is helpful if any one of them need to be upgraded and could be done individually. 

With any modern browser you can go to the site url. Then it should redirect to message about clicking here to setup the new site. We wont go any further since we dont have database setup yet. 
```
https://opensim.spotcheckit.org/wordpress/
There doesn't seem to be a wp-config.php file. It is needed before the installation can continue.
```

**w4os – OpenSimulator Web Interface Plugin :**

In the current state, without being connected to Wordpress's database we wont be able to install any plugins, unless we do it manually. So lets do so. This is in an unstable status so it is good to keep it updated often. If a good stable point take place , I will fork it and make a static one at least until the core one become stable. 
```
cd wp-content/plugins/
git clone https://github.com/GuduleLapointe/w4os.git
```

Now when we setup the empty database for Wordpress and step through the browser based installation which will be done later in this tutorial the plugin will be ready to activate and configure and use.   

***

## **OpenSim Section Configuration**

***

Now that we have all our programs that are required for a basic setup we will start the actual configuration of these programs. 


**Files changed from default settings:**

The inital configuration file. 
```
BASEPATH/opensim/bin/OpenSim.ini
```

Change the name of the constant to the hostname prompt display.Enable saving the history of the console file and how many lines of it and if it is to be timestampped. I disabled crashes, dont need to enable this until problems start occuring and need to debug the main binary and I have enabled the PID file for the binary. No limit set by default, but always good to set a small and large prime size limit to limit any crashes from excessively large prims or prims you can not find in your map. Also allow prims to be bound by the physics engine or not.  

Changes in the [Const] section. This needs to be a domain name and not IP to support the grid welcome web page load on start up and change the PublicPort to reflect the active port. 
```
BaseHostname = "opensim.spotcheckit.org"
PublicPort = "9000"

```

Changes in the [Network] uncomment the default just to make sure. 
```
http_listener_port = 9000
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

Changes in the [Messaging] section. While this is optional it will enable Offline messaging service and uses mysql for storage, which we will also be setting up.  
```
  OfflineMessageModule = "Offline Message Module V2"
  StorageProvider = OpenSim.Data.MySQL.dll
  MuteListModule = MuteListModule
  ForwardOfflineGroupMessages = true
```

Changes in the [RemoteAdmin] section. This will be needed for our custom web interface to manage the grids configuration files by using XML-RPC.  
```
 enabled = true
 port = 0
 access_password = "pAs$w0Rd"
```

Changes in the [Groups] section. This is part of and used by the messaging service. 
```
Enabled = true
LevelGroupCreate = 0
HomeURI = "http://${Const|BaseHostname}:${Const|PublicPort}"
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

Not changed but default,to show its connection to the next. This is at the end of the file.
```
Include-Common = "config-include/Standalone.ini"
```


The changes here will reflect the switch from default SQLite to mysql.
By selecting Standalone.ini it will also need StandaloneCommon.ini
says do not change any option in Standalone.ini
```
BASEPATH/opensim/bin/config-include/Standalone.ini
```

Enabling the file Standalone.ini also enables the below file. 
```
BASEPATH/opensim/bin/config-include/StandaloneCommon.ini
```

Changes made to [DatabaseService] section. Comment out the below line to disable SQLite so we will be using mysql instead.
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


***

## **Web Intro page**

***

In this section we will enable the display of an actual web page that can be used for showing news,registeration to grid or even maps or webGL pages when the user selects
your grid to login to. As a user follow the below steps to set the Opensim webserver welcome page.
```
cd /home/dyount/opensim.spotcheckit.org/
mkdir welcome 
cd welcome
```

Under the DOCROOT in the weclome directory of Apache place the below basic web page we will use as a starter for the grid. 
```
<!DOCTYPE html>
<html>
<head>
    <title>SPOTTY GRID</title>
    <style>
        .head1 {
            font-size:40px;
            color:#009900;
        }
        footer {
            width: 100%;
            bottom: 0px;
            background-color: #000;
            color: #fff;
            position: absolute;
            padding-top:20px;
            text-align:center;
        }
    </style>
</head>
<body>
    <header>
        <div class="head1">SPOTTY GRID</div>
    </header>
        <section id="Content">
            <h3>Content section</h3>
        <a href="http://opensimulator.org">HOME</a>
        <a href="https://get.webgl.org">NEWS</a>
        </section>
    <footer>Footer Section</footer>
</body>
</html>
```

Now still in section [GridInfoService] comment out the below line to enable this
```
[GridInfoService]
welcome = ${Const|BaseURL}/welcome
```

***

## **OpenSim Setup**

***

Now that we have setup the base Opensim system we will use the screen command to do the inital start and setup of Opensim as currently the database is empty and needs to be setup and will pull the login credtials in the [DatabaseService] section of the configuration file. Later we will install an automatic start and stop program, just do not have a good way to do this from systemD as of yet and the one I use I made part of the OSMW package currently. 

We will change directory and manually start Opensim in a detached screen then reattach to the screen and enter the inital database settings. This will make it easier to switch to Opensim and back to commandline throughout the rest of the tutorial.
```
cd BASEPATH/opensim/bin/
screen -S Opensim_1 -dm bash -c "mono --desktop -O=all OpenSim.exe";
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

After entering the IP address it should finish starting up the service and filling the database that was created for this service. Next we will create the user on the console and setup the client binary to create a login for the grid. One of the users wa screate already by the Estate section. To exit the session to the main console press CTRL+A+D


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

***

## **Cool VR Viewer Setup**

***

Once the viewer starts.
  Click on the menu "Edit" option and then select "prefrences". 
  
  Now make sure the tab "grids list" is highlighted.
  
  In the Login URI enter the IP address with port http://69.167.171.208:9000
  This may need to be done with the domain name instead. Will do tests later for best method to auto fill the new grids information. 
  
  Then click "get paramaters". It may try to change the port to 8002 , change it back to 9000 or it wont work.
  
  Then if everything works you should see the grid name show up in the "Grid Name:" section.
  
  Then click over into the list and select "Add New Grid" it should be in the list at this time and able to pick and log in as a user to.
  
  Fill out the "First name:" and "Last name:" and password of the user. Click login.
  
  When you first log in, errors dialog boxes will pop up about no inventory and region data. Just click through these as it will create this initial data and you wont have to click through them again.
  
  Now you have your avatar enabled and active to walk/fly around the region. 

***

## **Terrain changes and water levels**

***

Back to the server side within the screen command we will download and install custom terrains. We will also change the default water level. 
```
screen -r Opensim_1
```
From the opensim console we will change the default water level wihich is 20m to 24.8m.  
While logged into the opensim server with our user avatar when the command is issued you will immedialy see the level of water change. 
```
set water height 24.8
```

Now we can CTRL+A+D to exit console to terminal will create a directory for our custom terrains and general backups. Then download a few free terrians and load them while we
are in the viewer. In the DOCROOT of your opensim account as user. Not within the opensim directory. 
```
mkdir backups
cd backups
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
screen -r Opensim_1
terrain load /home/dyount/opensim.spotcheckit.org/backups/dw-island1x1-7.r32
terrain load /home/dyount/opensim.spotcheckit.org/backups/dw-flat-starter.r32
terrain load /home/dyount/opensim.spotcheckit.org/backups/dw-island1x1-9.raw
```
After loading each you can move your avatar around to see it was immediately change around the avatar. 

***

## **Backup and Restore regions**

***

Now in the screen Console to save the current opensim setup type:
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

Now that we have our region saved, lets change it by loading another terrain and see if we can actively load the saved OAR file. 
```
terrain load /home/dyount/opensim.spotcheckit.org/terrains/dw-flat-starter.r32
```
Now after you view the changed terrain happen, load up the previous state. 
```
load oar /home/dyount/opensim.spotcheckit.org/backups/region.oar
```

After running the above command the previous state will be activated again. Now you know how to save and restore your world. 

***

## **Additonal Databases and OSMW Setup config file**

***

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

A patch to the newly created config will need to be made to this new file. 
```
cd /home/dyount/opensim.spotcheckit.org/OpenSim-Manager-Web-V5/inc
vim config.php
```

Add the below lines to the config file. The slash will denote MS Windows or Linux OS.
At this time Linux is what is supported, although Windows may works but not be able to
use all of the componets. 
```
/*Save locations base */
$baseSave = "../../";
$baseBackups = "backups";
$baseImages = "images";

$slash = "/";
$base_dir = "/home/dyount/opensim.spotcheckit.org/";
$cache_dir = $base_dir."cache/";
$exec_dir = $base_dir."exec/";
```

Then go to the follow URL to log into OSMW for the first time and login using the inital password from the opensim database user,first name is Super,Last name is Admin and the password is password. Ths can be changed after login by going to right hand side Super Admin drop down and modify your account replace the password , or other credtials if you wish, but at least the password.   
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

Then click Update. Now the name and version of your current selected grid should be showing your grids name. Next in "Administrator Section" click "Configuring OSMW" . The first option is "/osmw/" and we created a symlink for this default options so no change needed. The second option is the only one needed to be changed for now to your email. Clock Save and then next will be in "Administrator Section" again "Editing configuration files" , once you click on this section you may see red notifications saying files do not exist. It will show the files it found and the configurations files can be edited and updated in here. 

***

## **Enable extra files to Opensim**

***

Create config files not set yet in Opensim as account user so the manager will find and use and be able to save to them.
```
cd /home/dyount/opensim.spotcheckit.org/opensim/bin
mv startup_commands.txt.example startup_commands.txt
mv shutdown_commands.txt.example shutdown_commands.txt
```

You will still see one other file in notifications but we dont need it for now, but will be used for a connected GRID instead of the Standalone GRID that we are setting up now.

***

## **Start and Stop Opensim with Systemd**

***

You should be able to from the commandline be able to easily start and stop Opensim from the SystemD service. 
```
service opensim start
or
service opensim stop
```

***

## **Wordpress Database,Config & Theme Setup**

***
The section below is still tentative and will be refined as more revisions are made, but does work. Although, is more stumbling and learning than showing how to set up Opensim.
***

Getting back to the Wordpress install for managing and creating users within Opensim and organizing avatars. We will need to setup the database before doing anything else. This database needs to be seperate from the Opensim database, but also is not a complex setup, Wordpress install will do that automaticaly when doing the setup for the first time. If you have a hosting company and using a control panel you will need to setup the database and user through it, if cPanel use the following command as root.
```
uapi --output=jsonpretty \
  --user=dyount \
  Mysql \
  create_user \
  name='dyount_osimwp' \
  password='12345ismypassword'

uapi  --user=dyount Mysql create_database name=dyount_osimwp

uapi --user=dyount Mysql set_privileges_on_database user=dyount_osimwp database=dyount_osimwp privileges=ALL
```

If not using any control panel then use the below commands as root to do it manually.
```
mysql 
create database dyount_osimwp
CREATE USER 'dyount_osimwp'@'localhost' IDENTIFIED BY 'pa$sW0rd';
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on dyount_osimwp.* TO 'dyount_osimwp'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

That's it for setting up the mysql database for Wordpress to install to. Now we will as user use the wp-cli for install and create the default wp-config.php file.
```
cd BASEDIR/wordpress
wp config create --dbname=dyount_osimwp --dbuser=dyount_osimwp --dbpass=12345notmypassword --locale=en_US
Success: Generated 'wp-config.php' file.
```

Now in the browser you will go to the sites url and select language and then click continue.
```
http://opensim.spotcheckit.org/wordpress/
```

Enter your info to setup the default wordpress theme.
```
Site Title "Spotty Grid"
Username "dyount"
Password "AreAllygoodpa$Ssw0rd!"
Your Email "factorf2@yahoo.com"
```

Then click the "Install Wordpress" button. You should then see Success! You should be able to login into the backend as admin now to check it out, but keep the commandline ready as we will continue with it.  
```
http://opensim.spotcheckit.org/wordpress/wp-admin
```

Still as user from the comandline you can search for themes, while the default theme is okay,we have many generes and styles to pick from and a good one to act as an opensim interface for the welcome,about,help and profile page will be the free theme "justread". first check if its avialable to you. 
```
wp theme search justread

Success: Showing 1 of 1 themes.
+----------+----------+--------+
| name     | slug     | rating |
+----------+----------+--------+
| Justread | justread | 100    |
+----------+----------+--------+
```

Now we can install this theme from commandline. 
```
wp theme install justread

Installing Justread (1.3.0)
Downloading installation package from https://downloads.wordpress.org/theme/justread.1.3.0.zip...
Unpacking the package...
Installing the theme...
Theme installed successfully.
Success: Installed 1 of 1 themes.
```

Now that the theme that we will be working with in Opensim is installed we now need to enable it, to see it. 
```
wp theme activate justread

Success: Switched to 'Justread' theme.
```

***

## **Wordpress User Opensim Avatar Plugin Setup**

***

Now that we have Wordpress setup and ready for the OpenSim plugin to be setup, click on the the "Plugins" option on left hand side of the Wordpress dashboard and you should see the Opensim Wordpress plugin in a disabled stated. Click "activate" on the  w4os plugin section.

After clicking it you should see a notification bar pop up in the top section of Wordpress dashboard that says "ROBUST database is not configured. To finish configuration, go to OpenSimulator settings page". Click the link in notice to finish setting it up.

You will fill out the form data with below info matching your setup.
```
Grid info
Login URI http://69.167.171.208:9000
Grid name Spotty Grid

Robust server database
Hostname opensim.spotcheckit.org
Database name dyount_osimwp
Username dyount_osimwp
Password  12345thisIsnotmypaSsword

Profile page Defaults
Check Show configuration instructions to new users.
Login page Default

Exclude from stats  Models / Accounts without mail address / Hypergrid visitors
```
Then click the "Save Changes" button. After doing this a red notification will pop up that says "Missing tables: Presence. The ROBUST database is connected, but some required tables are missing. OpenSimulator settings page".

To fix this we will need to edit the "Standalone.ini" file and comment out the Null library and replace it with the MySQL one. 
```
vim /home/dyount/opensim.spotcheckit.org/opensim/bin/config-include/Standalone.ini
[PresenceService]
    LocalServiceModule = "OpenSim.Services.PresenceService.dll:PresenceService"
    ;StorageProvider = "OpenSim.Data.Null.dll"
    StorageProvider = "OpenSim.Data.MySQL.dll"
```

Now if you click on the Opensim icon in the left hand menu and then status you should see some basic infomration. Scroll down to bottom section OpenSimulator pages, here it will check for active links to Opensim services and connections to it. What should be checked at this point is.

* Check pages now  - You can click this to get the latest stats. Looks to be connected to Wordpress cron service, so it will update automatically after a certian period has past. 

* Avatar profile - While is checked ,currently it is broken if you check this link. But is set to use default instead of link in settings for the profiles, so it wont error out. 

To get the Wordpress "Avatar profiles" option to work, as it will fail if not set to "true" in Opensim configuration "StandaloneCommon.ini" file, change the following line from "false" to "true".
```
[UserProfilesService]
    ;; To use, set Enabled to true then configure for your site...
    Enabled = true
```

Since we made changes to Opensim service we will need to restart it. 
```
service opensim stop
service opensim start
```


* Splash - While this option was setup earlier in this tutorial with only basic HTML, now that we have our CMS ready, we can now comment out that link and chang eit to the one of the Wordpress pages. 

```
vim /home/dyount/opensim.spotcheckit.org/opensim/bin/config-include/StandaloneCommon.ini
;welcome = ${Const|BaseURL}/welcome
welcome = http://opensim.spotcheckit.org/wordpress/welcome
```

We can now remove the welcome directory we created as a test page for the inital page that pops up when logging into the grid. 
```
rm -rf BASEDIR/welcome
```

* Registration page - While not checked , we will now setup the registration as well. Cut and paste the link from the grey helper box. 

```
;register = ${Const|BaseURL}/register
register = http://opensim.spotcheckit.org/wordpress/wp-login.php?action=register
```

* Password recovery - Also not checked, we will now setup the Lost Password link as well. cut and paste from the grey helper box. 

Within the same file search for below line and paste in the line below it matching your domain.
```
; password help: optional: page providing password assistance for users of your grid
;password = ${Const|BaseURL}/password
password = http://opensim.spotcheckit.org/wordpress/wp-login.php?action=lostpassword
```

To activate in Opensim these links we will need to restart the Opensim service. But before we do, I will note the "About" and "Help" links can be activated by adding any page as well and linked to a Wordpress page once you make them. 
```
service opensim stop
service opensim start
```

If you refresh and check the pages now it should show check marks in front of "Registration page" and "Password revovery". Since we have enabled Offline Messaging Service in Opensim we can set it up to a degree and make it available here. To do so, click Helpers option page. Scroll down to "Offline messages" section. We will cut and paste the "GridInfoService" section and paste in in the setion of the file of the same section.
```
[GridInfoService]
; Added for w4os
message = http://opensim.spotcheckit.org/wordpress/helpers/offline/
```

Open the main ini file and cut and paste section into it.
vim /home/dyount/opensim.spotcheckit.org/opensim/bin/OpenSim.ini
```
 ; OfflineMessageURL = ${Const|PrivURL}:${Const|PrivatePort}
OfflineMessageURL = http://opensim.spotcheckit.org/wordpress/helpers/offline/
```

Since we made changes to Opensim service we will need to restart it before the change will become active. 
```
service opensim stop
service opensim start
```

Offline messages should now show as an "!" mark. To set this up further an extra helper script needs to be added to connect to WP mail. Will leave this as is for now and may set this up at some point int the future as it is a nice option for the web site. But just to make aware of its an option at this time. 
https://github.com/GuduleLapointe/flexible_helper_scripts

***

## **Wordpress Customizing Theme & Welcome Page Setup:**

***

Using a browser, go to the url http://opensim.spotcheckit.org/wordpress/ and it should have at least two options, "Avatar Profile" installed by w4os plugin and "Sample Page" installed by the Justread theme. 

What we want to make is a custom Welcome page and then we will swap the raw HTML one we had with the Wordpress one. We now will replace the default page in Wordpress as it just shows the recent posts in blocks with our own custom page to have the basic site login options for new or returning users to get access to the sites Avatar functions. 
```
 wp post create --post_type=page --post_title="Welcome" 
 
 Success: Created post 116.
```

We have to install the importer plugin so we can then install the premade example site pages. 
```
wp plugin install wordpress-importer --activate

Installing WordPress Importer (0.8)
Downloading installation package from https://downloads.wordpress.org/plugin/wordpress-importer.0.8.zip...
Unpacking the package...
Installing the plugin...
Plugin installed successfully.
Activating 'wordpress-importer'...
Plugin 'wordpress-importer' activated.
Success: Installed 1 of 1 plugins.
```

To limit access for users who have logged for some menu items, we will add the "User Menu" plugin. 
```
wp plugin install user-menus

Installing User Menus – Nav Menu Visibility (1.3.1)
Downloading installation package from https://downloads.wordpress.org/plugin/user-menus.1.3.1.zip...
Unpacking the package...
Installing the plugin...
Plugin installed successfully.
Success: Installed 1 of 1 plugins.

wp plugin activate user-menus

Plugin 'user-menus' activated.
Success: Activated 1 of 1 plugins.
```

 We need to get rid of the "Sample page" so it does not clutter or show up in the menu.
We will need to find the ID of the page and then we can remove the theme Sample page. 
```
 wp post list --post_type=page
+-----+----------------+----------------+---------------------+-------------+
| ID  | post_title     | post_name      | post_date           | post_status |
+-----+----------------+----------------+---------------------+-------------+
| 2   | Sample Page    | sample-page    | 2022-11-03 16:30:53 | publish     |
+-----+----------------+----------------+---------------------+-------------+

wp post delete 2 --force 
Success: Deleted post 2.
```

With the Sample page deleted lets download and import the default template pages Welcome, Help, About and Grid.
```
wget https://raw.githubusercontent.com/icarusfactor/opensim-install/main/wordpress.welcome.xml

wget https://raw.githubusercontent.com/icarusfactor/opensim-install/main/wordpress.grid.xml

wget https://raw.githubusercontent.com/icarusfactor/opensim-install/main/wordpress.help.xml

wget https://raw.githubusercontent.com/icarusfactor/opensim-install/main/wordpress.about.xml

wp import wordpress.welcome.xml --authors=create

wp import wordpress.grid.xml --authors=create

wp import wordpress.help.xml --authors=create

wp import wordpress.about.xml --authors=create
```

After import of the examples Welcome,Grid,Help and About. We need to edit and link domain and make the default page Welcome. 
```
wp post list --post_type='page' --allow-root

+-----+----------------+----------------+---------------------+-------------+
| ID  | post_title     | post_name      | post_date           | post_status |
+-----+----------------+----------------+---------------------+-------------+
| 269 | Welcome        | welcome        | 2022-11-05 11:18:29 | publish     |
| 381 | Help           | help           | 2022-11-05 11:18:29 | publish     |
| 387 | Grid           | grid           | 2022-11-05 11:18:29 | publish     |
| 389 | About          | about          | 2022-11-05 11:18:29 | publish     |
| 5   | Avatar profile | profile        | 2022-11-03 19:13:14 | publish     |
| 3   | Privacy Policy | privacy-policy | 2022-11-03 16:30:53 | draft       |
+-----+----------------+----------------+---------------------+-------------+

wp option update page_on_front 269

Success: Updated 'page_on_front' option.

wp option update show_on_front page

Success: Updated 'show_on_front' option.
```

We dont need the Welcome menu item in the list and need it removed since its now the home link of the domain. 
```
wp menu list

+---------+--------+--------+-----------+-------+
| term_id | name   | slug   | locations | count |
+---------+--------+--------+-----------+-------+
| 7       | Menu 1 | menu-1 | menu-1    | 3     |
+---------+--------+--------+-----------+-------+

wp menu item list 7 

+-------+-----------+---------+--------------------------------------------+----------+
| db_id | type      | title   | link                                       | position |
+-------+-----------+---------+--------------------------------------------+----------+
| 367   | custom    | Home    | https://opensim.spotcheckit.org/wordpress/ | 1        |
| 370   | post_type | Welcome | http://opensim.spotcheckit.org/wordpress/  | 3        |
+-------+-----------+---------+--------------------------------------------+----------+

wp menu item delete 370 

Success: Deleted 1 of 1 menu items.
```

Lets add menu links to the newly imported pages Help,Grid and About items. 
```
wp menu item add-custom menu-1 "Grid" https://opensim.spotcheckit.org/wordpress/grid/

Success: Menu item added.

wp menu item add-custom menu-1 "Help" https://opensim.spotcheckit.org/wordpress/help/

Success: Menu item added.

wp menu item add-custom menu-1 "About" https://opensim.spotcheckit.org/wordpress/about/

Success: Menu item added.
```

Now that we have a base framework made with Wordpress for avatar & user setup with direct interface to Opensim working, our next step is to start building out the virtual regions that we will build our educational infrastructure onto for classrooms that teacher and students will be able to study with easy by using the streaming and virtual media resources that will create a Massivley Online Open Courses also known as MOOC.

 Not done yet, so many more exciting steps to come. 

***

## **Example OpenSim CURL command to call XMLRPC**

***

 Not required but good to test the XMLRPC access locally to make sure its working, I ran the curl command to shutdown my opensim service on port 9000 but any Console command can be replaced with a new one in the value tag for active command. 
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
