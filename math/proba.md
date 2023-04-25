

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
```output[6](4/25/2023 9:07:03 PM)
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
```output[8](4/25/2023 9:07:41 PM)
```

```lua
print(E(Xw))
```
```output[9](4/25/2023 9:07:52 PM)
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
```output[15](4/25/2023 9:10:16 PM)
```


```lua
XYw = mul(Xw,Yw)
```
```output[16](4/25/2023 9:10:17 PM)
```

```lua
print(E(XYw), E(Xw)*E(Yw))
```
```output[19](4/25/2023 9:10:33 PM)
12.25 12.25
```



