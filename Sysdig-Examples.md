This is an ever growing list of cool things you can do with sysdig commands.  
Is there anything that you would find useful and that is not in this list? Send us an email at info@draios.com!
  
_Note_: the command lines on this page return live data. However, you can use them with trace files too by just adding the -r<filename> switch.  
_Note_: if you need a list of basic sysdig commands, for instance to learn how to create a trace file, see the [quick reference guide](sysdig Quick Reference Guide#wiki-basic-command-list)
  
####Networking
* List all the incoming connections that are not served by apache.
> sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=httpd"

* Show the network data exchanged with the host 192.168.0.1  
> as binary:  
> sysdig -s2000 -X -c echo_fds fd.cip=192.168.0.1  
as ASCII:  
> sysdig -s2000 -A -c echo_fds fd.cip=192.168.0.1

* See the top processes in terms of network bandwidth usage
> sysdig -c topprocs_net

* See the top local server ports  
> in terms of established connections:  
> sysdig -c fdcount_by fd.sport "evt.type=accept"  
> in terms of total bytes:  
> sysdig -c fdbytes_by fd.sport

* See the top client IPs  
> in terms of established connections  
> sysdig -c fdcount_by fd.cip "evt.type=accept"  
> in terms of total bytes  
> sysdig -c fdbytes_by fd.cip

####Disk I/O
* See the top processes in terms of disk bandwidth usage
> sysdig -c topprocs_file

* List the processes that are using a high number of files
> sysdig -c fdcount_by proc.name "fd.type=file"

* See the top files in terms of read+write bytes
> sysdig -c topfiles_bytes

* Print the top files that apache has been reading from or writing to
> sysdig -c topfiles_bytes proc.name=httpd

####Processes and CPU usage
* See the top processes in terms of disk bandwidth usage
> sysdig -c topprocs_cpu

* See the top processes for CPU 0
> sysdig -c topprocs_cpu evt.cpu=0

* Observe the standard output of a process
> sysdig -s4096 -A -c stdout proc.name=cat

####Performance and Errors

* See all the failed file opens by httpd
> sysdig "proc.name=httpd and evt.type=open and evt.failed=true"

* See the files where most time has been spent
> sysdig -c topfiles_time

* See the files where apache spent most time
> sysdig -c topfiles_time proc.name=httpd

* See the top processes in terms of I/O errors
> sysdig -c topprocs_errors

* See the top files in terms of I/O errors
> sysdig -c topfiles_errors

* See the system calls where most time has been spent
> sysdig -c topscalls_time

* See the top system calls returning errors
> sysdig -c topscalls "evt.failed=true"

####Security

* Show the directories that the user "root" visits
> sysdig -p"%evt.arg.path" "evt.type=chdir and user.name=root"

* Observe ssh activity
> sysdig -A -c echo_fds fd.name=/dev/ptmx and proc.name=sshd

* Show every file open that happens in /etc
> sysdig evt.type=open and fd.name contains /etc