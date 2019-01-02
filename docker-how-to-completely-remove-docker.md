# to-completely-remove-virtualbox.md

Change everything to be for virtualbox here

Step 1

```shell
dpkg -l | grep -i docker
```

To identify what installed package you have:

Step 2

```shell
sudo apt-get purge -y docker-engine docker docker.io docker-ce  
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce  
```



The above commands will not remove images, containers, volumes, or user created configuration files on your host. If you wish to delete all images, containers, and volumes run the following commands:

```shell
sudo rm -rf /var/lib/docker
ls -al /var/lib/ | grep docker
sudo rm /etc/apparmor.d/docker
ls -al /etc/apparmor.d/ | grep docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

## Reboot the system

This should remove the docker network interface.

## Confirm

```sh
sudo ifconfig
```

You have removed Docker from the system completely.

## Ubuntu 16.04 example

```sh
dpkg -l | grep -i docker
sudo apt-get purge -y docker docker-ce
sudo apt-get purge -y docker docker.io
sudo apt-get autoremove -y --purge docker docker-ce
```


