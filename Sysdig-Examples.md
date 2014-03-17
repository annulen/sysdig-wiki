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

* See the top processes in terms of actively used network connections
> sysdig -cfdcount_by proc.name "fd.type=ipv4"

* See the top processes in terms of actively used files
> sysdig -cfdcount_by proc.name "fd.type=file"

* See the top local server ports in terms of established connections  
> sysdig -cfdcount_by fd.sport "evt.type=accept"

* See the top client ports in terms of established connections  
> sysdig -cfdcount_by fd.sport "evt.type=accept"
