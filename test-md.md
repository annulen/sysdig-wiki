NAME
----

sysdig - Interactively dump and analyze system calls

SYNOPSIS
--------

**sysdig** [*option*]... [*filter*]

DESCRIPTION
-----------

sysdig is a tool for system troubleshooting, analysis and exploration. It can be used to capture, filter and decode system calls and other OS events. 
sysdig can be both used to inspect live systems, or to generate trace files that can be analyzed at a later stage.

Sysdig includes a powerul filtering language, has customizable output, and can be extended through Lua scripts, called chisels.

Output format:

By default, sysdig prints the information for each captured event on a single line, with the following format:

<evt.time> %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.args

where:
 evt.time is the event timestamp
 evt.cpu is the CPU number where the event was captured
 proc.name is the name of the process that generated the event
 thread.tid id the TID that generated the event, which corresponds to the
   PID for single thread processes
 evt.dir is the event direction, > for enter events and < for exit events
 evt.type is the name of the event, e.g. 'open' or 'read'
 evt.args is the list of event arguments.

The output format can be customized with the -p switch, use 'sysdig -l'
to list the available fields.
OPTIONS
-------

**-a**, **--abstime**
  Show absolute event timestamps
  
**-c** _chiselname_ _chiselargs_, **--chisel**=_chiselname_ _chiselargs_
  run the specified chisel. If the chisel require arguments, they must be specified in the command line after the name.
  
**-cl**, **--list-chisels**
  lists the available chisels. Looks for chisels in ., ./chisels, ~/chisels and /usr/share/sysdig/chisels.
  
**-dv**, **--displayflt**   
  Make the given filter a dsiplay one Setting this option causes the events to be filtered after being parsed by the state system. Events are normally filtered before being analyzed, which is more efficient, but can cause state (e.g. FD names) to be lost
  
**-h**, **--help**
  Print this page
  
**-j**, **--json**         
  Emit output as json
  
**-l**, **--list**
  List the fields that can be used for filtering and output formatting. Use -lv to get additional information for each field.
  
**-L**, **--list-events**  
  List the events that the engine supports
  
**-n** _num_, **--numevents**=_num_
  Stop capturing after <num> events
  
**-p** _output_format_, **--print**=_output_format_
  Specify the format to be used when printing the events. See the examples section below for more info.
  
**-q**, **--quiet**
  Don't print events on the screen. Useful when dumping to disk.
  
**-r** _readfile_, **--read**=_readfile_
  Read the events from <readfile>.
  
**-S**, **--summary**
  print the event summary (i.e. the list of the top events) when the capture ends.
  
**-s** _len_, **--snaplen**=_len_
  Capture the first <len> bytes of each I/O buffer. By default, the first 80 bytes are captured. Use this option with caution, it can generate huge trace files.
  
**-v**, **--verbose**
  Verbose output
  
**-w** _writefile_, **--write**=_writefile_
  Write the captured events to _writefile_.

FILES
-----

*/opt/sysdig/chisels*
  The global chisel directory.

*~/.chisels*
  The user chisel directory.

BUGS
----

Bugs?

AUTHOR
------

Draios inc. <info@draios.com>

SEE ALSO
--------

strace(8)
