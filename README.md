# ark-server-tools-tutorial
ARK Server-Tools for Linux - Installation of multiple ark server instances in a crossark travel cluster

## System Setup ##

### Operating System ###
Debian 8 with ssh access and sudo.

### Instances, Maps, Ports set up in this tutorial ###
### Please change the values to your need ###
ARK server user name: ark
ARK cluster name: pveark
Map: TheIsland - Instance: theisland.cfg - Ports: 7777,7778,27015,32330
Map: TheCenter - Instance: thecenter.cfg - Ports: 7779,7780,27016,32331
Map: Ragnarok - Instance: ragnarok.cfg - Ports: 7781,7782,27015,32332
Map: ScorchedEarth - Instance: scorchedearth.cfg - Ports: 7783,7784,27015,32333
Map: Aberration - Instance: aberration.cfg - Ports: 7785,7786,27015,32334

## Install required packages ##
Switch to the root user, install required packes, add the ark server user and give him sudo rights.

```bash
su root
apt-get install perl-modules curl lsof libc6-i386 lib32gcc1 bzip2 nano htop
adduser ark
adduser ark sudo
```

```
su root
apt-get install perl-modules curl lsof libc6-i386 lib32gcc1 bzip2 nano htop
adduser ark
adduser ark sudo
```

## Configure and prepare Linux system ##
Edit the `sysctl.conf` file

```bash
vim /etc/sysctl.conf
```

```
vim /etc/sysctl.conf
```

Add the following line at the very end - if not already present (press `i` for edit mode)
```
fs.file-max=100000
```
Save and exit vim (press `ESC` -> `:wq`)

Edit the limits.conf file
´´´
vim /etc/security/limits.conf
´´´
Add following line above "# End of file" - if not already present (press `i` for edit mode)
´´´
* soft nofile 100000
* hard nofile 100000
´´´
Save and exit vim (press `ESC` -> `:wq`)

Edit the common-session file
´´´
vim /etc/pam.d/common-session
´´´
Add following line above "# end of pam-auth-update config" - if not already present (press `i` for edit mode)
´´´
session required pam_limits.so
´´´
Save and exit vim (press `ESC` -> `:wq`)

## Open firewall ports ##

### Short version (recommended) ###
´´´
iptables -A INPUT -p tcp -m multiport --dports 7777:7786,27015:27019,32330:32335 -j ACCEPT
iptables -A INPUT -p udp -m multiport --dports 7777:7786,27015:27019 -j ACCEPT
´´´
### Long version ###
´´´
iptables -A INPUT -p tcp --dport 7777:7786 -j ACCEPT
iptables -A INPUT -p udp --dport 7777:7786 -j ACCEPT
iptables -A INPUT -p tcp --dport 27015:27019 -j ACCEPT
iptables -A INPUT -p udp --dport 27015:27019 -j ACCEPT
iptables -A INPUT -p tcp --dport 32330:32335 -j ACCEPT
´´´
## Install the ARK Server-Tools and steamcmd (required for ark servers) ##

Download and install ARK Server-Tools
´´´
curl -sL http://git.io/vtf5N | bash -s ark --me
´´´
Switch to the ARK server user
´´´
su ark
´´´
Go to home directory
´´´
cd ~
´´´
Create the steamcmd folder
´´´
mkdir steamcmd
´´´
Switch to the steamcmd folder
´´´
cd steamcmd
´´´
Download and extract steamcmd
´´´
curl -sqL "https://steamcdn-a.akamaihd.net/clien..." | tar zxvf -
´´´
While still in the steamcmd directory, install the arkmanager
´´´
arkmanager install
´´´
Install steamcmd
´´´
cd /home/ark/ARK/
./SteamCMDInstall.sh
´´´

## Configure the arkmanager and ark server instances ##
Switch back to root user
´´´
exit
´´´
Configure the arkmanager
´´´
vim /etc/arkmanager/arkmanager.cfg
´´´
Add flags, options and more (press `i` for edit mode)
´´´
arkflag_log=true
arkflag_NoBattleEye=true
´´´
Save and exit vim (press `ESC` -> `:wq`)

### Configure the default instance ###
Switch to the instances folder
´´´
cd /etc/arkmanager/instances/
´´´
Copy main.cfg (with default settings) to your new instance
´´´
cp main.cfg NEW-INSTANCE.cfg
´´´
Edit your new config
´´´
vim NEW-INSTANCE.cfg
´´´
Add flags, options and more (press `i` for edit mode)
´´´
arkflag_log=true
arkflag_NoBattleEye=true
´´´
Save and exit vim (press `ESC` -> `:wq`)

## Install mods ##
Switch to ark server user
´´´
su ark
´´´
Install the mods
´´´
arkmanager installmods
´´´
Start the ark sever
´´´
arkmanager start
´´´

## Configure autorestart on servercrash ##
Create the file ark-watchdog
´´´
sudo vim ~/ARK/ShooterGame/Binaries/ark-watchdog
´´´
Enter following script (press `i` for edit mode)
´´´
#!/bin/bash
while true
do
if [ ! `pgrep ShooterGameServer` ] ; then
/usr/bin/ark-restart.sh
fi
sleep 30
done
´´´
Save and exit vim (press `ESC` -> `:wq`)

Create the file ark-restart.sh
´´´
sudo vim ~/ARK/ShooterGame/Binaries/ark-restart.sh
´´´
Enter following script (press `i` for edit mode)
´´´
cd /usr/local/bin
./arkmanager restart
´´´
Save and exit vim (press `ESC` -> `:wq`)

Create a symlink to ark-restart.sh
´´´
sudo ln -s /home/ark/ARK/ShooterGame/Binaries/ark-restart.sh /usr/bin/
´´´

## Create a cronjob to check for updates every hour ##
Switch to root user
´´´	
su
´´´
Install the cronjob
´´´
arkmanager install-cronjob --hourly update @all --saveworld --warn --update-mods
´´´

### Check if cronjob added successfully
Switch back to ark server user
´´´
exit
´´´
Show all cronjobs for ark and check if ark updat cronjob added successfully
´´´
crontab -e
´´´
This command (crontab -e) should display
´´´
0 * * * * /usr/local/bin/arkmanager --cronjob update @all  --saveworld --warn --update-mods --args  -- >/dev/null 2>&1
´´´

### FINISHED ~ HAVE FUN ###

# Important arkmanager commands #
Commands for all instances
´´´
arkmanager start @all // Start all instances
arkmanager stop @all // Stop all instances
arkmanager restart @all // ReStart all instances
arkmanager update @all // Check all instances for updates and install updates if available
arkmanager status @all // Check the online status of all instances
´´´

Commands for a single instance (available instances: @theisland, @thecenter, @ragnarok, @scorchedearth, @aberration)
´´´
arkmanager start @theisland // Start the specified instance
arkmanager stop @theisland // Stop the specified instance
arkmanager restart @theisland // Restart the specified instance
arkmanager update @theisland // Check the specified instance for updates and install updates if available
arkmanager status @theisland // Check the online status of the specified instance
´´´
