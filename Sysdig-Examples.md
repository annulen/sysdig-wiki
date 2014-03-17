This is an ever growing list of cool things you can do with sysdig commands.  
  
Note: For a reference list of basic sysdig commands, see the [quick reference guide](sysdig Quick Reference Guide#wiki-basic-command-list)  
  
* Print the top files that apache has been reading or writing to
> sysdig -c topfiles "proc.name=httpd"

* Show the directories that the user "root" visits
> sysdig -p"%evt.arg.path" "evt.type=chdir and user.name=root"

* List all the incoming connections that are not served by apache.
> sysdig -p"%proc.name) %fd.name" "evt.type=accept and proc.name!=httpd"

* Show the network data exchanged with the host 192.168.0.1  
> as binary:  
> sysdig -s2000 -X -cecho_fds fd.cip=192.168.0.1  
as ASCII:  
> sysdig -s2000 -e -cecho_fds fd.cip=192.168.0.1

* Observe ssh activity
> sysdig -e -cecho_fds fd.name=/dev/ptmx and proc.name=sshd

* See how many open network connections each process has
> sysdig -cfd_countby proc.name "fd.type=ipv4"
