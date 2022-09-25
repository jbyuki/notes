## Start a server

The port can be set to 0 to let libuv choose a free port.


```lua
server = vim.loop.new_tcp()
server:bind("127.0.0.1", port or 0)
```
```output[1](09/25/22 22:54:52)
```

To get the port, call `getsockname`.


```lua
print(vim.inspect(server:getsockname()))
```
```output[3](09/25/22 22:55:37)
{
  family = "inet",
  ip = "127.0.0.1",
  port = 56288
}
```
