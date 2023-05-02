

```lua
Xw = {
	[1] = 1/6,
	[2] = 1/6,
	[3] = 1/6,
	[4] = 1/6,
	[5] = 1/6,
	[6] = 1/6,
}

Yw = {
	[1] = 1/6,
	[2] = 1/6,
	[3] = 1/6,
	[4] = 1/6,
	[5] = 1/6,
	[6] = 1/6,
}
```
```output[1](5/2/2023 11:29:53 PM)
```

```lua
function E(X)
	if type(X) == "table" then
		sum = 0
		for v,p in pairs(X) do
			sum = sum + v*p
		end
		return sum
	elseif type(X) == "function" then
		sum = 0
		dx = 0.01
		for i=-1000,1000,dx do
			sum = sum + i*X(i)*dx
		end
		return sum
	end
end
```
```output[2](5/2/2023 11:29:53 PM)
```

```lua
print(E(Xw))
```
```output[3](5/2/2023 11:29:53 PM)
3.5
```


```lua
function mul(Aw, Bw)
	Cw = {}
	for a,pa in pairs(Aw) do
		for b,pb in pairs(Bw) do
			Cw[a*b] = (Cw[a*b] or 0) + pa*pb
		end
	end
	return Cw
end
```
```output[4](5/2/2023 11:29:53 PM)
```


```lua
XYw = mul(Xw,Yw)
```
```output[5](5/2/2023 11:29:53 PM)
```

```lua
print(E(XYw), E(Xw)*E(Yw))
```
```output[6](5/2/2023 11:29:53 PM)
12.25 12.25
```

```lua
```


```lua
function add(Aw, Bw)
	Cw = {}
	for a,pa in pairs(Aw) do
		for b,pb in pairs(Bw) do
			Cw[a+b] = (Cw[a+b] or 0) + pa*pb
		end
	end
	return Cw
end
```
```output[7](5/2/2023 11:29:53 PM)
```


```lua
function Bernoulli(p)
	return {
		[1] = p,
		[0] = 1 - p,
	}
end
```
```output[8](5/2/2023 11:29:53 PM)
```

```lua
function Square(Aw)
	if type(Aw) == "table" then
		SqAw = {}
		for v,p in pairs(Aw) do
			SqAw[v*v] = p
		end
		return SqAw
	elseif type(Aw) == "function" then
		return function(x) if x < 0 then return 0 else return 2*Aw(math.sqrt(x)) end end
	end
end
```
```output[9](5/2/2023 11:29:53 PM)
```


```lua
function Var(Aw)
	SqAw = Square(Aw)
	return E(SqAw) - E(Aw)^2
end
```
```output[10](5/2/2023 11:29:53 PM)
```

```lua
p = 0.3
X1 = Bernoulli(p)
print(E(X1), p)
print(Var(X1), p*(1-p))
```
```output[11](5/2/2023 11:29:53 PM)
0.3 0.3
0.21 0.21
```

```lua
n = 1000
lam = 30
Sn = Bernoulli(lam/n)
for i=2,n do
	Sn = add(Sn, Bernoulli(lam/n))
end

print(E(Sn))
print(Var(Sn), lam - lam^2/n)
```
```output[12](5/2/2023 11:29:53 PM)
29.999999999999
29.100000000024 29.1
```

```lua
function Normal(mu, sigma)
	local pdf = function(x)
		return math.exp(-0.5*((x - mu)/sigma)^2)/(sigma*math.sqrt(2*math.pi))
	end
	return pdf
end
```
```output[13](5/2/2023 11:29:53 PM)
```


```lua
local plt = require"plotly"
local X1 = Square(Normal(0, 1))
local x = {}
local y = {}
for i=-9,9,0.01 do
	table.insert(x, i)
	table.insert(y, X1(i))
end
plt.scatter(x, y)
print(X1(0))
```
```output[20](5/2/2023 11:33:13 PM)
0.79788456080287
```


```lua
print(Normal(0, 1))
```
```output[15](5/2/2023 11:29:53 PM)
function: 0x01b02a8805b8
```

