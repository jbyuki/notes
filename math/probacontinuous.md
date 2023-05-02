Numerically store continous random distribution and do computations with them.

The continous distribution is stored with a particule-like storing, first
sample particles according to the distribution PDF, then do all
computations similar to discrete distribution where each particule
each particule has a uniform PMF. Then for plotting and evaluating,
grid the space and compute the density for each cell.

Use the inversion method for sampling from a PDF. Let's start with the classic Gaussian.
This is already non-trivial.

[...]

After some searching, it seems to offically called the [probit function](https://en.wikipedia.org/wiki/Probit),
although it seems more commonly just called the inverse normal CDF.
And here are the [implementation details](https://web.archive.org/web/20151030215612/http://home.online.no/~pjacklam/notes/invnorm/).

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
```output[7](5/3/2023 12:29:26 AM)
```


```lua
plt = require"plotly"
x = {}
y = {}
for i=0,1,0.01 do
	table.insert(x, i)
	table.insert(y, InvNormCDF(i))
end
plt.scatter(x,y)
```
```output[11](5/3/2023 12:30:16 AM)
```


```lua
function Normal(mu, sigma)
	local particules = {}
	for i=1,1000000 do
		table.insert(particules, sigma*InvNormCDF(math.random()) + mu) -- so nice
	end
	return particules
end
```
```output[21](5/3/2023 12:46:23 AM)
```

```lua
function plot(p, min_x, max_x, N)
	local grid = {}
	local dx = (max_x-min_x)/N
	for _, pi in ipairs(p) do
		local n = math.ceil((pi - min_x)/dx)
		grid[n] = (grid[n] or 0) + 1
	end

	local x = {}
	local y = {}
	local lp = #p
	for i=1,N do
		table.insert(x, (i-0.5)*dx + min_x)
		table.insert(y, (grid[i] or 0)/(dx*lp))
	end
	plt.scatter(x,y)
end
```
```output[35](5/3/2023 12:52:13 AM)
```

```lua
plot(Normal(0,1), -3, 3, 50)
```
```output[24](5/3/2023 12:46:40 AM)
```

```lua
function square(p)
	return vim.tbl_map(function(x) return x*x end, p)
end
```
```output[33](5/3/2023 12:52:06 AM)
```

```lua
local x1 = Normal(0,1)
plot(square(x1), 0, 0.1, 50)
```
```output[38](5/3/2023 12:52:47 AM)
```

```lua
function add(p1, p2)
	local newp = {}
	for i=1,math.sqrt(#p1) do
		for j=1,math.sqrt(#p2) do
			table.insert(newp, p1[i] + p2[j])
		end
	end
	return newp
end
```
```output[39](5/3/2023 12:54:21 AM)
```

```lua
local x1 = Normal(0,1)
local x2 = Normal(0,1)
local y = add(x1,x2)
plot(y, -5, 5, 30)
```
```output[42](5/3/2023 12:55:21 AM)
```

```lua
function E(p)
	local sum = 0
	for _, pi in ipairs(p) do
		sum = sum + pi
	end
	return sum / #p
end
```
```output[46](5/3/2023 12:58:01 AM)
```

```lua
print(E(Normal(2,4)))
```
```output[50](5/3/2023 12:58:27 AM)
1.997524962141
```

```lua
function Var(p)
	local p2 = square(p)
	return E(p2) - E(p)^2
end
```
```output[51](5/3/2023 12:59:39 AM)
```

```lua
print(Var(Normal(0,1)))
```
```output[53](5/3/2023 1:00:09 AM)
0.99743614401543
```

```lua
function ChiSquared(k)
	local sum = square(Normal(0,1))
	for i=2,k do
		sum = add(sum, square(Normal(0,1)))
	end
	return sum
end
```
```output[55](5/3/2023 1:01:30 AM)
```


```lua
plot(ChiSquared(4), 0, 4, 10)
```
```output[66](5/3/2023 1:04:50 AM)
```


