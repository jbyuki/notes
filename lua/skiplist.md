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
N = (1/p)^max_level
print(N)
```
```output[55](10/13/22 00:16:01)
1048576
```

Let's create the skip list structure.

```lua
MAX_KEY = math.huge

local skiplist = {}

function skiplist.new()
  return {
    header = {
      forward = { {
        key = MAX_KEY
      } }
    },
    level = 1
  }
end
```
```output[28](10/13/22 00:08:42)
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
      update[i] = list.header
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
```output[29](10/13/22 00:08:45)
```

The random_level() will choose a level for the new node.


```lua
function skiplist.random_level()
  local lvl = 1
  while math.random() < p and lvl < max_level do
    lvl = lvl + 1
  end
  return lvl
end
```
```output[30](10/13/22 00:08:47)
```

Let's try the function a few times

```lua
for i=1,10 do
  local lvl = skiplist.random_level()
  print(("%d ( %g %% chance )"):format(lvl, math.floor((p^lvl)*10000)/100))
end
```
```output[31](10/13/22 00:08:49)
1 ( 50 % chance )
5 ( 3.12 % chance )
1 ( 50 % chance )
2 ( 25 % chance )
2 ( 25 % chance )
3 ( 12.5 % chance )
1 ( 50 % chance )
1 ( 50 % chance )
1 ( 50 % chance )
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
```output[32](10/13/22 00:08:51)
```

