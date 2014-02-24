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