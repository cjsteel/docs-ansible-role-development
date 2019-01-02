# how-to-completely-remove-virtualbox-ubuntu-16.04.md

* Uninstall any old versions of VirtualBox
* Install new version of VirtualBox (NO )



## Resources

* Https://askubuntu.com/questions/703746/how-to-completely-remove-virtualbox

## Steps

### Disable safe boot / module validation (Ubuntu 16.04)

```shell
sudo mokutil --disable-validation
```

### List currently installed versions of VirtualBox

```shell
sudo dpkg -l | grep virtualbox
```

Output example:

```shell
ii  virtualbox-5.2                                5.2.22-126460~Ubuntu~xenial                  amd64        Oracle VM VirtualBox
```

### Remove and purge any virtualbox installations

Example:

```shell
sudo dpkg --purge virtualbox-5.2
```

### Install a specific version known to work

```shell
wget https://download.virtualbox.org/virtualbox/5.2.18/https://download.virtualbox.org/virtualbox/5.2.18/virtualbox-5.2_5.2.18-124319~Ubuntu~xenial_amd64.deb
```

```shell
curl -sL -o/var/cache/apt/archives/puppetlabs-release-precise.deb https://apt.puppetlabs.com/puppetlabs-release-precise.deb && sudo dpkg -i /var/cache/apt/archives/puppetlabs-release-precise.deb
```

```shell
wget https://download.virtualbox.org/virtualbox/5.2.18/https://download.virtualbox.org/virtualbox/5.2.18/virtualbox-5.2_5.2.18-124319~Ubuntu~xenial_amd64.deb
```

### Run modprobe 

```shell
sudo modprobe vboxdrv
sudo modprobe vboxnetflt
```





## Uninstall virtualbox and configuration

Your problem : Virtual Box keeps its folder and settings in your home folder. Delete everything inside the folder.

Uninstall VirtualBox first. 

```shell
sudo apt-get remove virtualbox* --purge
```

Run these commands to **delete all virtual machines and settings and Virtual Hard Drives**:

```shell
sudo rm ~/"VirtualBox VMs" -Rf
sudo rm ~/.config/VirtualBox/ -Rf
```

Check for running processes:

```shell
sudo ps aux | grep -i "vbox"
```

## Uninstall dpkg installed versions

List all related installations:

```shell
sudo dpkg -l | grep virtualbox
```

Uninstall them

```shell
sudo dpkg --purge virtualbox-qt
sudo dpkg --purge virtualbox-guest-utils
sudo dpkg --purge virtualbox-5.2
sudo dpkg --purge unity-scope-virtualbox

```

Restart the system:

```shell
sudo shutdown -r now
```

## Install from scratch

Uninstalling virtualbox removed the group **vboxusers** but did not remove the group membership for my user:

```shell
groups
```

Output:

```shell
csteel groups: cannot find name for group ID 132
132 lxd aceusers aceadmins nagiosadmins
```

Attempted install without removing my users membership:

```shell
sudo apt install virtualbox
```

Dialog:

```shell
[sudo] password for csteel: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  dkms libgsoap8 libvncserver1 virtualbox-dkms virtualbox-qt
Suggested packages:
  vde2 virtualbox-guest-additions-iso
The following NEW packages will be installed:
  dkms libgsoap8 libvncserver1 virtualbox virtualbox-dkms virtualbox-qt
0 upgraded, 6 newly installed, 0 to remove and 7 not upgraded.
Need to get 0 B/24.5 MB of archives.
After this operation, 107 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

Output:

```shell
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  dkms libgsoap8 libvncserver1 virtualbox-dkms virtualbox-qt
Suggested packages:
  vde2 virtualbox-guest-additions-iso
The following NEW packages will be installed:
  dkms libgsoap8 libvncserver1 virtualbox virtualbox-dkms virtualbox-qt
0 upgraded, 6 newly installed, 0 to remove and 7 not upgraded.
Need to get 0 B/24.5 MB of archives.
After this operation, 107 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Selecting previously unselected package dkms.
(Reading database ... 296650 files and directories currently installed.)
Preparing to unpack .../dkms_2.2.0.3-2ubuntu11.5_all.deb ...
Unpacking dkms (2.2.0.3-2ubuntu11.5) ...
Selecting previously unselected package libgsoap8:amd64.
Preparing to unpack .../libgsoap8_2.8.28-1_amd64.deb ...
Unpacking libgsoap8:amd64 (2.8.28-1) ...
Selecting previously unselected package libvncserver1:amd64.
Preparing to unpack .../libvncserver1_0.9.10+dfsg-3ubuntu0.16.04.2_amd64.deb ...
Unpacking libvncserver1:amd64 (0.9.10+dfsg-3ubuntu0.16.04.2) ...
Selecting previously unselected package virtualbox-dkms.
Preparing to unpack .../virtualbox-dkms_5.1.38-dfsg-0ubuntu1.16.04.1_all.deb ...
Unpacking virtualbox-dkms (5.1.38-dfsg-0ubuntu1.16.04.1) ...
Selecting previously unselected package virtualbox.
Preparing to unpack .../virtualbox_5.1.38-dfsg-0ubuntu1.16.04.1_amd64.deb ...
Unpacking virtualbox (5.1.38-dfsg-0ubuntu1.16.04.1) ...
Selecting previously unselected package virtualbox-qt.
Preparing to unpack .../virtualbox-qt_5.1.38-dfsg-0ubuntu1.16.04.1_amd64.deb ...
Unpacking virtualbox-qt (5.1.38-dfsg-0ubuntu1.16.04.1) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for systemd (229-4ubuntu21.10) ...
Processing triggers for ureadahead (0.100.0-19) ...
ureadahead will be reprofiled on next reboot
Processing triggers for hicolor-icon-theme (0.15-0ubuntu1.1) ...
Processing triggers for desktop-file-utils (0.22-1ubuntu5.2) ...
Processing triggers for bamfdaemon (0.5.3~bzr0+16.04.20180209-0ubuntu1) ...
Rebuilding /usr/share/applications/bamf-2.index...
Processing triggers for gnome-menus (3.13.3-6ubuntu3.1) ...
Processing triggers for mime-support (3.59ubuntu1) ...
Processing triggers for shared-mime-info (1.5-2ubuntu0.2) ...
Setting up dkms (2.2.0.3-2ubuntu11.5) ...
Setting up libgsoap8:amd64 (2.8.28-1) ...
Setting up libvncserver1:amd64 (0.9.10+dfsg-3ubuntu0.16.04.2) ...
Setting up virtualbox-dkms (5.1.38-dfsg-0ubuntu1.16.04.1) ...
Loading new virtualbox-5.1.38 DKMS files...
First Installation: checking all kernels...
Building only for 4.15.0-39-generic
Building initial module for 4.15.0-39-generic
Done.
Progress: [ 80%] [###############################################...........] 
vboxdrv:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-39-generic/updates/dkms/

vboxnetadp.ko:%] [###############################################...........] 
Running module version sanity check.
 - Original module
   - No original module exists within this kernel################...........] 
 - Installation
   - Installing to /lib/modules/4.15.0-39-generic/updates/dkms/

vboxnetflt.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-39-generic/updates/dkms/

vboxpci.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-39-generic/updates/dkms/

depmod....

DKMS: install completed.
Setting up virtualbox (5.1.38-dfsg-0ubuntu1.16.04.1) ...
vboxweb.service is a disabled or a static unit, not starting it.
Setting up virtualbox-qt (5.1.38-dfsg-0ubuntu1.16.04.1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...#######################........] 
Processing triggers for shim-signed (1.33.1~16.04.1+13-0ubuntu2) ...
Secure Boot not enabled on this system.
Processing triggers for systemd (229-4ubuntu21.10) ...
Processing triggers for ureadahead (0.100.0-19) ...
```

Reboot again:

```shell
sudo shutdown -r now
```



## Reinstall

```shell
sudo apt-get install virtualbox-5.2
```

Reboot system

```shell
sudo shutdown -r now
```



Confirm group(s) exist

```shell
cat /etc/group | grep vbox
```

Output example:

```shell
vboxusers:x:132:csteel
vboxsf:x:133:
```

Confirm membership

```shell
groups
```

Output example:

```shell
csteel vboxusers lxd aceusers aceadmins nagiosadmins
```

Confirm version

```shell
VBoxManage --version
```

