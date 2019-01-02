## install and configure LXD (and ZFS!)

* **not a suitable configuration for production**
* Assumes a "clean" system (no LXD and no ZFS).
* Currently install LXD 3.0.2 and ZFS

----

### Install zfsutils

```shell
sudo apt-get install -y zfsutils-linux
```

---

### For Ubuntu 16.04 only

* Enable Ubuntu backports by uncommenting the following in /etc/apt/sources.list:

```shell
deb http://ca.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://ca.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
```

----

### Install LXD (Ubuntu 16.04 only!)

```shell
sudo apt update
sudo apt install -y -t xenial-backports lxd lxd-client
```

Note: -t=target_release

----

### Installing LXD (Ubuntu 18.04 only!)

```shell
sudo apt update
sudo apt install -y lxd lxd-client
```

----

### You should now be a member of the lxd group

* run the `groups` command, you should see `lxd` in the output.
* log out and then log back in to ensure that your membership in the `lxd` group takes.

----

### Confirm your LXD Version

You need LXD 3.0.2 or greater for this demo

```shell
lxd --version
```

Output example:

```shell
3.0.2
```

----

### lxd init

The `lxd init` command will configure LXD for you. Go with the default answers to each questions with the following two exceptions:

* For the name of the new storage pool enter **lxd** rather than **default**
* When asked what IPv6 address to use enter **none** rather than **auto**.

Launch lxd init:

```shell
lxd init
```

----

### Confirm your LXD/ZFS Initialization.

Once `lxd init` has been run the `lxc` command should provide you access to the other system components required to work with LXC/LXD containers. Here we confirm that the `lxc storage` command is able to access your ZFS zpool's information.

```shell
lxc storage info lxd
```

----

###  Command Output Example from `lxc storage info lxd`

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: ?.??GB
  total space: 17.89GB
used by:
  images:
  profiles:
  - default
```

---

## Using LXC/LXD

In addition to container managment the LXC/LXD CLI also allows you to interact storage, networking and devices, so for example, you can work with the underlying ZFS storage we use and do not need to mess with the ZFS CLI at all.

----

### Launching a Test Container

Here we launch a container called c1 from a Canonical (Container) Image Server. *images* is an alias for this particular server and *ubuntu/xenial* is the name of the container image we want.

```shell
lxc launch images:ubuntu/xenial c1
```

----

## Lets see how much space is used by our new container image:

```shell
lxc storage info lxd
```

output example:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: ?.??GB
  total space: 17.89GB
used by:
  images:
  - 34851ebf08f1c375504b9d83cd037f9cb09ce7ca2d9d0c45b28c98c87b6242e7
  profiles:
  - default
```

----

## Creating a second container

* `lxc launch` first checks to see if this image has been updated on the *images* server.
* It has not so `lxc launch` clones a second container from the existing local image.
* It's up in ready to go almost instantly.

```shell
lxc launch images:ubuntu/xenial c2
```

----

## Lets see how much space is used by our second container:

```shell
lxc storage info lxd
```

----

## Additional Space Used By Our Second Container?

output example:

```shell
info:
  description: ""
  driver: zfs
  name: lxd
  space used: ?.??GB
  total space: 17.89GB
used by:
  containers:
  - c1
  - c2
  images:
  - 34851ebf08f1c375504b9d83cd037f9cb09ce7ca2d9d0c45b28c98c87b6242e7
  profiles:
  - default
```

----

## List our running Containers

```shell
lxc list
```

----

## Notes before moving on

* COW Power + Linux Containers = Something Bordering on Magic.
* The `lxd init` and `lxc` CLI  allows us to use LXD with ZFS without the needing to know much about ZFS or the ZFS CLI.
* LXD on ZFS is very fast./*
* LXD on ZFS is very space efficient.
