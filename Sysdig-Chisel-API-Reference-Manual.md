sysdig chisels are Lua scripts that can be invoked from the sysdig command line, hook into the sysdig engine, and can be used to extend sysdig with advanced funcionality.

Chisels talk with sysdig using three separate interfaces: 
* the sysdig library, with functions to interact with the main program
* the evt library, used to extract information from a captured event
* a set of callbacks that get called when something interesting happens, e.g. when an event is received 

This page documents each of these interfaces.

## Mondatory Globals
In order to be recognized as a chisel, a Lua script *must* export the following global variables:
* _description_: a string containing the chisel verbose description
* _short_description_: a string that explains what the chisel does in a few words, and can be used in a list
* _category_: the chisel category, e.g. _IO_, _net_, _security_, etc.
* _args_: a table describing each of the chisel arguments, in the following format:

```lua 
args = 
{
	{
		name = "name1", 
		description = "description1", 
		argtype = "ipv4"
	},
	{
		name = "name2", 
		description = "description2", 
		argtype = "string"
	},
}
```

_Notes_: 
* For each entry in the _args_ table, sysdig will expect an argument to be specified on the command line after the chisel name, and fail if the argument is not there.
* Each argument in _args_ generates a call to the on_set_arg() callback.
* _args_ can be empty if the chisel doesn't require any argument. 

## Callbacks
Callbacks are the way sysdig uses to notify a chisel that something has happened. Most callbacks don't need to be registered. Just include the function in the chisel and, if present, the engine will call it. The only exception is on_interval(), which needs to be registered with sysdig.set_interval_s() or sysdig.set_interval_ns().
Callbacks are optional. If you don't need one of them, just don't include it.


**on_set_arg(name, val)**

This callback is called by the engine for every argument contained in the _args_ global table. _name_ is the name of the argument to set, while _val_ is the value that the user specified on the sysdig command line.

Returning false means that the parameter is not valid and will cause sysdig to quit.

**on_init()**

Called by sysdig *after* _on_set_arg()_ has been called for every chisel argument, but *before* the capture is configured. Usually, this is where the chisel initialization happens.

Returning false means that the chisel initialization failed and will cause sysdig to quit.

**on_capture_start()**

Called by sysdig *after* the capture is configured, *after* _on_set_arg()_ has been called for every chisel argument, but *before* any packet has been captured. It can be used for final chisel initializations that require to query the capture state.

Returning false means that the chisel initialization failed and will cause sysdig to quit.
**on_event()**

Invoked every time one of the captured events passes the filter specified with _set_filter()_, or for each event if _set_filter()_ has not been called. This is the function that usually contains the core chisel logic, and where you want to make sure things are efficient. From this callback you have access to the _evt_ library to extract information from the currently processed event.

If you specified a formatter, returning false in _on_event()_ will make the formatter ignore the event.

**on_capture_end(ts_s, ts_ns, delta)**

Called by the engine at the end of the capture, i.e.
* When CTRL+C is pressed for live captures.
* After the last event has been read from offline captures.  

The function receives the following arguments:  
* ts_s, ts_ns: the second and nanosecond parts of the timestamp of the last event in the capture
* delta: the time between the first and last packet in the capture, in nanoseconds

**on_interval(ts_s, ts_ns, delta)**

Periodic timeout callback. Can be used to do things like reporting information once a second. Use _sysdig.set_interval_s()_ or _sysdig.set_interval_ns()_ to configure it.  
The function receives the following arguments:  
* ts_s, ts_ns: the second and nanosecond parts of the time when this function is called
* delta: the delta time between the call and the previous call to on_interval()

## sysdig library
The functions in this library can be used to get or set global sysdig configuration, like the snaplen or the program capture filter.

**sysdig.get_machine_info()**

Return a table with information about the machine generating the events.
The returned table has this fields:  
* num_cpus: the number of CPU cores on the machine
* memory_size_bytes: the machine RAM size
* max_pid: the maximum pid allowed on the machine
* hostname: the machine hostname

_Note_: this call works with file captures as well, because the machine info is stored in the trace files. In that case, the returned machine info is the one of the machine where the capture happened.

**sysdig.islive()**

Return true if the current capture is live, i.e. if the events are coming from a the system and not from a trace file.

**sysdig.set_filter(filter)**

Configure the sysdig capture engine to apply the given filter before parsing the captured events. This has the same result as setting the filter from the sysdig command line, and will override the command line filter if one is specified.

**sysdig.set_snaplen(snaplen)**

Configure the number of bytes that are captured from buffers of I/O system calls like read(), write(), sendto(), recvfrom(), etc.

## chisel library
The functions in this library are mostly related to setting up the chisel environment and are usually called at initialization time, i.e. inside on_init().


**chisel.exec(chiselname, arg1, arg2, ...)**

This function can be used to stop the execution of the calling chisel and, instead, run a different chisel. The function arguments are the chisel name, followed by the arguments to pass to the new chisel. 

**chisel.request_field(fld_name)**

This function is used to configure the sysdig engine to extract a filter field when an event is captured. _fld_name_ is the name of the sysdig filter field to extract (see the sysdig tutorial or type _sysdig -l_ for a list of available fields).
The function returns a field handle that can be fed to evt.field() to get the field value from the on_event() callback. 

**chisel.set_filter(filter)**

Configure the sysdig engine to apply the given filter before handing the events to this chisel's on_event() callback.

_Notes_
* The filter set with set_filter() is private for this chisel and won't influence other chisels that are run from the same command line.
* You can set only one filter per chisel. Calling set_filter() twice will cause the first filter to be overridden.
* It's very important to be as aggressive as possible with filters. The sysdig engine is heavily optimized, so eliminating as many events as possible before reaching the chisel's on_event() will make the chisel much more efficient.

**chisel.set_event_formatter(format)**

Configure an event formatter. _format_ is a string containing a list of fields to print, with the same syntax that you would use with the -p sysdig command line switch (refer to the sysdig manual for more information).

By default, chisels have no event formatter, and that gives them the freedom to print whatever they want using Lua's print() function. Setting a formatter, on the other hand, lets you delegate event formatting to sysdig, leveraging sysdig's filter fields system. Note that only events for which on_event() returns _true_ are going to be printed.

**chisel.set_interval_s(interval)**

Set a periodic callback for this chisel. If you use this function, the chisel needs to include a function called on_interval(), which the engine will call every _interval_ seconds. This can be used to perform periodic tasks, like printing information to the screen once a second.

**chisel.set_interval_ns(interval)**

Like set_interval_s(), but allows more granular timeouts. 

## evt library
The functions in this library are related to the event that is currently processed and therefore can only be called from the on_event() callback.


**evt.get_cpuid()**

Return number of the CPU where this event was captured.

**evt.field(fld)**

Extract a field's value from an event. _fld_ is a field handle obtained from a previous call to request_field().
The function returns the field value, which can be a number or a string depending on the field. Use _sysdig -lv_ to find out the type of each exported field. 

_Notes_
* If the requested field is not exported by an event (e.g. non I/O events don't export the fd.name field), the return value of field() will be nil.
* One important field you need to be aware of is evt.rawarg. This field can be used to extract arbitrary system call arguments, e.g. evt.rawarg.res, and its type depends on the field you're asking. Use _sysdig -L_ to find out the type of event arguments.

**evt.get_num()**

Return the incremental event number.

**evt.get_ts()**

Return the raw event timestamp, expressed as nanoseconds since epoch. 

**evt.get_type()**

Return the event type as a number. This function can be used for efficient event filtering. For the list of event type numbers, please refer to the ppm_event_type enumeration in driver/ppm_event_events_public.h.
