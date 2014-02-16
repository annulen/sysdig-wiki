sysdig chisels are Lua scripts that can be invoked from the sysdig command line, hook into the sysdig engine, and can be used to extend sysdig with advanced funcionality.

Chisels talk with sysdig using of three separate interfaces: 
* the sysdig library, with functions to interact with the main program
* the evt library, used to extract information from a captured event
* a set of callbacks that get called when something interesting happens, e.g. when an event is received 

This page documents each of these interfaces.

## sysdig library
The functions in this library are mostly related to setting up the chisel environment and are usually called at initialization time, i.e. inside on_init().

**sysdig.request_field(fld_name)**

This function is used to configure the sysdig engine to extract a filter field when an event is captured. _fld_name_ is the name of the sysdig filter field to extract (see the sysdig tutorial or type _sysdig -l_ for a list of available fields).
The function returns a field handle that can be fed to evt.field() to get the field value from the on_event() callback. 

**sysdig.set_filter(filter)**

Configure the sysdig engine to apply the given filter before handing the events to this chisel's on_event() callback.

_Notes_
* The filter set with set_filter() is private for this chisels and won't influence other chisels that are run from the same command line.
* You can set only one filter per chisel. Calling set_filter() twice will cause the first filter to be overridden.
* It's very important to be aggressive as possible with filters. The sysdig engine is heavily optimized, so e,liminating as many events as possible before reaching the chisel's on_event() will make the chisel much more efficient.

**sysdig.set_snaplen(snaplen)**

Configure the number of bytes that are captured from buffers of I/O system calls like read(), write(), sendto(), recvfrom(), etc.

**sysdig.set_event_formatter(format)**

Configure an event formatter. _format_ is a string containing a list of fields to print, with the same syntax that you would use fwith the -p sysdig command line switch (refer to the sysdig manual for more information).

By default, chisels have no event formatter, and that gives them the freedom to print whatever they want using Lua's print() function. Setting a formatter, on the other hand, lets you delegate event formatting to sysdig, leveraging sysdig's filter fields system. Note that only events for which on_event() returns _true_ are going to be printed.

**sysdig.set_interval_s(interval)**

Set a periodic callback for this chisel. If you use this function, the chisel needs to include a function called on_interval(), which the engine will call every _interval_ seconds. This can be used to perform periodic tasks, like printing information to the screen once a second.

**sysdig.set_interval_ns(interval)**

Like set_interval_s(), but allows more granular timeouts. 

## evt library
The function in this library are related to the event that is currently processed and therefore can only be called from the on_event() callback.
 
**evt.field(fld)**

Extract a field's value from an event. _fld_ is a field handle obtained from a previous call to request_field().
The function returns the field value, which can be a number or a string depending on the field. Use _sysdig -lv_ to find out the type of each exported field. 

_Notes_
* If the requested field is not exported by an event (e.g. non I/O events don't export the fd.name field), the return value of field() will be nil.
*  one important field you need to be aware of is evt.rawarg. This field can be used to extract arbitrary system call arguments, e.g. evt.rawarg.res, and its type depends on the field you're asking. Use _sysdig -L_ to find out the type of event arguments.

**evt.get_num()**

Return the incremental event number.

**evt.get_ts()**

Return the raw event timestamp, expresses as nanoseconds since epoch. 

**evt.get_type()**

Returns the event type as a number, and can be used for efficient event filtering. For the list of event type numbers, please refer to the ppm_event_type enumeration in driver/ppm_event_events_public.h.

## callbacks
**on_set_arg(name, val)**

**on_init()**

**on_event()**

**on_capture_end()**

**on_interval()**