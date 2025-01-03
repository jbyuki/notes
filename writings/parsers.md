Parsers
=======

Learning about the parsers. Some informations could be wrong.

* **Top-down**: Build the parse tree starting from the _starting_ rule and essentially reading the rules from **left** to **right**, thus going top-down the rule tree. Examples are recursive descent parsers, parser combinators. They are easy to implement which contribute to their attractiveness.

	```
	Starting rule: A
	A -> Bz | y
	B -> x | z
	```

	* **Left recursion**: One notable issue of top-down parsers is the left recursion problem for a rule such as this:

		```
		A -> Az | y
		```

		Trying to matching this rule naively will lead to an infinite loop. There seem to be ad-hoc methods to solve this issue but I haven't delve too much into it. Also any left recursive rule can be transformed to a non-left recursive rule by rewriting, so in practice this is minor it seems.

	* **Backtracking**: One other issue is that if a certain rule fail, the top-down parser will backtrack and try with another rule. In certain scenarios, this can lead to exponentional time complexity. This is solved by the [Prackat parser](https://bford.info/pub/lang/packrat-icfp02.pdf) using memoization achieving linear time complexity. The paper on Prackat parser is highly interesting. The left recursion problem is not solved by memoization.

* **Bottom-up**: Build the parse tree by repeatdly reducing the tokens according to the rules. Reading the rules from **right** to **left**, thus going bottom-up the rule tree. Examples are LR(k), GLR, [Pika Parser](https://arxiv.org/abs/2005.06444).

	* **Left recursion**: This is not an issue for bottom-up parsers.

Grammars
========

* **CFG**: Context-free grammar. The LHS of the rule only contains one contain non-terminal symbol.
	* Top-down: LL(k) ? not sure about this one
	* Bottom-up: LR(k)

* **PEG**: Parser expression grammar. Same as CFG but matches eagerly. Also integrates nicely with Regex as it also matches eagerly. Thus the lexical analysis part can be expressed directly which is nice.
	* Top-down: Recursive descent parser, [Prackat parser](https://bford.info/pub/lang/packrat-icfp02.pdf)
	* Bottom-up: [Pika parser](https://arxiv.org/abs/2005.06444)

Pratt's Parser
==============

* Bottom-up ? not sure
* Is not expressed directly using a grammar but instead associate BP (binding power) to unary and binary operators.
