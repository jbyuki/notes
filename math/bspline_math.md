
$$
\begin{align*}
N_{i,0}(u) &= \begin{cases} 1 & \text{if   } u_i \le u < u_{i+1} \\ 0 & \text{otherwise} \end{cases} \\
N_{i,p}(u) &= \frac{u-u_i}{u_{i+p} - u_i} N_{i,p-1}(u) + \frac{u_{i+p+1}-u}{u_{i+p+1} - u_{i+1}} N_{i+1,p-1}(u)
\end{align*}
$$

Define $u^* \in [u_i, u_{i+1})$.

Then

```lua
local i_index = function(di)
	if di == 0 then return "i"
	elseif di < 0 then return ("i-%d"):format(-di)
	else return ("i+%d"):format(di) end
end

function N(di, p)
	local result = ""
	if p == 0 then
		result = p == 0 and "" or "0"
	else
		if di > 0 then
			result = "0"
		elseif di < -p then
			result = "0"
		elseif di == 0 then
			result = ("\\frac{u^{\\ast}-u_{%s}}{u_{%s} - u_{%s}}"):format(i_index(di), i_index(di+p), i_index(di))
			result = result .. N(di, p-1)
		elseif di == -p then
			result = ("\\frac{u_{%s}-u^{\\ast}}{u_{%s} - u_{%s}}"):format(i_index(di+p+1), i_index(di+p+1), i_index(di+1))
			result = result .. N(di+1, p-1)
		else
			result = ("\\frac{u^*-u_{%s}}{u_{%s} - u_{%s}}"):format(i_index(di), i_index(di+p), i_index(di))
			result = result .. N(di, p-1)
			result = result .. " + "
			result = result .. ("\\frac{u_{%s}-u^{\\ast}}{u_{%s} - u_{%s}}"):format(i_index(di+p+1), i_index(di+p+1), i_index(di+1))
			result = result .. N(di+1, p-1)
		end
	end
	return result
end

local di = -1
local p = 1
print(("$$%s = %s$$"):format(("N_{%s,%d}(u^{\\ast})"):format(i_index(di), p), N(di, p)))
```
```output[39](4/18/2023 9:11:54 PM)
```

---

$$N_{i,0}(u^{\ast}) = 1$$

---

$$N_{i,1}(u^{\ast}) = \frac{u^{\ast}-u_{i}}{u_{i+1} - u_{i}}$$

$$N_{i-1,1}(u^{\ast}) = \frac{u_{i+1}-u^{\ast}}{u_{i+1} - u_{i}}$$
