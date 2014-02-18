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
Our chisel needs an argument: the name of the system call that the user wants to count. We need sysdig know about it, and then be ready to receive the argument value once sysdig has parsed it. In oder to do that, we fill the _arg_ table with our argument information, and we add the _on_set_arg()_ function. _on_set_arg()_ gets called with the name and value of each of the arguments that are listed in _args_.

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
We can run the new code 
> sysdig -c countsc open

And the output will be an equally unexciting

> open
> open  
> open  
> open  
## Chisel initialization and Filtering

## Printing the results
