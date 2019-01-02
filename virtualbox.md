# virtualbox.md

## Author

Christopher Steel 

2018-05-29 - initial draft

## Resources

* [Network_Configuration_in_VirtualBox](https://www.thomas-krenn.com/en/wiki/Network_Configuration_in_VirtualBox#Example_configuration_of_a_static_NAT_network)
* [Virtualbox (raw) download site](https://download.virtualbox.org/virtualbox/)
* [File sharing in Oracle VirtualBox for Ubuntu 16.04 LTS](https://medium.com/@thanuja919/file-sharing-in-oracle-virtualbox-for-ubuntu-16-04-lts-fc39c36b81ef)

### ansible vmware provisioning

* [](https://github.com/MindPointGroup/ansible-vmware-provisioning)

## Requirements

* Python version ≥ 2.6 is required.
* Since VirtualBox 5.1 Python 3 is also supported.
* Qt 5.3.2 or higher (Qt 5.6.2 or higher recommended)
* SDL 1.2.7 or higher (this graphics library is typically called libsdl or similar).

### Notes

* VirtualBox manager GUI requires both Qt and SDL.
* The simplified GUI requires only SDL.
* VBoxHeadless does not require Qt or SDL.

### IMPORTANT ?

* **Backups should be added for all actions to allow for a restore.**
* A Script to remove the backups could be made available as well to clean up if everything is OK.



## Ansible role overview

* Check for availability of newer version
  * Check for any previously installed version
  * Remove any previously installed version
  * Restart system (preferred)
  * Install new version
    * Ensure for requirements
    * Retrieve resources
    * Install, configure and test

## Installation

### Options

#### Package source

* Host package vs. Download from creator

#### Installation type

* Upgrade - upgrade the current version
* "Clean" install (Remove any existing installation, configuration and VM's)

## Removal

* Close VirtualBox
* Remove Virtualbox

```shell
apt-cache search virtualbox
sudo apt-get remove -y virtualbox*
```

* Remove current configuration

```shell
rm -R ~/.config/VirtualBox/
ls -al ~/.config/ | grep VirtualBox
```

* Remove any existing VM's

```shell
rm -R ~/VirtualBox\ VMs/
ls -al ~/ | grep VirtualBox
```

* Backup then remove `'/usr/share/virtualbox/`

**DANGER _ NOT TESTED YET!!!**

```shell
sudo cp -R /usr/share/virtualbox/ /usr/share/virtualbox_original
sudo rm -R /usr/share/virtualbox/
```

## Reboot System

* update / upgrade ?

## Download from creator

### Check for latest stable version

```shell
mkdir -p ~/resources/sw/ubuntu/18.04/virtualbox/latest
cd ~/resources/sw/ubuntu/18.04/virtualbox/latest
wget https://download.virtualbox.org/virtualbox/LATEST-STABLE.TXT
```

Grab the version

```shell
echo "VBOX_LATEST=`cat LATEST-STABLE.TXT`" > get_version.sh
chmod +x get_version.sh
source get_version.sh
echo $VBOX_LATEST
```

Output example:

```shell
5.2.12
```

### Download SHA256SUMS

This will be used to obtain the filenames of available downloads

```
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/SHA256SUMS
```

### Extract resource filenames from SHA256SUMS 

First we extract the values in the second column from SHA256SUMS and place them in resource_filenames_raw

```shell
awk '{print $2}' SHA256SUMS > resource_filenames_raw
cat resource_filenames_raw
```

Next we create the file `resource_filenames` without the "*" preceding each file name:

```shell
cat resource_filenames_raw | cut -c 2-1000 > resource_filenames
cat resource_filenames
```

### Download resources

Using `resource_filenames` as our source we download the files we want for our targets distribution.

### Download Virtualbox Distribution Package

Ansible scripts will require the distribution and architecture

#### Download the `xenial_amd64.deb` file

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/`cat resource_filenames | grep bionic_amd64.deb`
```

### Download the Guest Additions ISO

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/`cat resource_filenames | grep VBoxGuestAdditions_${VBOX_LATEST}.iso`
```

### Download the Manual

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/UserManual.pdf
```

### Download the SDK

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/SDKRef.pdf
```

### Download the Extension Pack

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/Oracle_VM_VirtualBox_Extension_Pack-${VBOX_LATEST}.vbox-extpack
```

## Checksums

Confirm the integrity of our downloaded files

Run your checksums:

```shell
sha256sum -c SHA256SUMS 2>&1 | grep OK
```

Output example:

```shell
Oracle_VM_VirtualBox_Extension_Pack-5.2.12.vbox-extpack: OK
SDKRef.pdf: OK
UserManual.pdf: OK
VBoxGuestAdditions_5.2.12.iso: OK
virtualbox-5.2_5.2.12-122591~Ubuntu~xenial_amd64.deb: OK
```

## Copy Resources to target systems

## Installation

### Install VirtualBox

```shell
sudo dpkg -i `cat resource_filenames | grep bionic_amd64.deb`
apt-get install -f
```

### Install Guest Additions ISO

#### Debian to default location

```shell
sudo cp VBoxGuestAdditions_${VBOX_LATEST}.iso /usr/share/virtualbox/.
sudo ls -al /usr/share/virtualbox/
# Create non versioned name of current iso version.
sudo cp /usr/share/virtualbox/VBoxGuestAdditions_${VBOX_LATEST}.iso /usr/share/virtualbox/VBoxGuestAdditions.iso 
```

### Copy Manual to /etc/skel/ ???

TODO

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/UserManual.pdf
```

### Copy SDK References to /etc/skel/ ???

TODO

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/SDKRef.pdf
```

### Install the Extension Pack

NON Commercial usage only

```shell
sudo VBoxManage extpack install --replace Oracle_VM_VirtualBox_Extension_Pack-${VBOX_LATEST}.vbox-extpack
```

Using a script:

```shell
#!/bin/bash
version=$(vboxmanage -v)
echo $version
var1=$(echo $version | cut -d 'r' -f 1)
echo $var1
var2=$(echo $version | cut -d 'r' -f 2)
echo $var2
file="Oracle_VM_VirtualBox_Extension_Pack-$var1-$var2.vbox-extpack"
echo $file
wget http://download.virtualbox.org/virtualbox/$var1/$file -O /tmp/$file
#sudo VBoxManage extpack uninstall "Oracle VM VirtualBox Extension Pack"
sudo VBoxManage extpack install /tmp/$file --replace
```

## Test

```shell
vboxmanage -v
```

Output example:

```shell
5.2.22r126460
```



## Ansible roles

https://github.com/Oefenweb/ansible-virtualbox

- name: add dependency manager
  apt: name=dkms
  sudo: yes

- name: add virtualbox repo for precise
  apt_repository: repo='deb http://download.virtualbox.org/virtualbox/debian precise contrib'
  sudo: yes

- name: add VirtualBox repo signing key
  apt_key: state=present
  ​         url=http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc
