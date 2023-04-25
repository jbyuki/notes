
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

function N(di, p, depth)
	local result = ""
	if p == 0 then
		result = p == 0 and "" or "0"
	else
		if di > 0 then
			result = "0"
		elseif di < -p then
			result = "0"
		elseif di == 0 then
			if depth == p-1 then
				result = ("\\left( \\frac{u^{\\ast}-u_{%s}}{u_{%s} - u_{%s}} \\right)"):format(i_index(di), i_index(di+p), i_index(di))
			else
				result = (" \\frac{u^{\\ast}-u_{%s}}{u_{%s} - u_{%s}}"):format(i_index(di), i_index(di+p), i_index(di))
			end
			result = result .. N(di, p-1, depth+1)
		elseif di == -p then
			result = ("\\frac{u_{%s}-u^{\\ast}}{u_{%s} - u_{%s}}"):format(i_index(di+p+1), i_index(di+p+1), i_index(di+1))
			result = result .. N(di+1, p-1, depth+1)
		else
			if depth == p-1 then
				result = ("\\left( \\frac{u^{\\ast}-u_{%s}}{u_{%s} - u_{%s}} \\right)"):format(i_index(di), i_index(di+p), i_index(di))
			else
				result = (" \\frac{u^{\\ast}-u_{%s}}{u_{%s} - u_{%s}}"):format(i_index(di), i_index(di+p), i_index(di))
			end
			result = result .. N(di, p-1, depth+1)
			result = result .. " \\\\\\\\&+ "
			result = result .. ("\\frac{u_{%s}-u^{\\ast}}{u_{%s} - u_{%s}}"):format(i_index(di+p+1), i_index(di+p+1), i_index(di+1))
			result = result .. N(di+1, p-1, depth+1)
		end
	end
	return result
end

for p=0,5 do
	print("---")
	for di=0,-p,-1 do
		print(("$$\\begin{align*}%s &= %s\\end{align*}$$"):format(("N_{%s,%d}(u^{\\ast})"):format(i_index(di), p), N(di, p, 0)))
		print("")
	end
end
print("---")
```
