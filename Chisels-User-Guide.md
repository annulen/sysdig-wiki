Sysdig’s chisels are little scripts that analyze the sysdig event stream to perform useful actions. 
If you’ve used system tracing tools like dtrace, you’re probably familiar with running scripts that trace OS events. Usually, with dtrace-like tools you write your scripts using a domain-specific language that gets compiled into bytecode and injected in the kernel. Draios uses a different approach: events are efficiently brought to user-level, enriched with context, and then scripts can be applied to them. This brings several benefits:

* A well known scripting language can be used instead of a custom one. In fact, sysdig’s chisels are Lua scripts. Lua is well known, powerful, stable and extremely efficient.
* Chisels can leverage the broad collection of Lua libraries.
* Chisels work well on live systems, but can also be used with trace files for offline analysis.  

To get the list of available chisels, just type

>$ sysdig –cl  

You will get a short description for each of the available chisels.  
If you need a detailed description and the list of arguments for one of the chisels, you use the -i command line switch:

>$ sysdig -ispy_ip

To run one of the chisels, you use the –c flag. For instance, let’s run the topfiles_bytes chisel:

>$ sysdig –c topfiles_bytes

And if a chisel needs arguments, you specify them after the chisel name:

>$ sysdig –c spy_ip 192.168.1.157

Chisels can be combined with filters, which usually makes them much more useful. For example, let’s take the simple topfiles_bytes chisel. If we run it without arguments, it will show us the most accessed files on the whole machine.

>./sysdig -c topfiles_bytes  
Bytes     Filename  
------------------------------  
23.32KB   /proc/net/unix  
9.11KB    /usr/share/icons/hicolor/16x16/actions/terminator_receive_off.png  
5.64KB    /etc/localtime  
4.92KB    /proc/interrupts  
4.38KB    /dev/input/event2  
4.37KB    /etc/wgetrc  
2.88KB    /proc/stat  
2.39KB    /usr/share/locale/locale.alias  
1.85KB    /proc/18263/status  

Let’s say we’re not interested in accesses to /dev. We can filter it out with something like this

>$ sysdig -c topfiles_bytes "not fd.name contains /dev"  
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

>$ sysdig -c topfiles_bytes "fd.name contains /root"  
--14:14:14------------------------------------------  
1.98KB    /root/agent/build/debug/test/lo.txt  
16B       /root/.dropbox/config.dbx  

Or the ones accessed by a specific process:

>$ sysdig -c topfiles_bytes "proc.name=vi"  
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

>$ sysdig -c topfiles_bytes "user.name=loris"  
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