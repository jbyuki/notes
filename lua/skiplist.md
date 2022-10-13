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

## Insertion

The maximum number of levels should be computed before hand.
The output is the maximum number of elements.

```lua
p = 0.5
MAX_LEVEL = 20
N = (1/p)^MAX_LEVEL
print(N)
```
```output[137](10/13/22 08:38:48)
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
```output[138](10/13/22 08:38:48)
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
```output[139](10/13/22 08:38:48)
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
```output[140](10/13/22 08:38:48)
```

Let's try the function a few times

```lua
for i=1,10 do
  local lvl = skiplist.random_level()
  print(("%d ( %g %% chance )"):format(lvl, math.floor((p^lvl)*10000)/100))
end
```
```output[141](10/13/22 08:38:48)
1 ( 50 % chance )
1 ( 50 % chance )
1 ( 50 % chance )
1 ( 50 % chance )
2 ( 25 % chance )
2 ( 25 % chance )
2 ( 25 % chance )
3 ( 12.5 % chance )
1 ( 50 % chance )
2 ( 25 % chance )
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
```output[142](10/13/22 08:38:48)
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
```output[143](10/13/22 08:38:48)
```

Testing the insertion of keys.

```lua
local L = skiplist.new()
for i=1,20 do
  L:insert(i)
end
print(L)
```
```output[144](10/13/22 08:38:48)
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
```

## Deletion 

The deletion of an element is done by first searching the element
and then deleting the key much like a linked list. Additionnally, 
reduce the level of the list if the highest level element is deleted.


```lua
function skiplist:delete(key)
  local update = {}
  x = self.header
  for i=self.level,1,-1 do
    while x.forward[i].key < key do
      x = x.forward[i]
    end
    update[i] = x
  end

  x = x.forward[1]
  if x.key == key then
    for i=1,self.level do
      if update[i].forward[i] ~= x then 
        break
      end
      update[i].forward[i] = x.forward[i]
    end

    while self.level > 1 do
      if not self.header.forward[self.level].forward then
        self.level = self.level - 1
      else
        break
      end
    end
  end
end
```
```output[145](10/13/22 08:38:48)
```

In the deletion process, see the drawing at the beginning,
if a node in update[i] is not linked to x, any other node
above in update will not be linked to x thus we can break
immediatly.

Let's test the method.


```lua
local L = skiplist.new()
for i=1,20 do
  L:insert(i)
end
L:delete(20)
L:delete(10)
L:delete(3)
print(L)
```
```output[149](10/13/22 08:39:10)
1, 2, 4, 5, 6, 7, 8, 9, 11, 12, 13, 14, 15, 16, 17, 18, 19
```

## Searching

The last method implemented here is searching. Thanks to
the links in the levels, it should be sublinear.

It returns true if the key is in the list, false otherwise.

```lua
function skiplist:search(key)
  local x = self.header
  for i=self.level,1,-1 do
    while x.forward[i].key < key do
      x = x.forward[i]
    end
  end
  x = x.forward[1]
  return x.key == key
end
```
```output[152](10/13/22 08:41:57)
```


```lua
local L = skiplist.new()
for i=1,20 do
  L:insert(i)
end
L:delete(20)
L:delete(10)
L:delete(3)
print(L:search(4))
print(L:search(3))
```
```output[153](10/13/22 08:41:58)
true
false
```

## Benchmarking

Finally some benchmarking to compare very approximatively with b-trees.
A better comparaison is done in the paper directly.

```lua
local start = vim.fn.reltime()
for m=1,1000 do
  local L = skiplist.new()
  for i=1,1000 do
    L:insert(i)
  end
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[155](10/13/22 08:43:25)
0.897807
```

Where for the same result with b-trees, it took `0.9027144`.
