Effect of the scale matrix


```lua
swan = require"swan"

e1x = swan.sym "e1x"
e1y = swan.sym "e1y"
e1z = swan.sym "e1z"

e2x = swan.sym "e2x"
e2y = swan.sym "e2y"
e2z = swan.sym "e2z"

e3x = swan.sym "e3x"
e3y = swan.sym "e3y"
e3z = swan.sym "e3z"

tx = swan.sym "tx"
ty = swan.sym "ty"
tz = swan.sym "tz"
```
```output[9](09/30/22 15:28:22)
```

```lua
A = swan.mat {
  {e1x, e2x, e3x, tx},
  {e1y, e2y, e3y, ty},
  {e1z, e2z, e3z, tz},
  {0, 0, 0, 1}
}
print(A)
```
```output[10](09/30/22 15:28:25)
[
  e1x, e2x, e3x, tx
  e1y, e2y, e3y, ty
  e1z, e2z, e3z, tz
  0, 0, 0, 1
]
```


```lua
sx = swan.sym "sx"
sy = swan.sym "sy"
sz = swan.sym "sz"
S = swan.mat {
  {sx, 0, 0, 0},
  {0, sy, 0, 0},
  {0, 0, sz, 0},
  {0, 0, 0, 1}
}
print(S)
```
```output[13](09/30/22 15:29:04)
[
  sx, 0, 0, 0
  0, sy, 0, 0
  0, 0, sz, 0
  0, 0, 0, 1
]
```


```lua
print((A*S):simplify():normalize())
```
```output[14](09/30/22 15:29:07)
[
  e1xsx, e2xsy, e3xsz, tx
  e1ysx, e2ysy, e3ysz, ty
  e1zsx, e2zsy, e3zsz, tz
  0, 0, 0, 1
]
```
