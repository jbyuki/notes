# Skiplist

After a small survey on B-trees, which are quite difficult to implement,
I'm turning my attention to skiplists because of their supposed simpler
implementation and still providing a O(log n) complexity for insertion,
search, and deletion.

The main reference will be [Skips Lists: A probabilistic alternative to
balanced trees](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf).

## Description

```
 HEADS
                     ┏━━━━━━━┓                             ╔═══════╗
  ╔═╗                ┃   │   ┃                             ║       ║
  ╚═╝──────────────► ┃   │   ┃────────────────────────────►║       ║
                     ┃   │   ┃                ┏━━━━━━━┓    ║       ║
  ╔═╗                ┃ 6 │   ┃                ┃   │   ┃    ║  NIL  ║
  ╚═╝──────────────► ┃   │   ┃───────────────►┃   │   ┃───►║       ║
         ┏━━━━━━━┓   ┃   │   ┃    ┏━━━━━━━┓   ┃   │   ┃    ║       ║
  ╔═╗    ┃ 3 │   ┃   ┃   │   ┃    ┃ 7 │   ┃   ┃ 9 │   ┃    ║       ║
  ╚═╝───►┗━━━━━━━┛─► ┗━━━━━━━┛───►┗━━━━━━━┛──►┗━━━━━━━┛───►╚═══════╝

```

Skiplists are a probabilisitc alternative to balanced trees. 

The idea is add levels to a simple linked list. A naive idea would be 
to add levels so that the first level, every element is linked, on the
second level, every other element is linked, on the next level, every
forth element is linked, etc... This is good for search but impratical
for insertion and deletion because all the levels above 1 need to be
  adjusted again.

Instead in skiplists, the elements are linked probabilistically with
a probability p.

## Insertion and Deletion

The maximum number of levels should be computed before hand.
The output is the maximum number of elements.

```lua
p = 0.5
MAX_LEVEL = 20
N = (1/p)^MAX_LEVEL
print(N)
```
```output[119](10/13/22 08:23:59)
1048576
```

Let's create the skip list structure.

```lua
MAX_KEY = math.huge

skiplist = {}
mt = { __index = skiplist }
function skiplist.new()
  local header = {
    forward = {}
  }

  local nil_element = {
    key = MAX_KEY
  }

  for i=1,MAX_LEVEL do
    table.insert(header.forward, nil_element)
  end

  return setmetatable({
    header = header,
    level = 1
  }, mt)
end
```
```output[120](10/13/22 08:23:59)
```


```lua
function skiplist:insert(key)
  local update = {}
  local x = self.header
  for i=self.level,1,-1 do
    while x.forward[i].key < key do
      x = x.forward[i]
    end
    update[i] = x
  end

  local lvl = skiplist.random_level()
  if lvl > self.level then
    for i=self.level+1,lvl do
      update[i] = self.header
    end
    self.level = lvl
  end

  x = skiplist.new_node(key)
  for i=1,lvl do
    x.forward[i] = update[i].forward[i]
    update[i].forward[i] = x
  end
end
```
```output[121](10/13/22 08:23:59)
```

The random_level() will choose a level for the new node.


```lua
function skiplist.random_level()
  local lvl = 1
  while math.random() < p and lvl < MAX_LEVEL do
    lvl = lvl + 1
  end
  return lvl
end
```
```output[122](10/13/22 08:23:59)
```

Let's try the function a few times

```lua
for i=1,10 do
  local lvl = skiplist.random_level()
  print(("%d ( %g %% chance )"):format(lvl, math.floor((p^lvl)*10000)/100))
end
```
```output[123](10/13/22 08:23:59)
1 ( 50 % chance )
1 ( 50 % chance )
1 ( 50 % chance )
5 ( 3.12 % chance )
1 ( 50 % chance )
2 ( 25 % chance )
1 ( 50 % chance )
3 ( 12.5 % chance )
2 ( 25 % chance )
1 ( 50 % chance )
```

new_node() will create a new empty node.


```lua
function skiplist.new_node(key)
  return {
    forward = {},
    key = key,
  }
end
```
```output[124](10/13/22 08:23:59)
```

A regular printing routine.

```lua
function mt:__tostring()
  local elem = self.header.forward[1]
  local keys = {}
  while elem.forward do
    table.insert(keys, tostring(elem.key))
    elem = elem.forward[1]
  end
  return table.concat(keys, ", ")
end
```
```output[131](10/13/22 08:25:06)
```

Testing the insertion of keys.

```lua
local L = skiplist.new()
for i=1,20 do
  L:insert(i)
end
print(L)
```
```output[133](10/13/22 08:25:12)
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
```

