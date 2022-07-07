# Lua Sandbox running Lua Version 5.1 | Re-created
* This sandbox is made for multiple uses, main uses include: Security, Obfuscating, Deobfuscating, Debugging, General Usage
* Created by Cypher#6678

## Security Cracking Library
* This library is consistent with its name and comes equipped with some very powerful functions we will get into soon, for now keep reading and continue on
### hookfunction
* HookFunction is a function in lua thats entire purpose is to hook functions, it can be used to break security and trick functions into thinking there calling the right function when there not
```lua
function Print(...)
     return print(...)
end
local old
old = hookfunction(Print,(function(...)
     return warn(...)
end))

--// Instead of calling "print" it calls "warn"
Print("lol")

--// Metamethod hooks example
local oldm = nil
oldm = hookfunction(getrawmetatable(_G).__call,(function(...)
     return {"trolled"} --// Returns this instead of _G
end))

--// lol
hookfunction(hookfunction,hookfunction)
```
### hookmetamethod
* This function is very similar to hookfunction except it only hooks metamethods
```lua
local old
old = hookmetamethod(_G,"__call",(function(...)
     return {"no","thanks"} --// Does the exact same thing that hookfunction did above
end))

--// If you used the old version of this sandbox then you know this used to not work, it does now
local stringhook = nil
stringhook = hookmetamethod(_G,"__metatable,"lol")
```
### getrawmetatable
* Returns the raw metatable of a table (not readonly)
```lua
local t = {"joeware"}
local m = getrawmetatable(t)
m.__metatable = "changed"
m.__call = (function(...)
     return "changed (essentially the same as the 2 examples above)"
end)
```
## Deobfuscation Library
* With this library you can crack and/or reverse lua code that is encrypted/obfuscated, this is useful is decoding malicious lua code to protect your users from a potential RCE and/or getting logged via an RCE or some function you added
### deobfuscate.sandboxFunction
* This function is a very useful function in doing analysis's of potentially malicious functions or whatever function you are trying to analyze. This function takes 2 args: `deobfuscate.sandboxFunction( function, arguments... ), Return Value: Debug Table`
```lua
function i_am_malicious()
   print("you have been hacked! haha!")
end
local dumped = deobfuscate.sandboxFunction(i_am_malicious,nil)
--// the function will log all of the functions behaviour and return its return value to the "dumped" table and also return every single call log of every function in the "dumped" table
```
### deobfuscate.unpack
* This function is a *first layer* table unpacker, meaning it only unpacks the table you specify, not any buried tables, for that you would need the **disassembleTable** function which we will talk about later, the unpacker takes 2 args: `deobfuscate.unpack( table, type: string, argument options: ["string" or "table"]), Return Value: String OR Table`
```lua
local shitter = {"boooboo","fart"}
print(deobfuscate.unpack(shitter,"string"))
--// This will print the arguments of "shitter" as a formatted string
```
### deobfuscate.reverseTable
* This function is used for reversing table arguments, when you call it on a table, all of the tables arguments will be flipped, Example: (original)index = value, (after) value = index, pretty cool, this function takes 1 argument: `deobfuscate.reverseTable( table ), Return Value: nil`
```lua
local dang_wrong_way = {
     value = "index" --// We will invert this to ["index"] = "value"
}
deobfuscate.reverseTable(dang_wrong_way)
print(dang_wrong_way.index) --// Successfully prints
```
### deobfuscate.dumpTable
* Essentially this function is similar to the unpacker except its a faster way to actually unpack a table, its essentially just a for loop + give it a table and it'll print everything in the table for you, useful for saving time, this function takes 2 args: `deobfuscate.dumpTable( table, bool )`
```lua
local t = {"joe","ware"}
deobfuscate.dumpTable(t)
--// It will print everything in the current table passed (except for other tables insides (that requires the disassembler))
```
### deobfuscate.disassembleTable
* This function is a bit of an interesting one, its one of the key recources to deobfuscating/dumping as it strips any table down to the last table inside it, its useful for dumping function return values/**deobfuscate.sandboxFunction** output, i will show a quite a few examples as to how it can be used, this function takes 1 argument: `deobfuscate.disassembleTable( table: [any] ), Return Value: Table`
```lua
--// Usage with deobfuscate.sandboxFunction & deobfuscate.dumpTable
deobfuscate.dumpTable(deobfuscate.disassembleTable(deobfuscate.sandboxFunction(function(...)
	local packed = {}
	local function pack(arg)
		packed[math.random(1,10000)] = {
			[math.random(1,10000)] = {
				[math.random(1,10000)] = math.random(1,10000),
				[math.random(1,10000)] = math.random(1,10000),
				[math.random(1,10000)] = math.random(1,10000)
			},
			[math.random(1,10000)] = {
				[math.random(1,10000)] = {
					[math.random(1,10000)] = math.random(1,10000),
					[math.random(1,10000)] = math.random(1,10000),
					[math.random(1,10000)] = math.random(1,10000)
				},
				[math.random(1,10000)] = {
					[math.random(1,10000)] = {
						[math.random(1,10000)] = math.random(1,10000),
						[math.random(1,10000)] = math.random(1,10000),
						[math.random(1,10000)] = math.random(1,10000)
					},
					[math.random(1,10000)] = math.random(1,10000),
					[math.random(1,10000)] = arg,
					[math.random(1,10000)] = math.random(1,10000)
				}
			}
		}
	end
	local protected = {...}
	for i,v in pairs(protected)do
		pack(v)
	end
	return packed
end,"joe")))
--// What to expect is ALOT of dumping output as all 3 of these functions working togther, dumps the function, dumps the table then outputs the table
--// Try it
```
## Obfuscation Library
* This library focuses and the exact opposite of the library above, it is made to secure/hide code inside of code, its still in beta and isnt finished yet.
### obfuscate.multibyte
* This function is used to actually encode strings in the lua byte encoding to Kinda secure your code, its not great but its a start for your obfuscator, this function takes 2 args: `obfuscation.multibyte( string, table: { Type = ["string" or "table"], Withslashes = [true or false], return value: String`
```lua
local encode = "hello"
encode = obfuscate.multibyte(encode,{
     Type = "string",
     Withslashes = true
})
encode = encode:sub(3,#encode)
--// The string is now encoded (try it)
```
### obfuscate.unconcat
* This function basically takes a string and breaks it up into a table (unconcat), This function takes 1 arg: `obfuscate.unconcat( table ), return value: Table`
```lua
local lol = "complex"
lol = obfuscate.unconcat(lol)
--// The string is now a table
```
## Windows Library
* Contains elevated win functions for higher level use
### windows.write
* Outputs the arguments you pass to it, `windows.write( string )`
```lua
windows.write("joe\n")
--// Prints joe
```
### windows.closeprompt
* Closes the current window (cmd)
```lua
windows.closeprompt()
--// Closes the prompt
```
### windows.messagebox
* Opens a popup message from windows, windows.messagebox( Title, Message, Int )
```lua
windows.messagebox("joe","lol bruh",16)
--// Opens an error message displaying the text
```
### windows.echo [C]
* Alternative function that directly calls to C instead to print, useful to avoid c stack overflow's when using **hookfunction**
```lua
windows.echo("from C")
--// Prints "from C"
```
### windows.clear
* Clears the cmd of all text
```lua
windows.clear()
--// Clears the prompt
```
## Global Environment
* All of the basic functions that arent contained in a library
### typeof
* Returns the type of the specified value, `typeof(value)`
```lua
print(typeof(1))
--// Prints "number" as its a number
```
### getgenv
* Returns the G env (secure env)
```lua
print(getgenv())
--// Prints a table
```
### setgenv
* sets a value at an index, `setgenv( index, value )`
```lua
setgenv("ni",420)
--// Sets a value named "ni" to 420
```
### callgenv
* Calls a function and filters it through a new closure and C
```lua
callgenv(print,"joe")
--// Prints joe
```
### getglobals
* Returns the _G
```lua
getglobals().print("joe")
--// Prints
```
### setglobal
* Set a global value, `setglobal( index, value )
```lua
setglobal(print.warn)
--// Sets _G.print to warn
```
