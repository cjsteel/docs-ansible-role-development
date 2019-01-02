# docker-installation-ubuntu16.04.md

## References

* https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

## Installation

### Ubuntu 16.04

add the GPG key for the official Docker repository:

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:

```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

update

```shell
sudo apt-get update
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:

```
sudo apt-cache policy docker-ce
```

You should see output similar to the follow:

Output of apt-cache policy docker-ce

```
docker-ce:
  Installed: (none)
  Candidate: 5:18.09.0~3-0~ubuntu-xenial
  Version table:
     5:18.09.0~3-0~ubuntu-xenial 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     18.06.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
     18.06.0~ce~3-0~ubuntu 500
...     
```

Notice that `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 16.04 (`xenial`).

Finally, install Docker:

```shell
sudo apt-get install -y docker-ce
```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it's running:

```shell
sudo systemctl status docker
```

The output should be similar to the following, showing that the service is active and running:

```shell
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-12-06 12:50:54 EST; 4s ago
     Docs: https://docs.docker.com
 Main PID: 18372 (dockerd)
   CGroup: /system.slice/docker.service
           └─18372 /usr/bin/dockerd -H unix://

Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573874749-05:00" level=warning msg="Your kernel does not support swap memory limit"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573925651-05:00" level=warning msg="Your kernel does not support cgroup rt period"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.573939631-05:00" level=warning msg="Your kernel does not support cgroup rt runtime"
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.574293750-05:00" level=info msg="Loading containers: start."
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.674632926-05:00" level=info msg="Default bridge (docker0) is assigned with an IP address 
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.737679846-05:00" level=info msg="Loading containers: done."
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.826287859-05:00" level=info msg="Docker daemon" commit=4d60db4 graphdriver(s)=overlay2 ve
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.826358016-05:00" level=info msg="Daemon has completed initialization"
Dec 06 12:50:54 ace-ws-101 systemd[1]: Started Docker Application Container Engine.
Dec 06 12:50:54 ace-ws-101 dockerd[18372]: time="2018-12-06T12:50:54.869624332-05:00" level=info msg="API listen on /var/run/docker.sock"
```

Installing Docker now gives you not just the Docker service (daemon) but also the `docker` command line utility, or the Docker client. We'll explore how to use the `docker` command later in this tutorial.



## Step 2 — Executing the Docker Command Without Sudo (Optional)

By default, running the `docker` command requires root privileges — that is, you have to prefix the command with `sudo`. It can also be run by a user in the **docker** group, which is automatically created during the installation of Docker. If you attempt to run the `docker` command without prefixing it with `sudo` or without being in the docker group, you'll get an output like this:

```
Outputdocker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

If you want to avoid typing `sudo` whenever you run the `docker` command, add your username to the `docker` group:

```
sudo usermod -aG docker ${USER}
```

To apply the new group membership, you can log out of the server and back in, or you can type the following:

```
su - ${USER}
```

You will be prompted to enter your user's password to continue.  Afterwards, you can confirm that your user is now added to the `docker` group by typing:

```
id -nG
Outputsammy sudo docker
```

If you need to add a user to the `docker` group that you're not logged in as, declare that username explicitly using:

```
sudo usermod -aG docker username
```

The rest of this article assumes you are running the `docker` command as a user in the docker user group. If you choose not to, please prepend the commands with `sudo`.



## sudo systemctl status docker