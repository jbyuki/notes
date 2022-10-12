# B-tree

This is a short document on B-trees. I'm writing the document at the same
time I'm learning about B-trees but hopefully my prior knowledge should
serve me and not write something too wrong. The coding will be done entirely
in Lua as this is the built-in language in the editor I use, Neovim.

The source of reference is "Introduction to Algorithms (Third edition)" written
by Cormen, Leiserson, Rivest and Stein. (Chapter 18)

## Motivation

My motivation comes from the Neovim editor's source code. B-trees are used
in several areas in the source code already and I plan to use them in the future
also for an extension of Neovim's feature for my personal benefit (and also
anyone who is interested by these features). The features in question
relate to the addition of first class literate programming support in Neovim.

## Description

B-trees are balanced search trees similar to red-black trees but offer optimizations
for the number of disk access over the ladder. It is a natural generalization of
binary trees by having potentially a lot of children per node.

To go into more details, the tree has different types of nodes:

* **root node**: This is the special node at the top
* **internal node**. This is an intermediate node. It contains keys, where each key
  is sorted in a non-decreasing order. If the key is laid out as a sequence, between
  each pair of contiguous key or before the first key and after the last key, is a pointer
  to a child node, where the keys in the child nodes are all between the delimiting keys
  of the internal node. This will become more clear later.
* **leaf node**. Leaf nodes do not contain any child nodes but can contain keys.

Additionnally, there are some rules about the key structure:

* *t* is a constant which defines the **minimum** and **maximum** number of keys. Any
  node other than the root node must have at least t-1 keys. Thus, by consequence,
  any internal node has at least t children. Any node contains at most 2t - 1 keys,
  by consequence, any internal node has at most 2t children.


For example, if t = 2, the tree is called a **2-3-4 tree** because it can have either
2, 3 or 4 children. In practice, much larger values of t are used.

I won't go into the details about the height, but basically the height is constrained
by the rules that were decided. It is O(log n) where n is the number of keys.

## Implementation

For the interesting part now, the implementation of B-trees.

### Create an empty B-tree

The tree must have a pointer to the root node, each node has the number
of keys it contains n, and the pointer to the children.


```lua
btree = {}
t = 3
mt = { __index = btree }
function btree.new()
  local o = {}
  o.root = {
    keys = {},
    children = {},
    leaf = true,
    n = 0,
  }

  return setmetatable(o,mt)
end
```
```output[1](10/11/22 22:49:43)
```

Create a tree T which will be used for testing throughout.


```lua
T = btree.new()
print(vim.inspect(T))
```
```output[2](10/11/22 22:49:47)
{
  root = {
    children = {},
    keys = {},
    leaf = true,
    n = 0
  },
  <metatable> = {
    __index = {
      new = <function 1>
    }
  }
}
```

### Insert key into a B-tree

A naive way would be to find a node where the key should belong and insert it there. But
of course, the node could be full, having reached its upper bound. That's where the splitting
operation comes into play. A node is split by identifying the **median key**, moving it up,
into the parent and splitting the node in two. Again, the parent could be full also so,
we would do the same procedure again all the way up. Because the B-tree insertion was thought
by very clever people, the tree is instead split prematurely so to speak, when traveling down
to find the correct node for the key already. Any internal node which is full, is split on the 
way down. So that the key insertion does not have this problem.

A question might arise, what happens when the root node needs to be split. In that case, a new empty
node is created and put on top of the root node.

#### Split a node in a B-tree

A helper procedure will split an individual node. It takes as input a non-full parent node and a 
full child node. 

But first a function to create a new node.


```lua
function btree.new_node()
  return {
    leaf = true,
    children = {},
    keys = {},
    n = 0,
  }
end
```
```output[3](10/11/22 22:49:51)
```

Now the function to split, the child indexed at child_i is split into
left_child and right_child.

```lua
function btree.split(parent, child_i)
  local left_child = parent.children[child_i]
  local right_child  = btree.new_node()

  right_child.leaf = left_child.leaf 
  right_child.n = t-1

  -- Transfer keys to right side
  for i=1,t-1 do
    right_child.keys[i] = left_child.keys[i+t]
  end

  -- Transfer children to right side
  if not left_child.leaf then
    for i=1,t do
      right_child.children[i] = left_child.children[i+t]
    end
  end

  left_child.n = t-1


  -- Make room for the new median key
  for i=parent.n+1,child_i+1,-1 do
    parent.children[i+1] = parent.children[i]
  end

  parent.children[child_i+1] = right_child

  for i=parent.n,child_i,-1 do
    parent.keys[i+1] = parent.keys[i]
  end

  parent.keys[child_i] = left_child.keys[t]
  parent.n = parent.n + 1
end
```
```output[4](10/11/22 22:49:55)
```

#### Inserting key into a B-tree in a single pass

The insertion procedure is as follows. It starts with the root node and
check whether it should be split or not. That is if the root node
is full, it should split it. Next, a helper function
which takes a node non - full as parameter, so the root node, will recurse
on internal nodes or if it's a leaf node, it will insert the key.
Because the node as input is always non-full in that helper function,
if the node is a leaf, the key can be inserted, this is guaranteed.


```lua
function btree:insert(k)
  if self.root.n == 2*t - 1 then
    local new_root = btree.new_node()
    new_root.leaf = false
    new_root.children[1] = self.root
    self.root = new_root
    btree.split(new_root, 1)
    btree.insert_nonfull(new_root, k)
  else
    btree.insert_nonfull(self.root, k)
  end
end
```
```output[5](10/11/22 22:49:58)
```

And now the procedure for the non-full node key insertion. This adds the key in case
of a leaf node, and recurses in case of an internal node. Additionnaly, if the
child node in the internal node is full, split it.


```lua
function btree.insert_nonfull(node, k)
  local i = node.n
  if node.leaf then
    -- Shift bigger keys to the right
    while i >= 1 and k < node.keys[i] do
      node.keys[i+1] = node.keys[i]
      i = i - 1
    end
    node.keys[i+1] = k
    node.n = node.n + 1
    
    -- we're done, the key was added to the tree.
  else
    -- Find which child should contain the key
    while i >= 1 and k < node.keys[i] do
      i = i - 1
    end
    i = i + 1

    if node.children[i].n == 2*t - 1 then
      btree.split(node, i)
      if k > node.keys[i] then
        i = i + 1
      end
    end

    btree.insert_nonfull(node.children[i], k)
  end
end
```
```output[6](10/11/22 22:50:00)
```

The insertion code is now done. We will now to see what the results
look like but first, some functions to print out the tree
in a format that raw lua tables.

#### Print formatting functions

First a recursive function to print out a node. This populates a list
of strings, corresponding to each lines.


```lua
function btree.print_node(lines, node, depth)
  if not node then return end

  for i=1,node.n do
    if i == 1 then
      btree.print_node(lines, node.children[i], depth+1)
    end
    table.insert(lines, ("  "):rep(depth) .. node.keys[i])
    btree.print_node(lines, node.children[i+1], depth+1)
  end
end
```
```output[7](10/11/22 22:50:05)
```

Then a metamethod for string formatting so that the tree can be directly
output.


```lua
function mt.__tostring(tree)
  local lines = {}
  btree.print_node(lines, tree.root, 0)
  return table.concat(lines, "\n")
end
```
```output[8](10/11/22 22:50:08)
```

Printing the empty tree should yield nothing.


```lua
print(T)
```
```output[9](10/11/22 22:50:10)
```


Now we insert a key and print it.

```lua
T = btree.new()
T:insert(3)
print(T)
```
```output[10](10/11/22 22:50:12)
3
```

And insert several keys at random and print the tree.


```lua
T = btree.new()
for i=1,10 do
  local new_key = math.random(1,100)
  T:insert(new_key)
end
print(T)
```
```output[22](10/06/22 23:30:12)
  8
20
  37
  37
68
  69
81
  83
  99
  100
```

Let's try to insert the number in an increasing order and see the result

```lua
local start = vim.fn.reltime()
for m=1,1000 do
  local T = btree.new()
  for i=1,1000 do
    T:insert(i)
  end
end
local elapsed = vim.fn.reltimefloat(vim.fn.reltime(start))
print(elapsed)
```
```output[19](10/11/22 23:13:02)
0.9027144
```

**Why does the tree have this shape?**

### Searching a B-tree

The search is very similar to a binary tree, 
it will search through the tree recursively. Because
the keys are ordered, it can be done in a single pass.

```lua
function btree.search_node(node, k)
  local i = 1
  while i <= node.n and k > node.keys[i] do
    i = i + 1
  end

  if i <= node.n and k == node.keys[i] then
    return {node, i}
  end

  if node.leaf then
    return nil
  end

  return btree.search_node(node.children[i], k)
end

function btree:search(k)
  return btree.search_node(self.root, k)
end
```
```output[49](10/06/22 23:42:18)
```

Let's test it.


```lua
print(T:search(3))
```
```output[52](10/06/22 23:43:05)
table: 0x01ee98340950
```

The function returns a table, which means the key was found.
Otherwise the result will be nil like this.

```lua
print(T:search(101))
```
```output[53](10/06/22 23:43:35)
```

### Deleting a key from the B-tree


```lua
function btree.delete_nonmin(node, k, leftmost, rightmost)
  if node.leaf then
    if leftmost then
      k = node.keys[1]
      table.remove(node.keys, 1)
      node.n = node.n - 1
      return k
    elseif rightmost then
      k = node.keys[node.n]
      table.remove(node.keys)
      node.n = node.n - 1
      return k
    end
  end

  local i = 1
  if leftmost then
    i = 1
  elseif rightmost then
    i = node.n+1
  else
    while i <= node.n and k > node.keys[i] do
      i = i + 1
    end
  end

  if not leftmost and not rightmost and i <= node.n and k == node.keys[i] then
    -- case 1
    if node.leaf then
      table.remove(node.keys, i)
      node.n = node.n - 1
      return k
    -- case 2
    else
      -- case 2a
      if #node.children[i].keys >= t then
        local kp = btree.delete_nonmin(node.children[i], false, true)
        node.keys[i] = kp
        return k
      -- case 2b
      elseif #node.children[i+1].keys >= t then
        local kp = btree.delete_nonmin(node.children[i], true, false)
        node.keys[i] = kp
        return k
      -- case 2c
      else
        local left_child = node.children[i]
        local right_child = node.children[i+1]

        for j=i+1,node.n do
          node.children[j] = node.children[j+1]
        end

        for j=i,node.n-1 do
          node.keys[i] = node.keys[i+1]
        end

        node.n = node.n - 1

        left_child.keys[left_child.n+1] = k
        left_child.n = left_child.n + 1

        for j=1,right_child.n+1 do
          left_child.children[left_child.n+j] = right_child.children[j]
        end

        for j=1,right_child.n do
          left_child.keys[left_child.n+j] = right_child.keys[j]
        end

        left_child.n = left_child.n + right_child.n
        return btree.delete_nonmin(left_child, k)
      end
    end
  -- case 3
  else
    -- case 3a
    if #node.children[i].keys == t-1  then
      if node.children[i+1] and node.children[i+1].n >= t then
        local left_child = node.children[i]
        local right_child = node.children[i+1]

        left_child.keys[left_child.n+1] = node.keys[i]
        left_child.n = left_child.n + 1

        left_child.children[left_child.n+1] = right_child.children[1]
        node.keys[i] = right_child.keys[1]

        for j=1,right_child.n-1 do
          right_child.keys[j] = right_child.keys[j+1]
        end

        for j=1,right_child.n do
          right_child.children[j] = right_child.children[j+1]
        end

        right_child.n = right_child.n - 1
      elseif node.children[i-1] and node.children[i-1].n >= t then
        local left_child = node.children[i-1]
        local right_child = node.children[i]

        for j=right_child.n+1,2,-1 do
          right_child.keys[j] = right_child.keys[j-1]
        end

        for j=right_child.n+2,2,-1 do
          right_child.children[j] = right_child.children[j-1]
        end

        right_child.n = right_child.n + 1

        right_child.keys[1] = node.keys[i-1]
        right_child.children[1] = left_child.children[left_child.n+1]
        node.keys[i-1] = left_child.keys[left_child.n]

        left_child.n = left_child.n - 1

      -- case 3b
      elseif node.children[i+1] then
        local left_child = node.children[i]
        local right_child = node.children[i+1]

        left_child.keys[left_child.n+1] = node.keys[i]
        left_child.n = left_child.n + 1


        for j=1,right_child.n do
          left_child.keys[left_child.n+j] = right_child.keys[j]
        end

        for j=1,right_child.n+1 do
          left_child.children[left_child.n+j+1] = right_child.children[j]
        end

        left_child.n = left_child.n + right_child.n

        for j=i,node.n-1 do
          node.keys[j] = node.keys[j+1]
        end

        for j=i+1,node.n do
          node.children[j] = node.children[j+1]
        end

        node.n = node.n - 1

      elseif node.children[i-1] then
        local left_child = node.children[i-1]
        local right_child = node.children[i]

        for j=right_child.n,1,-1 do
          right_child.keys[j+1+left_child.n] = right_child.keys[j]
        end

        for j=right_child.n+1,1,-1 do
          right_child.children[j+1+left_child.n] = right_child.children[j]
        end

        right_child.keys[left_child.n+1] = node.keys[i]
        right_child.n = right_child.n + 1 + left_child.n

        for j=1,left_child.n do
          right_child.keys[j] = left_child.keys[j]
        end

        for j=1,left_child.n+1 do
          right_child.children[j] = left_child.children[j]
        end

        for j=i,node.n-1 do
          node.keys[j] = node.keys[j+1]
        end

        for j=i-1,node.n do
          node.children[j] = node.keys[j+1]
        end

        node.n = node.n - 1
      end
    end
    return btree.delete_nonmin(node.children[i], k)
  end
end
```
```output[55](10/07/22 01:39:34)
```

