
Testing the built-in neovim functions for map-like tables.


```lua
a = {}
a["foo"] = true

b = {}
b["bar"] = true

print(vim.inspect(a))
print(vim.inspect(b))
```
```output[2](10/18/22 20:33:10)
{
  foo = true
}
{
  bar = true
}
```

vim.tbl_extend allows to merge map-like tables together.


```lua
c = vim.tbl_extend("force", a,b)
print(vim.inspect(c))
```
```output[4](10/18/22 20:34:25)
{
  bar = true,
  foo = true
}
```
