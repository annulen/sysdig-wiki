**Here are some examples of commonly used sysdig command formats.**

Capture all the events from the live system and print them to screen
> sysdig

Capture all the events from the live system and save them to disk
> sysdig -qw dumpfile.scap

Read events from a file and print them to screen
> sysdig -r dumpfile.scap

Print all the open system calls invoked by cat
> sysdig proc.name=cat and evt.type=open

Print the name of the files opened by cat
> ./sysdig -p"%evt.arg.name" proc.name=cat and evt.type=open

List the available chisels
> ./sysdig -cl

Run the spy_ip chisel for the 192.168.1.157 IP address:
> sysdig –c spy_ip 192.168.1.157