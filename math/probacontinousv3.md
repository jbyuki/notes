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
```output[98](5/4/2023 10:21:58 PM)
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

function RV.__add(p1, p2)
	return function(saved)
		saved = saved or {}
		return RV.resolve(p1, saved) + RV.resolve(p2, saved)
	end
end

function RV.__sub(p1, p2)
	return function(saved)
		saved = saved or {}
		return RV.resolve(p1, saved) - RV.resolve(p2, saved)
	end
end

function RV.__mul(p1, p2)
	return function(saved)
		saved = saved or {}
		return RV.resolve(p1, saved) * RV.resolve(p2, saved)
	end
end

function RV.__div(p1, p2)
	return function(saved)
		saved = saved or {}
		return RV.resolve(p1, saved) / RV.resolve(p2, saved)
	end
end

function RV:__pow(k)
	return function(saved)
		saved = saved or {}
		return RV.resolve(self, saved) ^ k
	end
end

function RV:__unm()
	return function(saved)
		saved = saved or {}
		return -RV.resolve(self, saved)
	end
end

function RV:__call(saved)
	local eps
	if saved then
		if not saved[self.id] then
			saved[self.id] = math.random()
		end
		eps = saved[self.id]
	else
		eps = math.random()
	end
	return self:sample(eps)
end
```
```output[99](5/4/2023 10:21:58 PM)
```


```lua
Normal = RV:new_abstract()
function Normal:new(mu, sigma)
	return Normal:new_abstract { mu = mu, sigma = sigma }
end
function Normal:sample(eps)
	return self.sigma * InvNormCDF(eps) + self.mu
end
```
```output[100](5/4/2023 10:21:58 PM)
```

```lua
x1 = Normal:new(0, 1)
x2 = Normal:new(0, 1)
y = x1 + x2
print(y())
```
```output[101](5/4/2023 10:21:58 PM)
1.5496994663368
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
```output[102](5/4/2023 10:21:58 PM)
```

```lua
x1 = Normal:new(0, 1)
plot(x1, -3, 3)
```
```output[103](5/4/2023 10:21:59 PM)
```

```lua
print(1/(1*math.sqrt(2*math.pi)))
```
```output[104](5/4/2023 10:21:59 PM)
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
```output[115](5/4/2023 10:23:39 PM)
```

```lua
function Var(p)
	return E(p^2) - E(p)^2
end
```
```output[114](5/4/2023 10:23:38 PM)
```


```lua
local x1 = Normal:new(2.5,3)
print(E(x1))
print(Var(x1))
```
```output[118](5/4/2023 10:23:57 PM)
2.4994513139749
9.002169607584
```

