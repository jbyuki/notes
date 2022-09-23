# Testing with threads

Question: Is it possible to spawn multiple threads
in neovim to read files and then concatenate all
the result and present them to the user?

```lua
print(vim.inspect(vim.version()))
```
```output[4](09/16/22 13:59:02)
{
  api_compatible = 0,
  api_level = 10,
  api_prerelease = true,
  major = 0,
  minor = 8,
  patch = 0,
  prerelease = true
}
```

A thread can be spawned in the recent versions of neovim. It is 
runs separately at the os-level. It has its own interpreter state
and event loop.  (see `help lua-loop-threading`)

When new_thread is called, it starts immediatly. Then, there is no
way to interrupt it and resume it. It's only possible to check
if it has terminated.

```lua
local thread = vim.loop.new_thread({}, function() print("hello") end)
print(thread)
```
```output[2](09/16/22 15:07:08)
uv_thread_t: 0x00000214
```

Communication between threads is possible via asyncs. An async object
is essentially a handle to a callback that can be passed around to
other threads. It allows the thread to communicate back to the main
thread with the results for example.

```lua
local async
result = nil
async = vim.loop.new_async(function(arg)
  result = arg
  async:close()
end)

local thread = vim.loop.new_thread({}, function(async) async:send("hello") end, async)
```
```output[17](09/16/22 15:13:04)
```


```lua
print(result)
```
```output[20](09/16/22 15:13:29)
hello
```

## Loading a file

Read a file regurarly. I always forget how this works. 


```lua
vim.fn.chdir("~/fakeroot/doc/notes/md")
print(vim.fn.getcwd())
```
```output[1](09/16/22 16:03:22)
C:\Users\Julien\fakeroot\doc\notes\md
```


```lua
f = io.open("threading.md")
print(f)
```
```output[2](09/16/22 16:03:25)
file (0x01c1e4cf6d20)
```

Using "*a", the file is read entirely at once. It is just a
single string. It can be broken down into a table of lines as well.

```lua
content = f:read("*a")
print(#content)
```
```output[54](09/16/22 15:40:08)
2267
```


```lua
lines = vim.split(content, "\n")
print(#lines)
```
```output[55](09/16/22 15:40:19)
121
```


```lua
print(lines[1])
```
```output[60](09/16/22 15:40:49)
# Testing with threads
```

Now combining everything together the lines are read from the thread
and passed back to the main thread via asyncs.


```lua
local async
result = nil
async = vim.loop.new_async(function(lines)
  result = lines
  async:close()
end)

local thread = vim.loop.new_thread({}, function(async) 
  local f = io.open("threading.md")
  if not f then
    async:send({})
  else
    local content = f:read("*a")
    f:close()
    async:send(vim.split(content, "\n"))
  end
end, async)
```
```output[61](09/16/22 15:44:37)
```


```lua
print(#lines)
```
```output[62](09/16/22 15:44:44)
121
```

This works well as expected. This could now extended to load multiple files in parallel.

## Loading multiple files

This will load the files contained in my RFC folder. It is a list of text files.

```lua
vim.fn.chdir("~/fakeroot/doc/wiki")
files = vim.split(vim.fn.glob("*.txt"), "\n")
print(#files)
```
```output[3](09/16/22 16:03:32)
314
```


```lua
print(files[2])
print(vim.inspect(io.open(files[2])))
```
```output[80](09/16/22 16:02:12)
2021_02_06_21_37_56.txt
nil
```

Let's benchmark the time to load the files sequentially and then load them all in parallel and compare.
This could be interesting.


```lua
local all_files = {}

local start_time = vim.fn.reltime()

for _, fn in ipairs(files) do
  local f = io.open(fn)
  if f then
    local content = f:read("*a")
    f:close()
    local lines = vim.split(content, "\n")
    table.insert(all_files, lines)
  end
end

local end_time = vim.fn.reltime(start)
print(#all_files)
print(("elapsed: %g"):format(vim.fn.reltimefloat(end_time)))
```
```output[7](09/16/22 16:04:38)
314
elapsed: 0
```

