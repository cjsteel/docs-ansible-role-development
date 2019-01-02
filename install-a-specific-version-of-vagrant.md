# install-lastest-vagrant-from-repository.md



## Requires

* https://github.com/aaronsw/html2text

## Resources

* https://releases.hashicorp.com/vagrant/
* [](https://vagrant-deb.linestarve.com/)
* [](https://github.com/wolfgang42/vagrant-deb)

## Installation Type

###Clean Install

For a standard Vagrant upgrade skip to the section titled **Install Vagrant** below.

If you really want to do a **Clean Install** keep in mind that **this will delete all local vagrant boxes!!!!**

#### Get list of all installed plugins

This could  be used to reinstall plugins at end of process when doing a clean install.

Example:

```shell
mkdir /tmp/vagrant_state
vagrant plugin list > /tmp/vagrant_state/vagrant_installed_plugins
cat /tmp/vagrant_state/vagrant_installed_plugins
```

output example:

```shell
vagrant-auto_network (1.0.2)
vagrant-hostsupdater (1.0.2)
vagrant-share (1.1.9, system)
vagrant-vbguest (0.15.0)
vagrant-winnfsd (1.3.1)
```

#### Uninstall vagrant

* Shutdown all vagrant VM's.

Get the current Vagrant version and do a `set_fact`or similar that Vagrant was previously installed in any Ansible role interpretation of this document.

```powershell
set_fact vagrant_previous_installation: true
```

Unitall any version of Vagrant currently installed on the system.

```shell
sudo apt-get remove -y vagrant*
rm -R ~/.vagrant.d/
```

For the clean install continue with install Vagrant below

## Install Vagrant

### Determine latest version of Vagrant available

#### install html2text

```shell
mkdir ~/bin
cd ~/bin/
git clone https://github.com/cjsteel/html2text.git
ln -s html2text/html2text.py html2txt
```

Log out and back in or source .profile and test

```shell
source ~/.profile
which html2txt
```

#### Determine latest version 

##### For BIONIC + host require edit of html2text.py to enable python3 usage or installation of python2.x

```shell
sudo apt install python-minimal
```

##### Get Vagrant resources

```shell
mkdir -p ~/sw/vagrant/
cd ~/sw/vagrant/
html2txt --ignore-links https://releases.hashicorp.com/vagrant/ > versions_raw.txt
cat versions_raw.txt | grep vagrant | cut -c 13-18 > versions.txt
cat versions.txt
```

Parse largest value and create environment var:

```shell
sort -r versions.txt | head -n1 | awk '{print "VAGRANT_LATEST_VER="$1}' > latest_vagrant_vesion.txt
# confirm
cat latest_vagrant_vesion.txt
```

Set var

```shell
source latest_vagrant_vesion.txt
echo $VAGRANT_LATEST_VER
```

### Download latest version

Create directory

```shell
mkdir -p ~/sw/vagrant/${VAGRANT_LATEST_VER}
cd ~/sw/vagrant/${VAGRANT_LATEST_VER}
```

SHA256SUMS

```shell
wget https://releases.hashicorp.com/vagrant/${VAGRANT_LATEST_VER}/vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS
```

SHA256SUMS.sig

```shell
wget https://releases.hashicorp.com/vagrant/${VAGRANT_LATEST_VER}/vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS.sig
```

vagrant_${VAGRANT_LATEST_VER}_x86_64.deb

```shell
wget https://releases.hashicorp.com/vagrant/${VAGRANT_LATEST_VER}/vagrant_${VAGRANT_LATEST_VER}_x86_64.deb
```

## Install latest

### Confirm signature (TEST ON VM?)

Get key id

```shell
gpg vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS.sig
```

Output

```shell
gpg: assuming signed data in 'vagrant_2.1.1_SHA256SUMS'
gpg: Signature made Mon 07 May 2018 07:23:19 PM EDT using RSA key ID 348FFC4C
gpg: Can't check signature: No public key
```

Import public key

```shell
gpg --keyserver pgp.mit.edu --recv-keys 348FFC4C
```

output example:

```shell
gpg: requesting key 348FFC4C from hkp server pgp.mit.edu
gpg: key 348FFC4C: public key "HashiCorp Security <security@hashicorp.com>" imported
gpg: public key of ultimately trusted key 9B8545CB not found
gpg: public key of ultimately trusted key ACCA5FE7 not found
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
```

### verify

```shell
gpg --verify vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS.sig vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS
```

output:

```shell
SUMS.sig vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS
gpg: Signature made Mon 07 May 2018 07:23:19 PM EDT using RSA key ID 348FFC4C
gpg: Good signature from "HashiCorp Security <security@hashicorp.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 91A6 E7F8 5D05 C656 30BE  F189 5185 2D87 348F FC4C
```

### Confirm checksums

This confirms the checksum for any downloaded items other than the `vagrant_2.1.1_SHA256SUMS` and `vagrant_2.1.1_SHA256SUMS.sig` files.

```shell
sha256sum -c vagrant_${VAGRANT_LATEST_VER}_SHA256SUMS 2>&1 | grep OK
```

Output example:

```shell
vagrant_2.1.1_x86_64.deb: OK
```

### Install vagrant

```shell
sudo dpkg -i vagrant_${VAGRANT_LATEST_VER}_x86_64.deb
```

Output eample:

```shell
[sudo] password for USER_NAME: 
Selecting previously unselected package vagrant.
(Reading database ... 329162 files and directories currently installed.)
Preparing to unpack vagrant_2.1.1_x86_64.deb ...
Unpacking vagrant (1:2.1.1) ...
Setting up vagrant (1:2.1.1) ...
```

Confirm version

```shell
vagrant --version
```

Outut example:

```shell
Vagrant 2.1.1
```

## Install plugins

### vagrant-vbguest

#### References

* https://github.com/dotless-de/vagrant-vbguest

#### Install

```shell
vagrant plugin install vagrant-vbguest
```

Output example:

```shell
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching: micromachine-2.0.0.gem (100%)
Fetching: vagrant-vbguest-0.15.2.gem (100%)
Installed the plugin 'vagrant-vbguest (0.15.2)'!
```

#### Setup

```shell
vagrant up
vagrant vbguest --do install
vagrant reload
vagrant vbguest --status
```

#### Troubleshooting

Check logs on VM

```shell
cat /var/log/vboxadd-setup.log
```

Set path to iso

ensure for iso with generic name:

```shell
sudo mkdir ~/sw/virtualbox/5.2.12/

VBoxGuestAdditions_5.2.12.iso
VBoxGuestAdditions_5.2.12.iso
sudo chown 0755 /usr/share/virtualbox
cp ~/.vagrant.d/tmp/VBoxGuestAdditions_5.2.12.iso /usr/share/virtualbox/.
```

manually configure path:

```shell
# -*- mode: ruby -*-
# vim: ft=ruby


# ---- Configuration variables ----

GUI               = false # Enable/Disable GUI
RAM               = 128   # Default memory size in MB

# Network configuration
DOMAIN            = ".nat.example.com"
NETWORK           = "192.168.50."
NETMASK           = "255.255.255.0"

# Default Virtualbox .box
# See: https://wiki.debian.org/Teams/Cloud/VagrantBaseBoxes
BOX               = 'ubuntu/xenial64'


HOSTS = {
   "web" => [NETWORK+"10", RAM, GUI, BOX],
   "db" => [NETWORK+"11", RAM, GUI, BOX],
}

ANSIBLE_INVENTORY_DIR = 'tests'
ANSIBLE_INVENTORY_FILENAME = 'inventory'


# ---- Vagrant configuration ----

Vagrant.configure(2) do |config|
  HOSTS.each do | (name, cfg) |
    ipaddr, ram, gui, box = cfg

    config.vbguest.iso_path = "#{ENV['HOME']}/sw/virtualbox/5.2.12/VBoxGuestAdditions_5.2.12.iso"

    config.vm.define name do |machine|
      machine.vm.box   = box
      machine.vm.guest = :ubuntu
      machine.vm.synced_folder ".vagrant/synced", "/home/vagrant", disabled: true

      machine.vm.provider "virtualbox" do |vbox|
        vbox.gui    = gui
        vbox.memory = ram
        vbox.name = name
      end

      machine.vm.hostname = name + DOMAIN
      machine.vm.network 'private_network', ip: ipaddr, netmask: NETMASK
    end
  end # HOSTS-each

  config.vm.provision "vai" do |ansible|
    ansible.inventory_dir=ANSIBLE_INVENTORY_DIR
    ansible.inventory_filename=ANSIBLE_INVENTORY_FILENAME
    # optional: add a group listing all vagrant machines
    ansible.groups = {
      'firstGroup' => [ "web" ],
      'secondGroup' => [ "db" ],
    #  '_provided_by_vagrant_'=> HOSTS.keys,
    }
  end

end
```

Examples:

```shell
Vagrant::Config.run do |config|
  # we will try to autodetect this path. 
  # However, if we cannot or you have a special one you may pass it like:
  # config.vbguest.iso_path = "#{ENV['HOME']}/Downloads/VBoxGuestAdditions.iso"
  # or an URL:
  # config.vbguest.iso_path = "http://company.server/VirtualBox/%{version}/VBoxGuestAdditions.iso"
  # or relative to the Vagrantfile:
  # config.vbguest.iso_path = "../relative/path/to/VBoxGuestAdditions.iso"
  
  # set auto_update to false, if you do NOT want to check the correct 
  # additions version when booting this machine
  config.vbguest.auto_update = false
  
  # do NOT download the iso file from a webserver
  config.vbguest.no_remote = true
end
```



#### testing

```shell
vagrant vbguest
```



### vai

#### References

* https://github.com/MatthewMi11er/vai

#### Install

```shell
vagrant plugin install vai
```

Output:

```shell
Installing the 'vai' plugin. This can take a few minutes...
Fetching: vai-0.9.3.gem (100%)
Installed the plugin 'vai (0.9.3)'!
```



## Update plugins

If a complete unistall of vagrant was not performed then your ansible role should look for `set_fact vagrant_previous_installation: true` to determine if this is an upgrade, if so do the following:

On systems that have Vagrant plugins from previous versions try the following to reinstall them and/or update them to the new Vagrant version:

```shell
vagrant plugin expunge --reinstall
```

