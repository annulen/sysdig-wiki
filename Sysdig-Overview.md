###Sysdig is a tool for system-level exploration and troubleshooting.
We built sysdig to give admins and developers unprecedented visibility into the behavior of their systems. Traditionally, system-level monitoring and troubleshooting still largely involve logging into the
machine with SSH and using a plethora of dated tools with very inconsistent interfaces. And
since most tools don’t offer any kind of history, you’re left struggling to reproduce the problem.
Or even worse, just staring at the screen hoping that it happens again. Sysdig wants to improve your life, by introducing some key concepts:

* offering unified, coherent, and granular visibility into the storage, processing, network, and memory subsystems 
* making it possible to create trace files for system activity, similarly to what you can do for networks with tools like tcpdump and Wireshark, so that the problem can be analyzed at a later time, without losing important information
* including rich system state in the trace files, so that the captured activity can be put in context and understood
* offering a filtering language to dig into the information in a natural and interactive way
* including a rich library of Lua scripts to solve common problems, which we call [chisels](Chisels Overview) (to carve up the data you unearthed... get it?).
Think about sysdig as strace + tcpdump + lsof + steroids.

###How does it work?
Sysdig instruments your physical and virtual machines at the OS level by installing into the Linux kernel and capturing system calls and other OS events. Then, using sysdig's command line interface, you can filter and decode these events in order to extract useful information. Sysdig can be used to inspect systems live in real-time, or to generate trace files that can be analyzed at a later stage.

###How can I get the most out of sysdig?
Please explore this wiki where you will find a variety of documents that will walk you through the full functionality of sysdig. For example, [here are some examples](sysdig Examples) of cool things you can do with sysdig, and [here's a user guide](Sysdig User Guide) that will introduce you to using the tool.

Happy digging!
