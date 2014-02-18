This page will guide you through creating a simple but fully featured sysdig chisel. We're going to create a chisel that takes a system call name as a parameter and prints how many times that system call has been called.

sysdig chisels are written in Lua, a powerful and fast embedded scripting language. If you are not familiar with it, I suggest you take a look at THE book,  
http://www.lua.org/pil/  
and at the reference manual,  
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
Our chisel needs an argument: the name of the system call that the user wants to count. We need sysdig to know about it, and then be ready to receive the argument value once sysdig has parsed it. In order to do that, we fill the _arg_ table with our argument information, and we add the on_set_arg() function. on_set_arg() gets called with the name and value of each of the arguments that are listed in _args_.

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
	if evt.field(ftype) == syscallname and evt.field(fdir) == ">" then
		count = count + 1
		print(count)
	end
	
	return true
end
```
Usually, after receiving all the parameters, a chisel wants to initialize things before the capture starts. Doing that is just a matter of adding the on_init() function, which is called by the engine before receiving the first event.

In our case, since we want to count how many times a system call has been called, we need to know what system call type each of the captured events corresponds to. We also need the event direction, because sysdig generates two events (an enter one and an exit one) for every system call, and we don't want to double count. We instruct the engine to extract this data for us by requesting the _evt.type_ and _evt.dir_ fields to the engine from on_init(), with sysdig.request_field().

sysdig.request_field() accepts any sysdig filter/display field. If you're curious about which fields you can use, take a look at the sysdig tutorial, or type
> sysdig -l

or
> sysdig -L  

Now take a look at on_event(). Once the values have been requested with request_field(), we can extract their value for each incoming event.

The output of our script now makes much more sense, but it's still very verbose. It would be nice to print just a summary at the end of the capture.

## Getting Notified When the Capture Ends
Adding on_capture_end() to our script allows us to get notified when the capture ends, i.e.:
* when the last packet is reached in an offline capture
* when CTRL+C is called on a live capture

This is exactly what we need to make our script less verbose, so let's add it to the code:

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
	if evt.field(ftype) == syscallname and evt.field(fdir) == ">" then
		count = count + 1
	end
	
	return true
end

-- End of capture callback
function on_capture_end()
	print(syscallname .. " has been called " .. count .. " times")
	return true
end
```

Let's take a look at the result

> sysdig -c countsc open
> open has been called 7484 times

Beautiful! We're done, right?
Well, not yet. We can still improve our script, and use it as an opportunity to learn about filtering. 

## Filtering
Our current code receives every single event in on_event(), just to count a little subset of them. That's not ideal because:
* it makes our code more complex than it should be
* it's not as efficient as it could be. Just in time compiled Lua is very fast, but if possible we should delegate stuff to the sysdig engine, which is written in C++ and heavily optimized.
Fortunately, chisels can leverage the sysdig filtering engine. Look at this code:

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
	
	-- set the filter
	sysdig.set_filter("evt.type=" .. syscallname .. " and evt.dir = >")
	
	return true
end

count = 0

-- Event parsing callback
function on_event()
	count = count + 1
	return true
end

-- End of capture callback
function on_capture_end()
	print(syscallname .. " has been called " .. count .. " times")
	return true
end
```

We use sysdig.set_filter() in on_init() to set a sysdig filter that keeps only open enter events. At that point on_event is just a trivial counter increment. Again, I recommend that you read the sysdig tutorial if you want to learn more about filtering.

That's it! I hope you enjoyed this tutorial. For more information, consult the [chisel API manual](sysdig Chisel API Reference Manual) and use the [existing chisels](https://github.com/draios/sysdig/tree/master/userspace/sysdig/chisels) as a reference. If you build something cool, don't forget to submit it to the community, through our [github repository](https://github.com/draios/sysdig) or the mailing list.

Happy chiseling! 