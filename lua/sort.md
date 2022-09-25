Sorting a table.

```lua
a = {4,3,2,1}
table.sort(a)
print(vim.inspect(a))
```
```output[3](09/25/22 22:08:32)
{ 1, 2, 3, 4 }
```


```lua
a = {"4", "3", "2", "1"}
table.sort(a)
print(vim.inspect(a))
```
```output[5](09/25/22 22:09:06)
{ "1", "2", "3", "4" }
```


```lua
a = {1,2,3,4}
table.sort(a, function(ai, aj)
  return ai > aj
end)
print(vim.inspect(a))
```
```output[6](09/25/22 22:09:47)
{ 4, 3, 2, 1 }
```
