Start an embedded neovim instance.


```lua
local kernel = vim.fn.jobstart({vim.v.progpath, '--embed', '--headless'}, {rpc = true})
local ret = vim.rpcrequest(kernel, "nvim_exec_lua", [[return vim.fn.getcwd()]], {})
print(ret)
vim.fn.jobstop(kernel)
```
```output[21](8/26/2023 9:52:17 PM)
C:\Users\jybur\fakeroot\code\nvimplugins\pack\custom\start\one-small-step-for-vimkind\src
```


```lua
print(nvim)
```
```output[13](8/26/2023 9:50:41 PM)
5
```

```lua
local pwd_result = vim.fn.rpcrequest(nvim, 'nvim_exec_lua', [[print("hello")]], {})
print(pwd_result)
```
```output[14](8/26/2023 9:50:43 PM)
Vim:Error invoking 'nvim_exec_lua' on channel 5:
Invalid channel: 5
```

