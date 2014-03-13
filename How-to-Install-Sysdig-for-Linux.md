**NOTE**: Run all commands as root or with sudo  

## Requirements

### Distributions

The following distributions are supported:

* Debian, from 6.0
* Ubuntu, from 10.04
* CentOS, from 6.3
* RHEL, from 6.3
* Fedora, from 13
* Amazon Linux, any version available from the AWS Marketplace 

## Automatic Installation  
To install sysdig automatically in one step, simply run the following command. This is the recommended installation method. For step-by-step manual installation, see the guide below. To install sysdig from the source code, see the instructions [here](How to Install sysdig from the Source Code).

```
curl -s http://download.draios.com/stable/install-sysdig | sudo bash
```

## Manual Installation
 
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