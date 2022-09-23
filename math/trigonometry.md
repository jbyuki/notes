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

Go into the complex domain with $sin(x)=\frac{e^{ix} - e^{-ix}}{2i}$

```lua
x = swan.sym "x"
y = swan.sym "y"
cosx = ((swan.e ^ (swan.i * x) + swan.e ^ (-swan.i * x)))/2
sinx = ((swan.e ^ (swan.i * x) - swan.e ^ (-swan.i * x)))/(2*swan.i)
-- exp = (cosx*cosx):simplify()
exp = (sinx*sinx):simplify():expand()
num = exp.o.lhs
print(num:expand())
```
```output[9](09/23/22 23:56:50)
2(-(e^((-i)x)))(e^(ix)) + (-(e^((-i)x)))^2 + (e^(ix))^2
```
