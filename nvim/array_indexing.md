quick test

```lua
local start = vim.fn.reltime()
for i=1,100000 do
	local a = {}
	for i=1,1000 do
		table.insert(a, i)
	end
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[30](4/30/2023 2:10:33 PM)
0.5392994
```

```lua
local start = vim.fn.reltime()
for i=1,100000 do
	local a = {}
	for i=1,1000 do
		a[#a+1] = i
	end
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[31](4/30/2023 2:10:35 PM)
0.5255041
```

