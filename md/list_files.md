## List files

The main motivation to use this as opposed to `vim.fn.glob` for example
which is very convient is _speed_. 

Use the libuv functions to list files.

There are:
* `fs_opendir` : Open a directory
* `fs_readdir` : Iterate through the directory.
* `fs_closedir` : Close directory


```lua
vim.fn.chdir("~/fakeroot/doc/notes")
print(vim.fn.glob("**/*"))
```
```output[2](09/16/22 16:26:55)
md
md\list_files.md
md\threading.md
```

`10` is the number of entries returned each call by readdir().
By default, if no argument is given, it is `1`.

```lua
vim.fn.chdir("~/fakeroot/doc/wiki")
dir = vim.loop.fs_opendir(".", nil, 10)
print(dir)
```
```output[29](09/16/22 16:44:01)
uv_dir_t: 0x01c76ad94e20
```


```lua
local entries = dir:readdir()
print(vim.inspect(entries))
```
```output[13](09/16/22 16:30:45)
{ {
    name = ".git",
    type = "directory"
  }, {
    name = "md",
    type = "directory"
  } }
```

```lua
local entries = dir:readdir()
print(vim.inspect(entries))
```
```output[14](09/16/22 16:32:02)
nil
```

## Recursive 

```lua
function open_dir(path, files)
  local dir = vim.loop.fs_opendir(path, nil, 50)
  if dir then
    while true do
      local entries = dir:readdir()
      if not entries then
        break
      end

      for _, entry in ipairs(entries) do
        if entry["type"] == "directory" then
          open_dir(path .. "/" .. entry["name"])
        else
          table.insert(files, path .. "/" .. entry["name"])
        end
      end
    end
    dir:closedir()
  end
end

local all_files = {}
open_dir(".", all_files)
print(#all_files)
```
```output[34](09/16/22 16:45:03)
317
```

Let's compare this function versus `vim.fn.glob`.


```lua
local start = vim.fn.reltime()
for i=1,10 do
  local files = vim.split(vim.fn.glob("**/*"), "\n")
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[33](09/16/22 16:44:34)
0.3517121
```


```lua
local start = vim.fn.reltime()
for i=1,10 do
  local files = {}
  open_dir(".", files)
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[35](09/16/22 16:45:28)
0.0055646
```

