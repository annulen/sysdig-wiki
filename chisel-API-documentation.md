# sysdig library
This library exports functions that can be used to interface with sysdig from a chisel.
## Functions
**request_field(fld_name)**

**set_filter(filter)**

**set_snaplen(snaplen)**

**set_event_formatter(format)**

**set_interval_ns(interval)**

**set_interval_s(interval)**

# evt library
**field(fld)**

**get_num()**

**get_ts()**

**get_type()**

# callbacks
**on_set_arg(name, val)**

**on_init()**

**on_event()**

**on_capture_end()**
