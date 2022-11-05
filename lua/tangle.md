The tangling algorithm
----------------------

These are some ideas on the data structure to store
and process untangled code, to tangle it, more specifically
incrementally tangle it (similar to incremental compilation).

Let's take an example
```
@hello=
@sec1
aaaaaaaa
@sec1+=
bbbbbbbbb
@sec1+=
ccccccccc
```

The code is composed of a root section `hello` which contains
a reference to sec1 followed by a line "aaaaaaaa". The sec1
section contains "bbbbbbbbb" followed by "ccccccccc". 

This structure ressemble one of a DAG (directed acyclic graph).
This is good to keep in mind but this is probably more anecdotic
than useful.

```
   ┌───┐       │ ┌────────────┐
   │MAP│       │ │LINKED LISTS│
   └───┘       │ └────────────┘
 KEY     VALUE │
               │
"hello" [HEAD]───► hello 
               │     ▼
               │   @sec1
               │     ▼ 
               │   aaaaaaaaa
               │
"sec1"  [HEAD]───► sec1 ──────► sec1
               │     ▼           ▼ 
               │   bbbbbbbbb    ccccccccc
```

This is the data structure used in most of my plugins such as
[ntangle.nvim](www.github.com/jbyuki/ntangle.nvim). It is composed
of a main map called sectionlist where the key is the section name
and value is the head of a linkedlist of every section definition
and under each section definition is another linked list or array
with the lines which can either be text or references to sections.

The idea is to augment this data structure so that it also plays
nicely with incremental tangling.

It has to support:

1) Fast insertion/deletion of a line specified by a line number
   in the untangled code. Fast is O(log n).

2) Fast lookup of the line number given a line where the line
   number is in the tangled code.

My initial idea is to use a B-tree or B+ tree for 1) and augment
the data structure with a LOC number of each map entry for 2),
then given a line walk backward until the root section to find
the absolute line number for a line. Each section part in the
linked list could also hold its LOC for a faster lookup.

Implementation
--------------

The untangled code:

```lua
untangled = {
  "@hello=",
  "@sec1",
	"@sec2",
  "aaaaaaaaaaaaa",
  "@sec1+=",
  "bbbbbbbbbbbbb",
	"@sec2",
  "@sec1+=",
  "ccccccccccccc",
  "@sec2+=",
  "ddddddddddddd",
}

print(("#untangled = %d"):format(#untangled))
```
```output[137](11/05/22 16:19:32)
#untangled = 11
```

For the parsing, the data structure mentionned above will be constructed. 
Instead of using linked lists, we will just use regular tables for simplicity.

First the parse function

```lua
function parse(lines, section_list)
	local cur_section = {}

	for lnum, line in ipairs(lines) do
		-- line with @ escaped
		if string.match(line, "^%s*@@") then
			local _,_,pre,post = string.find(line, '^(.*)@@(.*)$')
			local text = pre .. "@" .. post
			local l = { 
				linetype = "text",
				str = text 
			}

			table.insert(cur_section, l)

		-- line is section
		elseif string.match(line, "^@[^@]%S*[+-]?=%s*$") then
			local _, _, name, op = string.find(line, "^@(%S-)([+-]?=)%s*$")

			section_list[name] = section_list[name] or {}
			cur_section = {}
			table.insert(section_list[name], cur_section)

		-- line is reference
		elseif string.match(line, "^%s*@[^@]%S*%s*$") then
			local _, _, prefix, name = string.find(line, "^(%s*)@(%S+)%s*$")
			local l = { 
				linetype = "reference",
				str = name,
				prefix = prefix
			}

			table.insert(cur_section, l)

		-- line is just plain text
		else
			local l = { 
				linetype = "text",
				str = line 
			}
			table.insert(cur_section, l)
		end
	end
end

```
```output[138](11/05/22 16:19:32)
```


```lua
sec_map = {}
parse(untangled, sec_map)
print(#sec_map["hello"])
print(#sec_map["sec1"])
```
```output[139](11/05/22 16:19:32)
1
2
```

`sec1` contains two entries because the section was defined two times
while `hello` only contains one entry. Looking in more details.


```lua
print(vim.inspect(sec_map["hello"][1]))
```
```output[140](11/05/22 16:19:32)
{ {
    linetype = "reference",
    prefix = "",
    str = "sec1"
  }, {
    linetype = "reference",
    prefix = "",
    str = "sec2"
  }, {
    linetype = "text",
    str = "aaaaaaaaaaaaa"
  } }
```


```lua
print(vim.inspect(sec_map["sec1"][1]))
```
```output[141](11/05/22 16:19:32)
{ {
    linetype = "text",
    str = "bbbbbbbbbbbbb"
  }, {
    linetype = "reference",
    prefix = "",
    str = "sec2"
  } }
```


```lua
print(vim.inspect(sec_map["sec1"][2]))
```
```output[142](11/05/22 16:19:32)
{ {
    linetype = "text",
    str = "ccccccccccccc"
  } }
```

Now that the code has been successfully parsed, actually this is just a copy-paste
of ntangle.nvim, the datastructure will be augmented with the number of lines.

The general idea is to keep to iterate through the section in the map and each time
count the number of lines under that section, going recursively if needed. Speed
up the process with memoization.


```lua
line_count = {} -- map (k: section, v:line count)

function rec_count(name) 
	local count = line_count[name]
	if count then
		return count[1]
	else
		count = 0
		local sec_counts = {}
		if not sec_map[name] then
			return 0
		end
		for _, sec in ipairs(sec_map[name]) do
			local sec_count = 0
			for _, line in ipairs(sec) do
				if line.linetype == "text" then
					sec_count = sec_count + 1
				elseif line.linetype == "reference" then
					sec_count = sec_count + rec_count(line.str)
				else
					assert(false)
				end
			end
			count = count + sec_count
			table.insert(sec_counts, sec_count)
		end

		line_count[name] = { count, sec_counts }
		return count
	end
end

for name, _ in pairs(sec_map) do
	local count = rec_count(name)
end

print(vim.inspect(line_count))
```
```output[143](11/05/22 16:19:32)
{
  hello = { 5, { 5 } },
  sec1 = { 3, { 2, 1 } },
  sec2 = { 1, { 1 } }
}
```

With this datastructure, it's possible to seek a particular line number more quickly.


```lua
function get_line(root, lnum) 
	local count, sec_count = unpack(line_count[root])
	local i = 1
	while i < #sec_count and lnum > sec_count[i] do
		lnum = lnum - sec_count[i]
		i = i + 1
	end

	local ssec = sec_map[root][i]
	local k = 1
	while k <= #ssec do
		if ssec[k].linetype == "text" then
			if lnum == 1 then
				return ssec[k].str
			end
			lnum = lnum - 1
		elseif ssec[k].linetype == "reference" then
			local count, _ = unpack(line_count[ssec[k].str])
			if lnum <= count then
				return get_line(ssec[k].str, lnum)
			end
			lnum = lnum - count
		else
			assert(false)
		end
		k = k + 1
	end
end
```
```output[144](11/05/22 16:19:32)
```

```lua
for i=1,5 do
	local line = get_line("hello", i)
	print(line)
end
```
```output[145](11/05/22 16:19:32)
bbbbbbbbbbbbb
ddddddddddddd
ccccccccccccc
ddddddddddddd
aaaaaaaaaaaaa
```
