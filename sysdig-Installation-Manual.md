## Requirements

### Distributions

The following distributions are supported:

* Debian, from 6.0
* Ubuntu, from 10.04
* CentOS, from 6.3
* RHEL, from 6.3
* Fedora, from 13
* Amazon Linux, any version available from the AWS Marketplace

## Installation

**NOTE**: Run the commands as root or with sudo

### Automatic

```
curl -s http://download.draios.com/stable/install-sysdig | sudo bash
```

### Manual
 
**Debian, Ubuntu**

1) Trust the Draios GPG key, configure the apt repository, and update the package list
```
curl -s http://download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -  
curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list  
apt-get update
```
2) Install kernel headers

**Warning**: The following command might not work with any kernel. Make sure to customize the name of the package properly
``` 
apt-get -y install linux-headers-$(uname -r)
``` 

3) Install sysdig
``` 
apt-get -y install sysdig
``` 

**CentOS, RHEL, Fedora, Amazon Linux**

1) Trust the Draios GPG key, configure the yum repository
```
rpm --import http://download.draios.com/DRAIOS-GPG-KEY.public  
curl -s -o /etc/yum.repos.d/draios.repo http://download.draios.com/stable/rpm/draios.repo
```

2) Install the EPEL repository

Note: The following command is required only if DKMS is not available in the distribution. You can verify if DKMS is available with `yum list dkms`

```
rpm -i http://mirror.us.leaseweb.net/epel/6/i386/epel-release-6-8.noarch.rpm
```

3) Install kernel headers

**Warning**: The following command might not work with any kernel. Make sure to customize the name of the package properly
```
yum -y install kernel-devel-$(uname -r)
```

4) Install sysdig
``` 
yum -y install sysdig
``` 

##Update

Updates are installed as part of the normal system updates available with _apt-get_ and _yum_. If you want to force an update here are the instructions for the various distributions.

**Debian, Ubuntu**
```
apt-get update  
apt-get -y install sysdig
```

**CentOS, RHEL, Fedora, Amazon Linux**
```
yum -y install sysdig
```

##Uninstallation

Here are the instructions for the various distributions.

**Debian, Ubuntu**
```
apt-get remove sysdig
```

**CentOS, RHEL, Fedora, Amazon Linux**
```
yum erase sysdig
```