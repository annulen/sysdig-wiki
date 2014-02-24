###Sysdig is a tool for system-level exploration and troubleshooting.
We built sysdig to give admins and developers unprecedented visibility into behavior of their systems. Sysdig lets you capture activity from a running Linux system, and then lets you save, filter and display that activity in real time. Think about it as strace + tcpdump + lsof + steroids.

###How does it work?
Sysdig instruments your real and virtual machines at the OS level by installing into the Linux kernal and capturing system calls and other OS events. Then using sysdig's command line interface, you can filter and decode these events in order to extract useful information. Sysdig can be used to inspect systems live in real-time, or to generate trace files that can be analyzed at a later stage.

###How can I get the most out of sysdig?
Sysdig includes a powerful filtering language, has customizable output, and can be extended through Lua scripts, which we call "chisels." Please explore this wiki where you will find a variety of documents that will walk you through the full functionality of sysdig. [Here are some examples](sysdig Examples) of cool things you can do with sysdig!