###Requirements

##Distributions##

The following distributions are supported:

* Debian, from 6.0
* Ubuntu, from 10.04
* CentOS, from 6.3
* RHEL, from 6.3
* Fedora, from 13
* Amazon Linux, any version available from the AWS Marketplace

###Installation

##NOTE##: Run the commands as root or with sudo

##Debian, Ubuntu##

1. Trust the Draios GPG key, configure the apt repository, and update the package list
``` curl -s http://download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -  
curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list  
apt-get update
```

2. Install kernel development files

Warning: The following command might not work with any kernel. Make sure to customize the name of the package properly
apt-get -y install linux-headers-$(uname -r)

3. Install, configure, and restart the Draios agent

apt-get -y install draios-agent

echo customerid = 8c3a3eef-5f95-4937-bc72-f2bb9a48556a >> /opt/draios/bin/dragent.properties

service dragent restart

##CentOS, RHEL, Fedora, Amazon Linux

Trust the Draios GPG key, configure the yum repository
rpm --import http://download.draios.com/DRAIOS-GPG-KEY.public

curl -s -o /etc/yum.repos.d/draios.repo http://download.draios.com/stable/rpm/draios.repo

Install the EPEL repository
Note: The following command is required only if DKMS is not available in the distribution. You can verify if DKMS is available with yum list dkms
rpm -i http://mirror.us.leaseweb.net/epel/6/i386/epel-release-6-8.noarch.rpm

Install kernel development files
Warning: The following command might not work with any kernel. Make sure to customize the name of the package properly
yum -y install kernel-devel-$(uname -r)

Install, configure, and start the Draios agent
yum -y install draios-agent

echo customerid = 8c3a3eef-5f95-4937-bc72-f2bb9a48556a >> /opt/draios/bin/dragent.properties

service dragent start

###Update

Updates are installed as part of the normal system updates available with apt-get and yum. If you want to force an update here are the instructions for the various distributions.

Debian, Ubuntu

apt-get update

apt-get -y install draios-agent

CentOS, RHEL, Fedora, Amazon Linux

yum -y install draios-agent

###Uninstallation

Here are the instructions for the various distributions.

Debian, Ubuntu

apt-get remove draios-agent sysdig

CentOS, RHEL, Fedora, Amazon Linux

yum erase draios-agent sysdig