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
```output[39](09/16/22 17:10:16)
uv_dir_t: 0x01c76ff39378
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
```output[38](09/16/22 17:10:04)
{ {
    name = "2021_02_05_16_55_03.txt",
    type = "file"
  }, {
    name = "2021_02_06_21_37_56.txt",
    type = "file"
  }, {
    name = "2021_02_10_10_34_12.txt",
    type = "file"
  }, {
    name = "2021_02_13_16_01_00.txt",
    type = "file"
  }, {
    name = "2021_02_14_18_52_27.txt",
    type = "file"
  }, {
    name = "2021_02_14_19_17_30.txt",
    type = "file"
  }, {
    name = "2021_02_15_16_15_47.txt",
    type = "file"
  }, {
    name = "2021_02_15_22_42_04.txt",
    type = "file"
  }, {
    name = "2021_02_15_23_14_51.txt",
    type = "file"
  }, {
    name = "2021_02_16_13_22_54.txt",
    type = "file"
  } }
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

all_files = {}
open_dir(".", all_files)
print(#all_files)
```
```output[43](09/16/22 17:10:42)
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
```output[37](09/16/22 17:09:54)
0.3803573
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


```lua
print(all_files[1]:match("%.([^.]*)$"))
```
```output[50](09/16/22 17:12:45)
txt
```
