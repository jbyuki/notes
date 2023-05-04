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
```output[1](5/5/2023 12:01:55 AM)
```


```lua
RV = {} -- random variable
id_table = {}
function RV:new_abstract(o)
	local id
	while true do
		id = math.floor(math.random()*1e10)
		if not id_table[id] then
			id_table[id] = true
			break
		end
	end

	o = o or {}
	o.id = id

	local mt = {}
	mt.__index = self
	for k, v in pairs(RV) do
		mt[k] = v
	end

	return setmetatable(o, mt)
end

function RV.resolve(p, saved)
	if type(p) == "number" then
		return p
	elseif type(p) == "table" then
		return p(saved)
	else
		assert(false, ("Unsupported %s!"):format(type(p)))
	end
end

function RV:copy()
	local o = {}
	for k, v in pairs(self) do
		o[k] = v
	end
	return setmetatable(o, getmetatable(self))
end

function RV.copy_one_of(...)
	for _, p in ipairs({...}) do
		if type(p) == "table" then
			return p:copy()
		end
	end
end

function RV.__add(p1, p2)
	local result = RV.copy_one_of(p1, p2)
	function result:sample(saved)
		return RV.resolve(p1, saved) + RV.resolve(p2, saved)
	end
	return result
end

function RV.__sub(p1, p2)
	local result = RV.copy_one_of(p1, p2)
	function result:sample(saved)
		return RV.resolve(p1, saved) - RV.resolve(p2, saved)
	end
	return result
end

function RV.__mul(p1, p2)
	local result = RV.copy_one_of(p1, p2)
	function result:sample(saved)
		return RV.resolve(p1, saved) * RV.resolve(p2, saved)
	end
	return result
end

function RV.__div(p1, p2)
	local result = RV.copy_one_of(p1, p2)
	function result:sample(saved)
		return RV.resolve(p1, saved) / RV.resolve(p2, saved)
	end
	return result
end

function RV.__pow(p1, k)
	local result = p1:copy()
	function result:sample(saved)
		return p1(saved) ^ k
	end
	return result
end

function RV.__unm(p1)
	local result = p1:copy()
	function result:sample(saved)
		return -p1(saved)
	end
	return result
end

function RV:__call(saved)
	return self:sample(saved)
end

function RV:rand(saved)
	local eps
	if saved then
		if not saved[self.id] then
			saved[self.id] = math.random()
		end
		eps = saved[self.id]
	else
		eps = math.random()
	end
	return eps
end
```
```output[2](5/5/2023 12:01:55 AM)
```

## Normal distribution

```lua
Normal = RV:new_abstract()
function Normal:new(mu, sigma)
	return Normal:new_abstract { mu = mu or 0, sigma = sigma or 1 }
end
function Normal:sample(saved)
	local eps = self:rand(saved)
	return self.sigma * InvNormCDF(eps) + self.mu
end
```
```output[3](5/5/2023 12:01:55 AM)
```

```lua
x1 = Normal:new(0, 1)
x2 = Normal:new(0, 1)
y = x1 + x2
print(y())
```
```output[4](5/5/2023 12:01:55 AM)
-0.71388329865408
```

```lua
function plot(p, min_x, max_x, N)
	min_x = min_x or -5
	max_x = max_x or 5
	N = N or 31
	local plt = require"plotly"
	local grid = {}
	local dx = (max_x-min_x)/N
	local M = 1000000
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
```output[5](5/5/2023 12:01:55 AM)
```

```lua
x1 = Normal:new(0, 1)
plot(x1, -3, 3)
```
```output[6](5/5/2023 12:01:55 AM)
Server started on port 8087
```

```lua
print(1/(1*math.sqrt(2*math.pi)))
```
```output[7](5/5/2023 12:01:55 AM)
0.39894228040143
```

```lua
function E(p)
	local sum = 0
	local M = 1000000
	for i=1,M do
		sum = sum + p()
	end
	return sum/M
end
```
```output[8](5/5/2023 12:01:55 AM)
```

```lua
function Var(p)
	return E(p^2) - E(p)^2
end
```
```output[9](5/5/2023 12:01:55 AM)
```


```lua
local x1 = Normal:new(2.5,3)
print(E(x1))
print(Var(x1))
```
```output[10](5/5/2023 12:01:58 AM)
2.5031737120709
8.9568123556338
```

## Chi-Squared distribution

```lua
ChiSquared = RV:new_abstract()
function ChiSquared:new(k)
	assert(k)
	local x = Normal:new(0,1)^2
	for i=2,k do
		local xi = Normal:new(0,1)^2
		x = x + xi
	end
	return ChiSquared:new_abstract { rv = x }
end
function ChiSquared:sample(saved)
	return self.rv(saved)
end
```
```output[11](5/5/2023 12:01:58 AM)
```

```lua
local x1 = Normal:new(0,1)^2
local x2 = Normal:new(0,1)^2
local y = x1 + x2
print(x1())
```
```output[12](5/5/2023 12:01:58 AM)
0.083002706670583
```

```lua
local x = ChiSquared:new(4)
plot(x, 0, 10, 20)
```
```output[13](5/5/2023 12:02:01 AM)
```

## Discrete distributions #1

```lua
Coin = RV:new_abstract()
function Coin:new(k)
	return Coin:new_abstract { }
end
function Coin:sample(saved)
	local eps = self:rand(saved)
	if eps < 0.5 then
		return -1
	else
		return 1
	end
end
```
```output[14](5/5/2023 12:02:01 AM)
```

```lua
function plotdiscrete(p)
	local plt = require"plotly"
	local grid = {}
	local M = 1000000
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
```output[15](5/5/2023 12:02:01 AM)
```

```lua
plotdiscrete(Coin:new())
```
```output[16](5/5/2023 12:02:01 AM)
```

```lua
Bernoulli = RV:new_abstract()
function Bernoulli:new(p)
	assert(p)
	return Bernoulli:new_abstract { p = p }
end
function Bernoulli:sample(saved)
	local eps = self:rand(saved)
	if eps < p then
		return 1 
	else
		return 0
	end
end
```
```output[17](5/5/2023 12:02:02 AM)
```

```lua
function plot2d(p1, p2, min_x, max_x, min_y, max_y, N)
	min_x = min_x or -5
	max_x = max_x or 5
	min_y = min_y or min_x
	max_y = max_y or max_x
	N = N or 15
	local plt = require"plotly"
	local grid = {}
	local dx = (max_x-min_x)/N
	local dy = (max_y-min_y)/N
	local M = 1000000
	for i=1,M do
		local saved = {}
		local xi = p1(saved)
		local yi = p2(saved)

		local nx = math.ceil((xi - min_x)/dx)
		local ny = math.ceil((yi - min_y)/dy)

		grid[nx] = grid[nx] or {}
		grid[nx][ny] = (grid[nx][ny] or 0) + 1
	end

	local x = {}
	local y = {}
	local z = {}

	for i=1,N do
		table.insert(x, (i-0.5)*dx + min_x)
		table.insert(y, (i-0.5)*dy + min_y)
	end

	for nx=1,N do
		local z_row = {}
		for ny=1,N do
			if not grid[nx] then
				table.insert(z_row, 0)
			else
				table.insert(z_row, (grid[nx][ny] or 0)/(dx*dy*M))
			end
		end
		table.insert(z, z_row)
	end

	plt.plot3d(x,y,z)
end
```
```output[35](5/5/2023 12:09:12 AM)
```


```lua
local x = Normal:new(0,1)
local y = Normal:new(0,1)
local z = Coin:new()*3
plot2d(x+y, y, -4, 4, -4, 4, 30)
```
```output[41](5/5/2023 12:10:51 AM)
```

