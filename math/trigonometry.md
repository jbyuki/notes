## Proving the trigonometric equations


```lua
swan = require"swan"
print(swan.version())
```
```output[1](09/23/22 23:16:03)
0.0.1
```

Find what $sin^3(x)$ is equal to in simpler
terms with swan.lua.

```lua
x = swan.sym "x"
y = swan.sym "y"
cosx = ((swan.e ^ (swan.i * x) + swan.e ^ (-swan.i * x)))/2
sinx = ((swan.e ^ (swan.i * x) - swan.e ^ (-swan.i * x)))/(2*swan.i)
-- exp = (cosx*cosx):simplify()
exp = (sinx*sinx*sinx):simplify():expand()
print(exp)
```
```output[2](09/23/22 23:16:04)
(3(-(e^((-i)x)))(e^(ix))^2 + 3(-(e^((-i)x)))^2(e^(ix)) + (-(e^((-i)x)))^3 + (e^(ix))^3)/(2i^3)
```
