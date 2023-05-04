See [probacontinousv1](probacontinuous.md).

```lua
function InvNormCDF(p)
	local x
	local a = {}
	a[1] = -3.969683028665376e+01
	a[2] =  2.209460984245205e+02
	a[3] = -2.759285104469687e+02
	a[4] =  1.383577518672690e+02
	a[5] = -3.066479806614716e+01
	a[6] =  2.506628277459239e+00

	local b = {}
	b[1] = -5.447609879822406e+01
	b[2] =  1.615858368580409e+02
	b[3] = -1.556989798598866e+02
	b[4] =  6.680131188771972e+01
	b[5] = -1.328068155288572e+01

	local c = {}
	c[1] = -7.784894002430293e-03
	c[2] = -3.223964580411365e-01
	c[3] = -2.400758277161838e+00
	c[4] = -2.549732539343734e+00
	c[5] =  4.374664141464968e+00
	c[6] =  2.938163982698783e+00

	local d = {}
	d[1] =  7.784695709041462e-03
	d[2] =  3.224671290700398e-01
	d[3] =  2.445134137142996e+00
	d[4] =  3.754408661907416e+00

	-- Define break-points.

	p_low  = 0.02425
	p_high = 1 - p_low

	-- Rational approximation for lower region.

	if 0 < p and p < p_low then
		q = math.sqrt(-2*math.log(p))
		x = (((((c[1]*q+c[2])*q+c[3])*q+c[4])*q+c[5])*q+c[6]) /
					((((d[1]*q+d[2])*q+d[3])*q+d[4])*q+1)
	end

	-- Rational approximation for central region.

	if p_low <= p and p <= p_high then
		q = p - 0.5
		r = q*q
		x = (((((a[1]*r+a[2])*r+a[3])*r+a[4])*r+a[5])*r+a[6])*q /
				 (((((b[1]*r+b[2])*r+b[3])*r+b[4])*r+b[5])*r+1)
	end

	-- Rational approximation for upper region.

	if p_high < p and p < 1 then
		q = math.sqrt(-2*math.log(1-p))
		x = -(((((c[1]*q+c[2])*q+c[3])*q+c[4])*q+c[5])*q+c[6]) /
					 ((((d[1]*q+d[2])*q+d[3])*q+d[4])*q+1)
	end

	return x
end
```
```output[5](5/4/2023 10:02:44 AM)
```

```lua
function Normal(mu, sigma)
	return function(w)
		return InvNormCDF(w or math.random())
	end
end
```
```output[31](5/4/2023 10:11:46 AM)
```


```lua
x1 = Normal(0, 1)
```
```output[7](5/4/2023 10:02:50 AM)
```


```lua
function plot(p, min_x, max_x, N)
	local plt = require"plotly"
	local grid = {}
	local dx = (max_x-min_x)/N
	local M = 500000
	for i=1,M do
		local pi = p()
		local n = math.ceil((pi - min_x)/dx)
		grid[n] = (grid[n] or 0) + 1
	end

	local x = {}
	local y = {}
	for i=1,N do
		table.insert(x, (i-0.5)*dx + min_x)
		table.insert(y, (grid[i] or 0)/(dx*M))
	end
	plt.scatter(x,y)
end
```
```output[96](5/4/2023 2:15:38 PM)
```

```lua
x1 = Normal(0,1)
plot(x1, -3, 3, 50)
```
```output[33](5/4/2023 10:11:51 AM)
```

```lua
function add(x1, x2)
	return function()
		return x1() + x2()
	end
end
```
```output[39](5/4/2023 10:13:38 AM)
```

```lua
x1 = Normal(0,1)
x2 = Normal(0,1)
y = add(x1, x2)
plot(y, -5, 5, 50)
```
```output[41](5/4/2023 10:13:57 AM)
```


```lua
function pow(x, k)
	return function()
		return x()^k
	end
end
```
```output[42](5/4/2023 10:14:48 AM)
```

```lua
x1 = add(pow(Normal(0,1), 2), pow(Normal(0,1), 2))
plot(x1, 0, 3, 50)
```
```output[52](5/4/2023 11:20:40 AM)
```

```lua
function ChiSquared(k)
	y = pow(Normal(0,1), 2)
	for i=2,k do
		y = add(y, pow(Normal(0,1), 2))
	end
	return y
end
```
```output[53](5/4/2023 11:21:51 AM)
```

```lua
plot(ChiSquared(8), 0, 20, 30)
```
```output[61](5/4/2023 11:26:58 AM)
```


```lua
function E(p, min_x, max_x)
	local sum = 0
	local M = 500000
	for i=1,M do
		sum = sum + p()
	end
	return sum/M
end
```
```output[62](5/4/2023 11:28:26 AM)
```

```lua
print(E(ChiSquared(3), 0, 30))
```
```output[64](5/4/2023 11:29:34 AM)
3.0005939935558
```

```lua
function Var1(p, min_x, max_x)
	return E(pow(p,2), min_x^2, max_x^2) - E(p, min_x, max_x)^2
end
```
```output[67](5/4/2023 1:51:45 PM)
```

```lua
print(Var1(ChiSquared(3), 0, 30))
```
```output[69](5/4/2023 1:52:00 PM)
5.9770650616874
```

```lua
function Var2(p, min_x, max_x)
	local mu = E(p, min_x, max_x)
	local xmu = function()
		return (p() - mu)^2
	end
	local L = (max_x - min_x)/2
	return E(xmu, -L^2, L^2)
end
```
```output[70](5/4/2023 1:53:50 PM)
```

```lua
local rms_1 = 0
local rms_2 = 0
local time_1 = 0
local time_2 = 0
for i=1,10 do
	local start = vim.fn.reltime()
	rms_1 = rms_1 + (Var1(ChiSquared(3), 0, 30) - 6)^2
	time_1 = time_1 + vim.fn.reltimefloat(vim.fn.reltime(start))
	local start = vim.fn.reltime()
	rms_2 = rms_1 + (Var2(ChiSquared(3), 0, 30) - 6)^2
	time_2 = time_2 + vim.fn.reltimefloat(vim.fn.reltime(start))
end
print(math.sqrt(rms_1/10), " elapsed ", time_1)
print(math.sqrt(rms_2/10), " elapsed ", time_2)
```
```output[79](5/4/2023 2:00:53 PM)
0.032686935911398  elapsed  26.6705192
0.033009648235057  elapsed  26.5092492
```

Choose `Var1` because it's simpler with similar results

```lua
Var = Var1
```
```output[81](5/4/2023 2:03:09 PM)
```

```lua
function Coin()
	return function(w)
		if (w or math.random()) < 0.5 then
			return -1
		else
			return 1
		end
	end
end
```
```output[84](5/4/2023 2:10:05 PM)
```

```lua
function Bernoulli(p)
	return function(w)
		if (w or math.random()) < p then
			return 1
		else
			return 0
		end
	end
end
```
```output[85](5/4/2023 2:10:53 PM)
```

```lua
plot(Bernoulli(0.5), -1, 3, 5)
```
```output[97](5/4/2023 2:15:40 PM)
```

```lua
function plotdiscrete(p)
	local plt = require"plotly"
	local grid = {}
	local M = 500000
	for i=1,M do
		local pi = p()
		grid[pi] = (grid[pi] or 0) + 1
	end

	local x = {}
	local y = {}
	for xi, yi in pairs(grid) do
		table.insert(x, xi)
		table.insert(y, yi/M)
	end
	plt.scatter(x,y)
end
```
```output[106](5/4/2023 2:19:03 PM)
```

```lua
function mul(p1, p2)
	return function()
		return p1() * p2()
	end
end
```
```output[113](5/4/2023 2:29:24 PM)
```


```lua
local X = Normal(0,1)
local Y = Normal(0,1)
local Z = Coin()
plot(add(X, mul(Z,Y)), -5, 5, 50)
```
```output[115](5/4/2023 2:30:16 PM)
```

```lua
local X = Normal(0,1)
local Y = Normal(0,1)
local Z = Coin()
print(Var(add(X, mul(Z,Y)), -10, 10))
```
```output[117](5/4/2023 2:31:23 PM)
1.9972335135316
```

