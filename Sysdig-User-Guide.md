###Table of Contents  
* [The Basics](#wiki-the-basics)  
* [Trace Files](#wiki-trace-files)  
* [Filtering](#wiki-filtering)  
* [Output Formatting](#wiki-output-formatting)  
* [Chisels](#wiki-chisels)

### The Basics
The simplest way to use sysdig is by invoking it without any argument. Doing this will cause sysdig to capture every event and write it to standard output, very much like strace does.

> $ sysdig  
34378 12:02:36.269753803 2 echo (7896) > close fd=3(<f>/usr/lib/locale/locale-archive)  
34379 12:02:36.269754164 2 echo (7896) < close res=0  
34380 12:02:36.269781699 2 echo (7896) > fstat fd=1(<f>/dev/pts/3)  
34381 12:02:36.269783882 2 echo (7896) < fstat res=0  
34382 12:02:36.269784970 2 echo (7896) > mmap  
34383 12:02:36.269786575 2 echo (7896) < mmap  
34384 12:02:36.269827674 2 echo (7896) > write fd=1(<f>/dev/pts/3) size=12  
34385 12:02:36.269839477 2 echo (7896) < write res=12 data=hello world.  
34386 12:02:36.269843986 2 echo (7896) > close fd=1(<f>/dev/pts/3)  
34387 12:02:36.269844466 2 echo (7896) < close res=0  
34388 12:02:36.269844816 2 echo (7896) > munmap  
34389 12:02:36.269850803 2 echo (7896) < munmap  
34390 12:02:36.269851915 2 echo (7896) > close fd=2(<f>/dev/pts/3)  
34391 12:02:36.269852314 2 echo (7896) < close res=0  

By default, sysdig prints the information for each event on a single line, with the following format:

```%evt.num %evt.time %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.args```

where:
* evt.num is the incremental event number
* evt.time is the event timestamp
* evt.cpu is the CPU number where the event was captured
* proc.name is the name of the process that generated the event
* thread.tid is the TID that generated the event, which corresponds to the PID for single thread processes
* evt.dir is the event direction, > for enter events and < for exit events
* evt.type is the name of the event, e.g. 'open' or 'read'
* evt.args is the list of event arguments. In case of system calls, these tend to correspond to the system call arguments, but that’s not always the case: some system call arguments are excluded for simplicity or performance reasons.

**NOTE**: 
Not all of the system calls are currently decoded by sysdig. Non-decoded system calls are still shown in the output, but with no arguments.

By looking at the output, you can immediately spot some key differences between this output and the strace one:
* For most system calls, sysdig shows two separate entries: an enter one (marked with a ‘>’) and an exit one (marked with a ‘<’). This makes it easier to follow the trace in multi-process environments.
* File descriptors are resolved. This means that, whenever possible, the FD number is followed by a human-readable representation of the FD itself: the tuple for network connections, the name for files, and so on. 
The exact format used to render an FD is the following: 
num(\<type\>resolved_string)
where: 
 * num is the FD number
 * resolved_string is the resolved representation of the FD, e.g. 127.0.0.1:40370->127.0.0.1:80 for a TCP socket
 * type is a single-letter-encoding of the fd type, and can be one of the following:
   * f for files  
    * 4 for IPv4 sockets
    * 6 for IPv6 sockets
    * u for unix sockets
    * s for signal FDs
    * e for event FDs
    * i for inotify FDs
    * t for timer FDs

###Trace Files
Sysdig lets you save the captured events to disk so that they can be analyzed at a later time. The syntax is the following:

>$ sysdig –w myfile.scap

If you want to limit the number of events saved to the file to 100, you can use the –n flag:

>$ sysdig –n 100 –w myfile.scap

Reading a previously saved trace file can be done with the –r flag:

>$ sysdig –r myfile.scap

Note that sysdig saves a full snapshot of the OS in each capture file (running processes, open files, user names…), and this means that no information is lost when doing offline analysis.
Note also that you can download a MAC and Windows version of sysdig. They won’t be able to do live capture, but they can be used to analyze trace files that have been captured under Linux.

###Filtering

Now that we took care of the basics, let’s start having some fun. Sysdig’s filtering system is powerful and versatile, and is designed to look for needles in a haystack. Filters are specified at the end of the command line, like in tcpdump, and can be applied to both a live capture or a trace file. For example, let’s look at the activity of a specific command, in this case cat:
> $ ./sysdig proc.name=cat  
21368 13:10:15.384878134 1 cat (8298) < execve res=0 exe=cat args=index.html. tid=8298(cat) pid=8298(cat) ptid=1978(bash) cwd=/root fdlimit=1024  
21371 13:10:15.384948635 1 cat (8298) > brk size=0  
21372 13:10:15.384949909 1 cat (8298) < brk res=10665984  
21373 13:10:15.384976208 1 cat (8298) > mmap  
21374 13:10:15.384979452 1 cat (8298) < mmap  
21375 13:10:15.384990980 1 cat (8298) > access  
21376 13:10:15.384999211 1 cat (8298) < access  
21377 13:10:15.385008602 1 cat (8298) > open  
21378 13:10:15.385014374 1 cat (8298) < open fd=3(<f>/etc/ld.so.cache) name=/etc/ld.so.cache flags=0(O_NONE) mode=0  
21379 13:10:15.385015508 1 cat (8298) > fstat fd=3(<f>/etc/ld.so.cache)  
21380 13:10:15.385016588 1 cat (8298) < fstat res=0  
21381 13:10:15.385017033 1 cat (8298) > mmap  
21382 13:10:15.385019763 1 cat (8298) < mmap  
21383 13:10:15.385020047 1 cat (8298) > close fd=3(<f>/etc/ld.so.cache)  
21384 13:10:15.385020556 1 cat (8298) < close res=0  

As you can see, sysdig doesn’t attach to processes. It just captures everything and then lets you filter out what you’re not interested in. Filter statements can use the standard comparison operators(=, !=, <, <=, >, >=, contains) and can be combined using Boolean operators (and, or and not) and brackets. For example

>$ sysdig proc.name=cat or proc.name=vi

captures the activity of both cat and vi, while

>$ sysdig proc.name!=cat and evt.type=open

shows all the files that are opened by programs that are not cat. Filter fields are expressed as _class.field_. A quick way to get a list of the available classes and the fields they include is

>$ sysdig -l

As a reference, here’s a list of the fields you can use:
```
Field Class: fd

fd.num         the unique number identifying the file descriptor.
fd.type         type of FD. Can be 'file', 'ipv4', 'ipv6', 'unix', 'pipe', 'event', 'signalfd', 'eventpoll', 'inotify' or 'signalfd'.
fd.typechar     type of FD as a single character. Can be 'f' for file, 4 for IPv4 socket, 6 for IPv6 socket, 'u' for unix socket, p for pipe, 'e' for eventfd, 's' for signalfd, 'l' for eventpoll, 'i' for inotify, or 's' for signalfd, 'o' for uknown.
fd.name         FD full name. If the fd is a file, this field contains the fullpath. If the FD is a socket, this field contain the connection tuple.
fd.ip           matches the ip address (client or server) of the fd.
fd.cip          client IP address.
fd.sip          server IP address.
fd.port         matches the port (client or server) of the fd.
fd.cport        for TCP/UDP FDs, the client port.
fd.sport        for TCP/UDP FDs, server port.
fd.l4proto      the IP protocol number.
fd.sockfamily   the socket family for socket events. Can be 'ip' or 'unix'.
fd.is_server    'true' if the process owning this FD is the server endpoint in the connection.

Field Class: process

proc.pid        the id of the process generating the event.
proc.exe        the full name (including the path) of the executable generating the event.
proc.name       the name (excluding thr path) of the executable generating the event.
proc.args       the arguments passed on the command line when starting the process generating the event.
proc.cwd        the current working directory of the event.
proc.nchilds    the number of child threads of that the process generating the event currently has.
thread.tid      the id of the thread generating the event.
thread.ismain   'true' if the thread generating the event is the main one in the process.
proc.parentname the name (excluding thr path) of the parent of the process generating the event.
iobytes         I/O bytes (either read or write) generated by I/O calls like read, write, send, receive...
totiobytes      aggregated number of I/O bytes (either read or write) since the beginning of the capture.
latency         number of nanoseconds spent in the last system call.
totlatency      aggregated number of nanoseconds spent in system calls since the beginning of the capture.

Field Class: evt

evt.num         event number.
evt.time        event timestamp as a time string.
evt.time.s      event timestamp as a time string.
evt.datetime    event timestamp as a time string that includes the date.
evt.rawtime     absolute event timestamp, i.e. nanoseconds from epoch.
evt.rawtime.s   integer part of the event timestamp (e.g. seconds since epoch).
evt.rawtime.ns  fractional part of the absolute event timestamp.
evt.reltime     number of nanoseconds from the beginning of the capture.
evt.reltime.s   number of seconds from the beginning of the capture.
evt.reltime.ns  fractional part (in ns) of the time from the beginning of the capture.
evt.latency     delta between an exit event and the correspondent enter event.
evt.dir         event direction can be either '>' for enter events or '<' for exit events.
evt.type        For system call events, this is the name of the system call (e. g. 'open').
evt.cpu         number of the CPU where this event happened.
evt.args        all the event arguments, aggregated into a single string.
evt.arg         one of the event arguments specified by name or by number. Some events (e.g. return codes or FDs) will be converted into a text representation when possible. E.g. 'resarg.fd' or 'resarg[0]'
                .
evt.rawarg      one of the event arguments specified by name. E.g. 'arg.fd'.
evt.res         event return value, as an error code string (e.g. 'ENOENT').
evt.rawres      event return value, as a number (e.g. -2). Useful for range comparisons.
evt.is_io       'true' for events that read or write to FDs, like read(), send, recvfrom(), etc.
evt.is_io_read  'true' for events that read from FDs, like read(), recv(), recvfrom(), etc.
evt.is_io_write 'true' for events that write to FDs, like write(), send(), etc.

Field Class: user

user.id         user ID.
user.name       user name.
user.homedir    home directory of the user.
user.shell      user's shell.

Field Class: group

group.gid       group ID.
group.name      group name.
```

As you can see, enough to do plenty of creative digging. For example, thanks to the fact that sysdig resolves file descriptors, you can do stuff like this:

>$ sysdig evt.type=accept and proc.name!=apache

to see the incoming network connections received by processes other than apache.  
But that’s not all.  
There are a couple of special fields, evt.arg and evt.rawarg, that deserve additional explanation. Every event that sysdig captures has a type (e.g. 'open', 'read'...), and a set of parameters (e.g. 'fd', 'name'...) that are encoded using a standardize type system. I know this sounds boring, so let’s just talk about the benefits of it: any parameter of any event can be used in filters. For example:

>$ sysdig evt.type=execve and evt.arg.ptid=bash

shows the programs that are run by interactive users, through a filter that keeps the execve system calls (which are used to execute programs), but only if the parent process name is ‘bash’.
The difference between evt.arg and event.rawarg is that the second doesn’t do resolution of PIDs, FDs, error codes, etc, and leaves the argument in its raw numeric form. For example, you can use

>$ sysdig evt.arg.res=ENOENT

To filter on a specific I/O error code or, since error codes are negative, this

>$ sysdig " evt.rawarg.res<0 or evt.rawarg.fd<0"

will give you all the system calls that produced errors.
To get a list of all the events you can use in your filters, plus their parameters, type

>$ sysdig –L

And, for your reference, here’s the list, where ‘>’ indicates and enter event and ‘<’ indicates an exit event:

```
> syscall(SYSCALLID ID, UINT16 nativeID)
< syscall(SYSCALLID ID)
> open()
< open(FD fd, FSPATH name, UINT32 flags, UINT32 mode)
> close(FD fd)
< close(ERRNO res)
> read(FD fd, UINT32 size)
< read(ERRNO res, BYTEBUF data)
> write(FD fd, UINT32 size)
< write(ERRNO res, BYTEBUF data)
> brk(UINT32 size)
< brk(UINT64 res)
> execve()
< execve(ERRNO res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, UINT64 fdlimit)
> clone()
< clone(PID res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, INT64 fdlimit, UINT32 flags, UINT32 uid, UINT32 gid)
> procexit()
> socket(UINT32 domain, UINT32 type, UINT32 proto)
< socket(FD fd)
> bind(FD fd)
< bind(ERRNO res, SOCKADDR addr)
> connect(FD fd)
< connect(ERRNO res, SOCKTUPLE tuple)
> listen(FD fd, UINT32 backlog)
< listen(ERRNO res)
> accept()
< accept(FD fd, SOCKTUPLE tuple, UINT8 queuepct)
> send(FD fd, UINT32 size)
< send(ERRNO res, BYTEBUF data)
> sendto(FD fd, UINT32 size, SOCKTUPLE tuple)
< sendto(ERRNO res, BYTEBUF data)
> recv(FD fd, UINT32 size)
< recv(ERRNO res, BYTEBUF data)
> recvfrom(FD fd, UINT32 size)
< recvfrom(ERRNO res, BYTEBUF data, SOCKTUPLE tuple)
> shutdown(FD fd, UINT8 how)
< shutdown(ERRNO res)
> getsockname()
< getsockname()
> getpeername()
< getpeername()
> socketpair(UINT32 domain, UINT32 type, UINT32 proto)
< socketpair(ERRNO res, FD fd1, FD fd2, UINT64 source, UINT64 peer)
> setsockopt()
< setsockopt()
> getsockopt()
< getsockopt()
> sendmsg(FD fd, UINT32 size, SOCKTUPLE tuple)
< sendmsg(ERRNO res, BYTEBUF data)
> sendmmsg()
< sendmmsg()
> recvmsg(FD fd)
< recvmsg(ERRNO res, UINT32 size, BYTEBUF data, SOCKTUPLE tuple)
> recvmmsg()
< recvmmsg()
> accept(INT32 flags)
< accept(FD fd, SOCKTUPLE tuple, UINT8 queuepct)
> creat()
< creat(FD fd, FSPATH name, UINT32 mode)
> pipe()
< pipe(ERRNO res, FD fd1, FD fd2, UINT64 ino)
> eventfd(UINT64 initval, UINT32 flags)
< eventfd(FD res)
> futex(UINT64 addr, UINT16 op, UINT64 val)
< futex(ERRNO res)
> stat()
< stat(ERRNO res, FSPATH path)
> lstat()
< lstat(ERRNO res, FSPATH path)
> fstat(FD fd)
< fstat(ERRNO res)
> stat64()
< stat64(ERRNO res, FSPATH path)
> lstat64()
< lstat64(ERRNO res, FSPATH path)
> fstat64(FD fd)
< fstat64(ERRNO res)
> epoll_wait(ERRNO maxevents)
< epoll_wait(ERRNO res)
> poll(FDLIST fds, INT64 timeout)
< poll(ERRNO res, FDLIST fds)
> select()
< select(ERRNO res)
> newselect()
< newselect(ERRNO res)
> lseek(FD fd, UINT64 offset, UINT8 whence)
< lseek(ERRNO res)
> llseek(FD fd, UINT64 offset, UINT8 whence)
< llseek(ERRNO res)
> ioctl(FD fd, UINT64 request)
< ioctl(ERRNO res)
> getcwd()
< getcwd(ERRNO res, CHARBUF path)
> chdir()
< chdir(ERRNO res, CHARBUF path)
> fchdir(FD fd)
< fchdir(ERRNO res)
> mkdir(FSPATH path, UINT32 mode)
< mkdir(ERRNO res)
> rmdir(FSPATH path)
< rmdir(ERRNO res)
> openat(FD dirfd, CHARBUF name, UINT32 flags, UINT32 mode)
< openat(FD fd)
> link(FSPATH oldpath, FSPATH newpath)
< link(ERRNO res)
> linkat(FD olddir, CHARBUF oldpath, FD newdir, CHARBUF newpath)
< linkat(ERRNO res)
> unlink(FSPATH path)
< unlink(ERRNO res)
> unlinkat(FD dirfd, CHARBUF name)
< unlinkat(ERRNO res)
> pread(FD fd, UINT32 size, UINT64 pos)
< pread(ERRNO res, BYTEBUF data)
> pwrite(FD fd, UINT32 size, UINT64 pos)
< pwrite(ERRNO res, BYTEBUF data)
> readv(FD fd)
< readv(ERRNO res, UINT32 size, BYTEBUF data)
> writev(FD fd, UINT32 size)
< writev(ERRNO res, BYTEBUF data)
> preadv(FD fd, UINT64 pos)
< preadv(ERRNO res, UINT32 size, BYTEBUF data)
> pwritev(FD fd, UINT32 size, UINT64 pos)
< pwritev(ERRNO res, BYTEBUF data)
> dup(FD fd)
< dup(FD res)
> signalfd(FD fd, UINT32 mask, UINT8 flags)
< signalfd(FD res)
> kill(PID pid, SIGTYPE sig)
< kill(ERRNO res)
> tkill(PID tid, SIGTYPE sig)
< tkill(ERRNO res)
> tgkill(PID pid, PID tid, SIGTYPE sig)
< tgkill(ERRNO res)
> nanosleep(RELTIME interval)
< nanosleep(ERRNO res)
> timerfd_create(UINT8 clockid, UINT8 flags)
< timerfd_create(FD res)
> inotify_init(UINT8 flags)
< inotify_init(FD res)
> getrlimit(UINT8 resource)
< getrlimit(ERRNO res, INT64 cur, INT64 max)
> setrlimit(UINT8 resource)
< setrlimit(ERRNO res, INT64 cur, INT64 max)
> prlimit(PID pid, UINT8 resource)
< prlimit(ERRNO res, INT64 newcur, INT64 newmax, INT64 oldcur, INT64 oldmax)
> switch(PID next)
```

Please just remember that the list changes with every new release, so make sure to use the program for the most updated one.

###Output Formatting 

Did you take some time to experiment with filtering and filter fields? Good, because now we’re going to learn how to use the same fields to customize what sysdig prints to the screen. Another really nice benefit of the type of system sysdig uses to encode fields, events and parameters is that they can all be used to customize the program output. Output customization happens with the –p command line flag, and works somewhat similarly to the C printf syntax. Here’s an example:

>$ sysdig -p"user:%user.name dir:%evt.arg.path" evt.type=chdir  
ubuntu) /root  
ubuntu) /root/tmp  
ubuntu) /root/Download  

This one-liner filters on the chdir system calls (the ones that get called every time a user does a cd), and prints the user name and the directory where the user is going. Essentially, it lets you follow a user as she moves in the file system.

Some notes about the –p formatting syntax:
* Fields must be prepended with a %
* You can add arbitrary text in the string, exactly as you would do in the C printf.
* By default, a line is printed only if **all** the fields specified by –p are present in the event. You can, however, prepend the string with a * to make it prints no matter what. In that case, the missing fields will be rendered as \<NA\>.

For example,

>$ sysdig -p"%evt.type %evt.dir %evt.arg.name" evt.type=open

will only print exit open events, like this 

>open < /proc/1285/task/1399/stat  
open < /proc/1285/task/1400/io  
open < /proc/1285/task/1400/statm  

because the enter events don’t contain the name argument, while 

>$ sysdig -p"*%evt.type %evt.dir %evt.arg.name" evt.type=open

will print both enter and exit open events, with a line finishing with \<NA\> for enter events:

>open > \<NA\>  
open < /proc/1285/task/1399/stat  
open > \<NA\>  
open < /proc/1285/task/1400/io  
open > \<NA\>  
open < /proc/1285/task/1400/statm  
open > \<NA\>  

Putting together filtering and output formatting makes sysdig a very flexible and powerful tool. Here are some examples:

>$ sysdig -s 65000 "-p%evt.rawarg.data" "proc.name=cat and evt.type=write and fd.num=1"

prints the standard output of a process (cat in this case). Note how we use the –s switch to capture more than the usual 80 bytes of each write. Use that flag with caution, it can generate huge trace files!

>$ sysdig -p"%user.name) %proc.name %proc.args" evt.type=execve and evt.arg.ptid=bash

shows user, command name and arguments for every program launched by a real user (i.e. from bash).

>$ sysdig -p"%user.name) %evt.arg.path" "evt.type=chdir"

shows the directories that interactive users visit

>$ sysdig -p"user:%user.name process:%proc.name file:%fd.name" "evt.type=write and fd.name contains /etc/”

prints user, process, and file name for all the accesses to the /etc directory.

>$ sysdig -p"%fd.name" “proc.name=apache and evt.type=accept"

lists TCP/IP endpoint information for all the connections received by apache.
Well, you get the idea. :-)
Are you interested in learning more useful sysdig one-liners? Visit the sysdig website on a regular basis or, even better, follow us on twitter. We’ll post new one-liners on a regular basis.

###Chisels

Sysdig’s chisels are little scripts that analyze the sysdig event stream to perform useful actions. Essentially, they enable you to do really cool stuff with your sysdig data. Just dig the data up, and then use a chisel to shape it into something beautiful. Get it? Awesome!

Chisels are written in LUA, a well known, powerful, and extremely efficient scripting language.

[Go here](Chisels User Guide) for a full tutorial.