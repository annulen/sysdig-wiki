This is an ever growing list of cool things you can do with sysdig commands.  
  
Note: For a reference list of basic sysdig commands, see the [quick reference guide](sysdig Quick Reference Guide#wiki-basic-command-list)  
  
* Print the top files that apache has been reading or writing to
'sysdig -c topfiles "proc.name=httpd"'

* Show the directories that the user "root" visits
> sysdig -p"%evt.arg.path" "evt.type=chdir and user.name=root"

* List all the incoming connections that are not served by apache.
> sysdig -p"%proc.name) %fd.name" "evt.type=accept and proc.name!=httpd"