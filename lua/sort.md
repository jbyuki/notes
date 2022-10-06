Sorting a table.

```lua
a = {4,3,2,2}
table.sort(a)
print(vim.inspect(a))
```
> { 1, 2, 3, 4 }


```lua
a = {"4", "3", "2", "1"}
table.sort(a)
print(vim.inspect(a))
```
> { "1", "2", "3", "4" }


```lua
a = {1,2,3,4}
table.sort(a, function(ai, aj)
  return ai > aj
end)
print(vim.inspect(a))
```
> { 4, 3, 2, 1 }
