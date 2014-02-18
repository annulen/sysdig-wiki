This page will guide you through creating a simple but fully featured sysdig chisels. We're going to create a chisel that takes a system call name as parameter and prints how many times that system call has been called.

sysdig chisels are written in Lua, a powerful and fast embedded scripting language. If you are not familiar with it, I suggest you take a look to THE book,  
http://www.lua.org/pil/  
and to the reference manual,  
http://www.lua.org/manual/5.2/
## The Basics
Let's start with the simplest possible running chisel.

```lua
-- Chisel description
description = "counts how many times the specified system call has been called";
short_description = "syscall count";
category = "misc";

-- Chisel argument list
args = {}

-- Event parsing callback
function on_event()
	print("event!")
	return true
end
```

In order to be recognized by sysdig as a chisel, a Lua script **must** define the following global variables:
* description
* short description
* category 
* list of arguments that, as you can see from this example, can be empty

In addition to that, we include the on_event(), so that we can see something happening when the chisel executes.
So let's run the beast! First, save the code as countsc.lua in the directory where the sysdig executable is, and then type

> sysdig -c countsc

The output is going to be very unexciting and will look like this

> event!  
> event!  
> event!  
> event!  

## Handling Arguments
Our chisel needs an argument: the name of the system call that the user wants to count. We need sysdig know about it, and then be ready to receive the argument value once sysdig has parsed it. In oder to do that, we fill the _arg_ table with our argument information, and we add the on_set_arg() function. on_set_arg() gets called with the name and value of each of the arguments that are listed in _args_.

```lua
-- Chisel description
description = "counts how many times the specified system call has been called";
short_description = "syscall count";
category = "misc";

-- Chisel argument list
args = 
{
	{
		name = "syscall_name", 
		description = "the name of the system call to count", 
		argtype = "string"
	},
}

-- Argument notification callback
function on_set_arg(name, val)
	syscallname = val
	return true
end

-- Event parsing callback
function on_event()
	print(syscallname)
	return true
end
```
We can run the new code with this command line
> sysdig -c countsc open

And the output will be an equally unexciting

> open  
> open  
> open  
> open

## Chisel Initialization and Extracting Fields
Let's complicate our code a bit.

```lua
-- Chisel description
description = "counts how many times the specified system call has been called";
short_description = "syscall count";
category = "misc";

-- Chisel argument list
args = 
{
	{
		name = "syscall_name", 
		description = "the name of the system call to count", 
		argtype = "string"
	},
}

-- Argument notification callback
function on_set_arg(name, val)
	syscallname = val
	return true
end

-- Initialization callback
function on_init()
	-- Request the fileds that we need
	ftype = sysdig.request_field("evt.type")
	fdir = sysdig.request_field("evt.dir")	
	return true
end

count = 0

-- Event parsing callback
function on_event()
	if evt.field(ftype) == "open" and evt.field(fdir) == ">" then
		count = count + 1
		print(count)
	end
	
	return true
end
```
Usually, after receiving all the parameters, a chisel wants to initialize things before the capture starts. Doing that is just a matter of adding the on_init() function, which is called by the engine before receiveng the first event.

In our case, since we want to count how many times a system call has been called, we need to know what system call type each of the captured events corresponds to. We also need the event direction, because sysdig generates two event (an enter one and an exit one) for every system call, and we don't want to double count. We instruct the engine to extract this data for us by requesting the _evt.type_ and _evt.dir_ fields to the engine from on_init(), with sysdig.request_field().

sysdig.request_field() accepts any sysdig filter/display field. If you're curious about which fields you can use, take a look at the sysdig tutorial, or type
> sysdig -l

or
> sysdig -L  

At that point, we can extract the valu


## Printing the Summary at the End of the Capture
 