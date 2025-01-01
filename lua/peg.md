**Question**: Can a grammar of this form be understood by PEG:

```
Starting: Expr
Expr -> Number | '(' Expr ')' | Expr '*' Expr | Expr '+' Expr
Number -> [0-9]+
```

Let's see with `lpeg`. Conveniently, it is readily available in nvim.

```lua
print(vim.inspect(lpeg))
```
```output[1](1/1/2025 4:30:35 PM)
{
  B = <function 1>,
  C = <function 2>,
  Carg = <function 3>,
  Cb = <function 4>,
  Cc = <function 5>,
  Cf = <function 6>,
  Cg = <function 7>,
  Cmt = <function 8>,
  Cp = <function 9>,
  Cs = <function 10>,
  Ct = <function 11>,
  P = <function 12>,
  R = <function 13>,
  S = <function 14>,
  V = <function 15>,
  locale = <function 16>,
  match = <function 17>,
  pcode = <function 18>,
  ptree = <function 19>,
  setmaxstack = <function 20>,
  type = <function 21>,
  utfR = <function 22>,
  version = "LPeg 1.1.0"
}
```

Let's enunciate the grammar rules:

```lua
calc = lpeg.P {
	"Expr";
	Expr = lpeg.V"Number" + "(" * lpeg.V"Expr" * ")" + lpeg.V"Expr" * "*" * lpeg.V"Expr" + lpeg.V"Expr" * "+" * lpeg.V"Expr",
	Number = lpeg.R("09")^1
}
```
```output[2](1/1/2025 7:02:10 PM)
[string "calc = lpeg.P {..."]:1: rule 'Expr' may be left recursive
```

Well, no, because of left recursiveness.
