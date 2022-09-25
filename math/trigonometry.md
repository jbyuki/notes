## Proving the trigonometric equations


```lua
swan = require"swan"
print(swan.version())
```
```output[1](09/25/22 20:31:00)
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
print(num:expand():simplify())
```
```output[3](09/25/22 20:32:02)
2-1(e^(-1ix + ix)) + -1(e^(-1ix))^2 + e^(2ix)
```
