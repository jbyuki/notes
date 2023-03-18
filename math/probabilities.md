Probabilities
-------------

$\sigma\text{-field}$
--------

1. $\Omega\ \in\ F$
2. $A\ \in\ F\ \rightarrow\ A^C\ \in\ F$
3. $A_1, A_2, ...\ \in\ F\ \rightarrow\ U_{i=1}^\infty A_i \in F$

Probability measure
-------------------

1. $P(\Omega)\ =\ 1$
2. $P(A)\ \ge\ 0$
3. $P(U_{i=1}^\infty A_i)\ =\ \sum_{i=1}^{\infty} P(A_i) \text{ if } \forall i \neq j \text{ we have } A_i \cap A_j = \emptyset$


Borel set
---------

$B = \sigma([x, y), x<y \in \Omega \text{)}$

Lebesgue measure
----------------

$P([x,y)\text{)} = y - x\text{, for } 0\le x \le y \le 1$

Random variable
---------------

$X: F\ \rightarrow\ \mathbb{R}$

is $F-measureable$ if $\{X \le x\}\in F$


$\sigma-field$ of random variable
---------------------------------

$\sigma(X)$ smallest $\sigma-field$ which makes X measurable with 
respect to $\sigma(X)$.

Independance of $\sigma-fields$
----------------------

$\sigma-fields$ are independant if whenever $A_i\ \in\ F_i \text{ for } i=1,...,n \text{ we have } P(\cap_{i=1}^\infty A_i)\ =\ \prod_{i=1}^{\infty} P(A_i)$

Independance of random variables
--------------------------------

$X_1,...,X_n$ independant if $\sigma(A_1),...,\sigma(A_n) \text{ are independant.}$

Expected value
--------------




 vim:ts=8:noet:ft=help:norl:
