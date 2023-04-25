
```lua
local f  = function(a,b)
	return a+b
end
print(f(1,2))

print(vim.inspect(debug.getinfo(vim.loop.fs_open)))

```
```output[7](4/19/2023 1:58:10 AM)
3
{
  currentline = -1,
  func = <function 1>,
  isvararg = true,
  lastlinedefined = -1,
  linedefined = -1,
  namewhat = "",
  nparams = 0,
  nups = 0,
  short_src = "[C]",
  source = "=[C]",
  what = "C"
}
```
