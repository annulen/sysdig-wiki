sysdig chisels are Lua scripts that can be invoked from the sysdig command line, hook into the sysdig engine, and can be used to extend sysdig with advanced funcionality.

Chisels talk with sysdig using of three separate interfaces: 
* the sysdig library, with functions to interact with the main program
* the evt library, used to extract information from a captured event
* a set of callbacks that get called when something interesting happens, e.g. when an event is received 

This page documents each of these interfaces.

## sysdig library
The functions in this library are mostly related to setting up the chisel environment and are usually called at initialization time, i.e. inside on_init().

**request_field(fld_name)**
> sdfsdgf

**set_filter(filter)**

**set_snaplen(snaplen)**

**set_event_formatter(format)**

**set_interval_ns(interval)**

**set_interval_s(interval)**

## evt library
**field(fld)**

**get_num()**

**get_ts()**

**get_type()**

## callbacks
**on_set_arg(name, val)**

**on_init()**

**on_event()**

**on_capture_end()**
