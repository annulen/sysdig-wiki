###Table of Contents  
[The Basics](#wiki-the-basics)  
[Trace Files](#wiki-trace-files)  
[Filtering](#wiki-filtering)  
[Output Formatting](#wiki-output-formatting)  
[Scripting with "Chisels"](#wiki-chiseling)  

### The Basics
The simplest way to use sysdig is by invoking it without any argument. Doing this will cause sysdig to capture every event and write it to standard output, very much like strace does.

> $ sysdig  
09:34:35.807452316 0 httpd (59416) > writev fd=13 size=270  
09:34:35.807452847 2 ab (59421) > write fd=17(<4>127.0.0.1:40370->127.0.0.1:80) size=77  
09:34:35.807477353 0 httpd (59416) < writev res=270 data=HTTP/1.1 302 Found..Date: Fri, 10 Jan 2014 17:34:35 GMT..Server: Apache/2.4.4 (U  
09:34:35.807487713 2 ab (59421) < write res=77 data=GET / HTTP/1.0..Host: 127.0.0.1..User-Agent: ApacheBench/2.3..Accept: */*....  
09:34:35.807489930 2 ab (59421) > connect fd=6(<4>127.0.0.1:40371->127.0.0.1:80)  
09:34:35.807492363 2 ab (59421) < connect res=0 tuple=4127.0.0.1:40371->127.0.0.1:80  
09:34:35.807493225 0 httpd (59416) > write fd=9(<f>/opt/lampp/logs/access_log (deleted)) size=66  
09:34:35.807494292 2 ab (59421) > epoll_ctl  

By default, sysdig prints the information for each event on a single line, with the following format:

```%evt.time %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.args```

where:
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
* For most system calls, sysdig shows two separate lines: an enter one (marked with a ‘>’) and an exit one (marked with a ‘<’). This makes it easier to follow the trace in case of context switches, especially in multi-processor environments.
* File descriptors are resolved. This means that, whenever possible, the FD number is followed by a human-readable representation of the FD itself: a tuple for network connections, a name for files, and so on. 
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
19:28:06.634075871 2 cat (45854) < execve res=0 exe=cat args= tid=45854(cat) pid=45854(cat) ptid=38714(bash) cwd=/root/agent/build/debug/test fdlimit=1024  
19:28:06.634149757 2 cat (45854) > brk size=0  
19:28:06.634151608 2 cat (45854) < brk res=6557696  
19:28:06.634176617 2 cat (45854) > mmap  
19:28:06.634182286 2 cat (45854) < mmap  
19:28:06.634198501 2 cat (45854) > access  
19:28:06.634210969 2 cat (45854) < access  
19:28:06.634222791 2 cat (45854) > open  
19:28:06.634236235 2 cat (45854) < open fd=3(<f>/etc/ld.so.cache) name=/etc/ld.so.cache flags=0 mode=0 
19:28:06.634270260 2 cat (45854) > fstat fd=3(<f>/etc/ld.so.cache)  
19:28:06.634274365 2 cat (45854) < fstat res=0  
19:28:06.634275117 2 cat (45854) > mmap  
19:28:06.634280896 2 cat (45854) < mmap  
19:28:06.634281325 2 cat (45854) > close fd=3(<f>/etc/ld.so.cache)  
19:28:06.634282155 2 cat (45854) < close res=0  
19:28:06.634301578 2 cat (45854) > open 
19:28:06.634316194 2 cat (45854) < open fd=3(<f>/lib64/libc.so.6) name=/lib64/libc.so.6 flags=0 mode=0

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

###Chiseling

Sysdig’s chisels are little scripts that analyze the sysdig event stream to perform useful actions. 
If you’ve used system tracing tools like dtrace, you’re probably familiar with running scripts that trace OS events. Usually, with dtrace-like tools you write your scripts using a domain-specific language that gets compiled into bytecode and injected in the kernel. Draios uses a different approach: events are efficiently brought to user-level, enriched with context, and then scripts can be applied to them. This brings several benefits:

* A well known scripting language can be used instead of a custom one. In fact, sysdig’s chisels are LUA scripts. LUA is well known, powerful, and extremely efficient.
* Chisels can leverage the broad collection of LUA libraries.
* Chisels work well on live systems, but can also be used with trace files for offline analysis.  

To get the list of available chisels, just type

>$ sysdig –cl  

For each chisel, you get the description and the list of arguments it expects. 
To run one of the chisels, you use the –c flag. For instance, let’s run the topfiles chisel:

>$ sysdig –c topfiles

And if a chisel needs arguments, you specify them after the chisel name:

>$ sysdig –c spy_ip 192.168.1.157

Chisels can be combined with filters, which usually makes them much more useful. For example, let’s take the simple topfiles chisel. If we run it without arguments, it will show us the most accessed files on the whole machine.

>./sysdig -c topfiles  
--14:08:14------------------------------------------  
569.74KB  /dev/kmsg  
4.88KB    /lib64/libc.so.6  
4.77KB    /usr/share/locale/locale.alias  
2.00KB    /proc/meminfo  
1.62KB    /lib64/libdl.so.2  
1.62KB    /lib64/libtinfo.so.5  
1.28KB    /dev/ptmx  
812B      /dev/pts/11  
664B      /dev/null  

Let’s say we’re not interested in accesses to /dev. We can filter it out with something like this

>$ sysdig -c topfiles "not fd.name contains /dev"  
--14:10:24------------------------------------------  
5.64KB    /etc/localtime  
4.88KB    /lib64/libc.so.6  
4.77KB    /usr/share/locale/locale.alias  
2.26KB    /proc/mounts  
2.18KB    /etc/passwd  
2.00KB    /proc/meminfo  
1.66KB    /etc/nsswitch.conf  
1.62KB    /lib64/libdl.so.2  
1.62KB    /lib64/libtinfo.so.5  

Or maybe we want to see the top files in a specific folder:

>$ sysdig -c topfiles "fd.name contains /root"  
--14:14:14------------------------------------------  
1.98KB    /root/agent/build/debug/test/lo.txt  
16B       /root/.dropbox/config.dbx  

Or the ones accessed by a specific process:

>$ sysdig -c topfiles "proc.name=vi"  
--14:18:35------------------------------------------  
4.00KB    /root/agent/build/debug/test/.lo.txt.swp  
3.36KB    /usr/share/terminfo/x/xterm-256color  
2.18KB    /etc/passwd  
1.98KB    /root/agent/build/debug/test/lo.txt  
1.92KB    /etc/virc  
1.66KB    /etc/nsswitch.conf  
832B      /lib64/libpcre.so.1  
832B      /lib64/libc.so.6  
832B      /lib64/libnss_files.so.2  

Or by a specific user:

>$ sysdig -c topfiles "user.name=loris"  
--14:15:43------------------------------------------  
3.31KB    /etc/nsswitch.conf  
2.18KB    /etc/passwd  
1.62KB    /lib64/libselinux.so.1  
1.62KB    /lib64/libc.so.6  
1.62KB    /lib64/libpcre.so.1  
1.62KB    /lib64/libdl.so.2  
1.62KB    /lib64/libnss_files.so.2  
898B      /etc/group  
54B       /proc/self/task/30414/attr/current  

One last thing to remember about chisels is that you can run as many as you want at the same time. For example,

>$ sysdig -c stdin -c stdout proc.name=cat

will replay both standard input and standard output of cat.
As mentioned before, chisels are written in LUA, so customizing them or writing new ones is very easy and pretty fun. The [chisel tutorial](Writing a sysdig Chisel, a Tutorial) and 
[chisel API reference](sysdig Chisel API Reference Manual) include all you need to get started.  