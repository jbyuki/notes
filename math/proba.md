

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
```output[1](4/25/2023 11:18:11 PM)
```

```lua
function E(X)
	sum = 0
	for v,p in pairs(X) do
		sum = sum + v*p
	end
	return sum
end
```
```output[2](4/25/2023 11:18:11 PM)
```

```lua
print(E(Xw))
```
```output[3](4/25/2023 11:18:11 PM)
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
```output[4](4/25/2023 11:18:11 PM)
```


```lua
XYw = mul(Xw,Yw)
```
```output[5](4/25/2023 11:18:11 PM)
```

```lua
print(E(XYw), E(Xw)*E(Yw))
```
```output[6](4/25/2023 11:18:11 PM)
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
```output[7](4/25/2023 11:18:11 PM)
```


```lua
function Bernoulli(p)
	return {
		[1] = p,
		[0] = 1 - p,
	}
end
```
```output[8](4/25/2023 11:18:11 PM)
```

```lua
function Square(Aw)
	SqAw = {}
	for v,p in pairs(Aw) do
		SqAw[v*v] = p
	end
	return SqAw
end
```
```output[9](4/25/2023 11:18:11 PM)
```


```lua
function Var(Aw)
	SqAw = Square(Aw)
	return E(SqAw) - E(Aw)^2
end
```
```output[10](4/25/2023 11:18:11 PM)
```

```lua
p = 0.3
X1 = Bernoulli(p)
print(E(X1), p)
print(Var(X1), p*(1-p))
```
```output[11](4/25/2023 11:18:11 PM)
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
```output[37](4/25/2023 11:25:42 PM)
29.999999999999
29.100000000024 29.1
```


```lua
plt = require"plotly"
x = {}
y = {}
for v,p in pairs(Sn) do
	if v < 100 then
		table.insert(x, v)
		table.insert(y, p)
	end
end
plt.scatter(x,y)
```
```output[38](4/25/2023 11:25:43 PM)
```



