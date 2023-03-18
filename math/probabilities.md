Pnrobabilities
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

Continous random variable
-------------------------

If there exists a function $f$, called the density function  (pdf) such that:

$P(X\le x)= \int_{-\infty}^x f(t)\ dt\ \text{ for all } x$

Discrete random variable
------------------------

If it can only takes a countable number of different values.

Simple random variable
----------------------

If it can only take a finite number of possible values.


Expected value (discrete)
-------------------------

Def (1)

$E[x] = \sum_i x_i P(x=x_i)$

Expected value (continuous)
---------------------------

$E[x] =\int x f(x)\ dx$

Expected value (simple)
-----------------------

Def (2)

$E[x] \equiv sup_{\text{all simple r.v.} Y\le X} E[Y]$

Markov's inequality
-------------------

If $X\ge 0 \text{ and } a > 0$

$P(X \ge a)\ \le\ E[x]/a$

See (A.1) for details.

Chebyshev's inequality
----------------------

$P(|X| \ge a)\ =\ P(X^2 \ge a^2)\ \le\ E[X^2]/a^2$

Linearity of expected value
---------------------------

If $E|X|, E|Y|\ <\ \infty$ then

1) $E[aX+b]\ =\ aE[X] + b$

2) $E[X+Y]\ =\ E[X]+E[Y]$



A.1 Proof of Markov's inequality
--------------------------------

We have that $X\ge 0 \text{ and } a > 0$

we also have $aI_{X\ge a} \le X$

which can be translated in $E[aI_{X\ge a}] \le E[x]$

which is true by definition (2). Also with the definition (1)

$E[I_{X\ge a}] \le E[X]/a$

and 

$P(X \ge a)\ \le\ E[X]/a\quad \square$


 vim:ts=8:noet:ft=help:norl:
