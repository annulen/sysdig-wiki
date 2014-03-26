## Install the dependencies on your host machine.

### Vagrant
Linux distributions probably have vagrant in their repositories already.  You can also go to the [vagrant](http://www.vagrantup.com/) website for more details.

### Virtualbox
Vagrant is capable of using many types of virtualization "containers". The one here uses VirtualBox - which is available for Linux, OSX, and Windows.  

> Refer to the version of vagrant you installed above and make sure you install the appropriate version of VirtualBox.

## Bring up the VM

Go into the directory with the `VagrantFile` in it, make sure you are on a decent internet connection (1.5 Mb DSL should be fine), have about 2.5GB of space, and run

`$ vagrant up`

It will download a ~250MB ISO of ubuntu (currently) and then create a VM for you with all the necessary tools in order to build and run sysdig in a VM.  This may take on the order of 30 minutes or so the first time (but it only needs to be done once).

> You may get an error stating `Chef never successfully completed!` if your internet connection drops during the process.  Run `vagrant provision` from the command line and you should be good.

## Log in to the VM

Now, from the same directory you ran the `vagrant up` command, run

`$ vagrant ssh`

The sysdig source-code files are mapped to the `/vagrant/` directory inside of the VM.

## Build sysdig

Inside the VM, run the following:

	vagrant@precise64:~$ cd /vagrant
	vagrant@precise64:/vagrant$ mkdir build-vagrant
	vagrant@precise64:/vagrant/build-vagrant$ cmake ..
	-- The C compiler identification is GNU
	...
	-- Build files have been written to: /vagrant/build-vagrant

	vagrant@precise64:/vagrant/build-vagrant$ sudo make install

	Scanning dependencies of target luajit
	...
	-- Installing: /usr/local/share/sysdig/chisels/stderr.lua

And that's it. You should now be able to run

	vagrant@precise64:/vagrant/build-vagrant$ sysdig -l

The `/vagrant` directory is a shared resource with the host machine.  You can edit code on your host machine and then rebuild from inside the VM without having to transfer anything over.


