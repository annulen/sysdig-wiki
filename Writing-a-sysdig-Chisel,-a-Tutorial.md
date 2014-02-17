This page will guide you through creating a simple but fully featured sysdig chisels.

sysdig chisels are written in Lua, a powerful and fast embedded scripting language. If you are not familiar with it, I suggest you take a look to THE book,

http://www.lua.org/pil/

and to the reference manual,

http://www.lua.org/manual/5.2/

## Chisel Backbone

```lua
-- Chisel description
description = "shows the network payloads exchanged with an IP endpoint";
short_description = "connection spy";
category = "net";

-- Chisel argument list
args = {}

-- Event parsing callback
function on_event()
	print("event!")
	return true
end
```