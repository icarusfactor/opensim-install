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

 This is the default setting so no change is needed , just to make aware of
how it connected to the rest of the system.

```
BASEPATH/opensim/bin/OpenSim.ini
Include-Architecture = "config-include/Standalone.ini"
```

We will be using the mysql database instead of the default sqlite. So these change will need to be made. 
```
BASEPATH/opensim/bin/config-include/Standalone.ini
StorageProvider = "OpenSim.Data.MySQL.dll"
```

Not changed but default ,to show its connection to the next.
```
Include-Common = "config-include/StandaloneCommon.ini"
```

The changes here will reflect the switch from default SQLite to mysql.
```
BASEPATH/opensim/bin/config-include/StandaloneCommon.ini
```

Comment out the below line to disable SQLite as storage. 
```
;Include-Storage = "config-include/storage/SQLiteStandalone.ini";
```

Uncomment the below lines and setup the database for opensim. 
```
 StorageProvider = "OpenSim.Data.MySQL.dll"
 ConnectionString = "Data Source=localhost;Database=dyount_opensim;User ID=dyount_opensim;Password=PaSsW0Rd;Old Guids=true;SslMode=None;"
Comment:  
 ;StorageProvider = "OpenSim.Data.Null.dll:NullRegionData"
Uncomment or add:
 StorageProvider = "OpenSim.Data.MySQL.dll:MySqlRegionData"
Give name to grid. 
 gridname = "spotcheckit.org"
Give nick to grid. 
 gridnick = "spottygrid"
```
  ... MORE TO COME LATER 
