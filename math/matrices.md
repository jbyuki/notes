Effect of the scale matrix


```lua
swan = require"swan"
swan.clear_syms()

c2w_i_R = swan.sym "c2w_i_R"
c2w_i_t = swan.sym "c2w_i_t"

c2w_j_R = swan.sym "c2w_j_R"
c2w_j_t = swan.sym "c2w_j_t"

g2c_R = swan.sym "g2c_R"
g2c_t = swan.sym "g2c_t"

b2g_i_R = swan.sym "b2g_i_R"
b2g_i_t = swan.sym "b2g_i_t"

b2g_j_R = swan.sym "b2g_j_R"
b2g_j_t = swan.sym "b2g_j_t"

S = swan.sym "S"

c2w_i = swan.mat {
  { c2w_i_R, c2w_i_t },
  { 0, 1 },
}

c2w_j = swan.mat {
  { c2w_j_R, c2w_j_t },
  { 0, 1 },
}

g2c = swan.mat {
  { S*g2c_R, g2c_t },
  { 0, 1 },
}

b2g_i = swan.mat {
  { b2g_i_R , b2g_i_t },
  { 0, 1 },
}

b2g_j = swan.mat {
  { b2g_j_R , b2g_j_t },
  { 0, 1 },
}

```
```output[24](10/12/22 14:06:18)
```

```lua
r1 = (c2w_i * g2c ):simplify()
print(r1)
r2 = (r1 * b2g_i):simplify()
print(r2)
```
```output[27](10/12/22 14:07:00)
[
  Sc2w_i_Rg2c_R, c2w_i_Rg2c_t + c2w_i_t
  0, 1
]
[
  Sb2g_i_Rc2w_i_Rg2c_R, Sb2g_i_tc2w_i_Rg2c_R + c2w_i_Rg2c_t + c2w_i_t
  0, 1
]
```


```lua
print((S*A):simplify():normalize())
```
```output[14](10/12/22 13:36:47)
[
  e1xsx, e2xsx, e3xsx, sxtx
  e1ysy, e2ysy, e3ysy, syty
  e1zsz, e2zsz, e3zsz, sztz
  0, 0, 0, 1
]
```
