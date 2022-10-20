# Treesitter

This is some tests using the string parser with 
c language snippets

The snippets which will be first used will be the
following.

```lua
str = [[
int unused;

int main()
{
	int a;
	int b;
	int d;
	int c = a + b;
}
]]

str = table.concat(vim.split(str, "\n"))
print(str)
```
```output[1](10/20/22 14:29:54)
int unused;int main(){	int a;	int b;	int d;	int c = a + b;}
```

The parser is created. New neovim versions allow to
directly parse strings easily.

The neovim version used for this document is:


```lua
print(vim.inspect(vim.version()))
```
```output[2](10/20/22 14:29:54)
{
  api_compatible = 0,
  api_level = 10,
  api_prerelease = false,
  major = 0,
  minor = 9,
  patch = 0,
  prerelease = true
}
```

The string parser is created.


```lua
parser = vim.treesitter.get_string_parser(str, "c")
print(vim.inspect(parser))
```
```output[3](10/20/22 14:29:54)
{
  _callbacks = {
    bytes = {},
    changedtree = {},
    child_added = {},
    child_removed = {},
    detach = {}
  },
  _children = {},
  _injection_query = {
    captures = <1>{ "c" },
    info = {
      captures = <table 1>,
      patterns = {}
    },
    query = <userdata 1>,
    <metatable> = <2>{
      __index = <table 2>,
      apply_directives = <function 1>,
      iter_captures = <function 2>,
      iter_matches = <function 3>,
      match_preds = <function 4>
    }
  },
  _lang = "c",
  _opts = {},
  _parser = <userdata 2>,
  _regions = {},
  _source = "int unused;int main(){\tint a;\tint b;\tint d;\tint c = a + b;}",
  _trees = {},
  _valid = false,
  <metatable> = <3>{
    __index = <table 3>,
    _do_callback = <function 5>,
    _get_injections = <function 6>,
    _on_bytes = <function 7>,
    _on_detach = <function 8>,
    _on_reload = <function 9>,
    add_child = <function 10>,
    children = <function 11>,
    contains = <function 12>,
    destroy = <function 13>,
    for_each_child = <function 14>,
    for_each_tree = <function 15>,
    included_regions = <function 16>,
    invalidate = <function 17>,
    is_valid = <function 18>,
    lang = <function 19>,
    language_for_range = <function 20>,
    named_node_for_range = <function 21>,
    new = <function 22>,
    parse = <function 23>,
    register_cbs = <function 24>,
    remove_child = <function 25>,
    set_included_regions = <function 26>,
    source = <function 27>,
    tree_for_range = <function 28>,
    trees = <function 29>
  }
}
```


The string is parsed and the tree is returned.


```lua
tree = parser:parse()
tree = tree[1]
root = tree:root()
print(root:sexpr())
```
```output[4](10/20/22 14:29:54)
(translation_unit (declaration type: (primitive_type) declarator: (identifier)) (function_definition type: (primitive_type) declarator: (function_declarator declarator: (identifier) parameters: (parameter_list)) body: (compound_statement (declaration type: (primitive_type) declarator: (identifier)) (declaration type: (primitive_type) declarator: (identifier)) (declaration type: (primitive_type) declarator: (identifier)) (declaration type: (primitive_type) declarator: (init_declarator declarator: (identifier) value: (binary_expression left: (identifier) right: (identifier)))))))
```

Create a search pattern to detect the main function and its body.


```lua
local search_pattern = [[
(function_definition declarator: (function_declarator declarator: (identifier) @name (#eq? @name "main")) body: (compound_statement) @body)
]]

local search_pattern2 = [[
(function_definition declarator: (function_declarator declarator: (identifier) @id) body: (compound_statement) @body)
]]

local query = vim.treesitter.parse_query("c", search_pattern2)
for pattern, match in query:iter_matches(root, str) do
    for id, node in pairs(match) do
		local row1, col1, row2, col2 = node:range() -- range of the capture
		local name = query.captures[id]
		if name == "id" then
			name = str:sub(col1+1, col2)
		else
			body_node = node
		end
	end

	if name == "main" then
		break
	end
end

for child, _ in body_node:iter_children() do
	local row1, col1, row2, col2 = child:range() -- range of the capture
	local type = child:type()
	if type ~= "{" and type ~= "}" then
		print(child:field("type")[1]:sexpr())
		local txt = str:sub(col1+1, col2)
		print(txt, ("."):rep(10), child:sexpr())
	end
end
```
```output[8](10/20/22 14:30:40)
(primitive_type)
int a; .......... (declaration type: (primitive_type) declarator: (identifier))
(primitive_type)
int b; .......... (declaration type: (primitive_type) declarator: (identifier))
(primitive_type)
int d; .......... (declaration type: (primitive_type) declarator: (identifier))
(primitive_type)
int c = a + b; .......... (declaration type: (primitive_type) declarator: (init_declarator declarator: (identifier) value: (binary_expression left: (identifier) right: (identifier))))
```

Some nodes are hidden or anonymous. For example the operator node
in binary expression.

```lua
local decl_node = body_node:child(4)
local init_decl = decl_node:field("declarator")[1]
local value_node = init_decl:field("value")[1]
print(value_node:sexpr())
local op_node = value_node:field("operator")[1]
print(op_node)
```
```output[17](10/20/22 14:33:07)
(binary_expression left: (identifier) right: (identifier))
<node +>
```
