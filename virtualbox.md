# virtualbox.md

## Author

Christopher Steel 

2018-05-29 - initial draft

## Resources

* [Network_Configuration_in_VirtualBox](https://www.thomas-krenn.com/en/wiki/Network_Configuration_in_VirtualBox#Example_configuration_of_a_static_NAT_network)
* [Virtualbox (raw) download site](https://download.virtualbox.org/virtualbox/)
* [File sharing in Oracle VirtualBox for Ubuntu 16.04 LTS](https://medium.com/@thanuja919/file-sharing-in-oracle-virtualbox-for-ubuntu-16-04-lts-fc39c36b81ef)

### ansible vmware provisioning

* [ansible-vmware-provisioning](https://github.com/MindPointGroup/ansible-vmware-provisioning)

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

* Check for resources mount
  * when virtualbox_resources_mount_check
* Check for availability of newer version
  * Get latest current version of virtualbox latest
  * Check for any previously installed version
    * check installed version if any
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

#### Install html2text (Must be installed)

```shell
cd ~/bin/
git clone git@github.com:aaronsw/html2text.git
ln -s html2text/html2text.py html2txt
```

#### Find package name

```shell
cd ~/resources/sw/ubuntu/16.04/virtualbox/5.2.22
html2txt --ignore-links https://download.virtualbox.org/virtualbox/5.2.22/ > index.txt
cat index.txt | grep xenial_amd64 | cut -c 11-81 > xenial_amd64_package.txt.raw
sed -r 's/\s+//g' xenial_amd64_package.txt.raw > xenial_amd64_package.txt
cat xenial_amd64_package.txt | awk '{print "VIRTUALBOX_PACKAGE_NAME="$1}' > xenial_amd64_package_var.txt
cat xenial_amd64_package_var.txt
```

#### Set var

```shell
source xenial_amd64_package_var.txt
echo $VIRTUALBOX_PACKAGE_NAME
```

#### Download the `xenial_amd64.deb` file

```shell
wget https://download.virtualbox.org/virtualbox/${VBOX_LATEST}/`cat xenial_amd64_package.txt`
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

### Ensure for resource storage directory

### Copy over resources

## Install

### Install VirtualBox

```shell
sudo dpkg -i `cat resource_filenames | grep bionic_amd64.deb`
apt-get install -f
```

### Install Guest Additions ISO

(Probably not required, does package install take care of this?

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

Non commercial / Personal and Educational usage

In order to get the or batch installation license key run the following command and accept the license terms:

```shell
sudo VBoxManage extpack install --accept-license=715c7246dc0f779ceab39446812362b2f9bf64a55ed5d3a905f053cfab36da9e --replace Oracle_VM_VirtualBox_Extension_Pack-5.2.22.vbox-extpack
```

This will output a (the?) batch installation license. Here is example output when running the above on the Extension Pack installation procedure for VirtualBox  version 5.2.22:

```shell
relating to this Agreement.

Do you agree to these license terms and conditions (y/n)? y

License accepted. For batch installaltion add
--accept-license=56be48f923303c8cababb0bb4c478284b688ed23f16d775d729b89a2e8e5f9eb
to the VBoxManage command line.

0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Successfully installed "Oracle VM VirtualBox Extension Pack".
```



```shell
virtualbox_extension_pack_batch_installation_license
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

## User Documentation

### References

* http://www.mydailytutorials.com/how-to-split-strings-and-join-them-in-a%E2%80%8Bnsibl%E2%80%8Be/
* 

### Virtualbox Included Documents

```shell
sudo cp UserManual.pdf /usr/share/doc/virtualbox-5.2/.
sudo chmod 0644 /usr/share/doc/virtualbox-5.2/UserManual.pdf

sudo cp SDKRef.pdf /usr/share/doc/virtualbox-5.2/.
sudo chmod 0644 /usr/share/doc/virtualbox-5.2/SDKRef.pdf 
ls -al /usr/share/doc/virtualbox-5.2/.

```

### Users local menu

#### Ensure for user documentation directory

Candidates:

```shell
~/bin/local-menu
```

```shell
~/.local/acemenu
```

#### Ensure menu requirements are installed

* menu program
* firefox
* 

```shell
mkdir -p ~/opt/acemenu/docs/virtualbox/
```

minimum file list

files/docs/virtualbox/