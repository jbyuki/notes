Monte carlo integration
-----------------------

The problem is of the form:

$\int_S f(x)\ ds$

Define:

$\Omega$ => Probability space

$X: \Omega \rightarrow S$

$f: S \rightarrow \mathbb{R}$

$f(X): \Omega \rightarrow \mathbb{R}$

$\mu: S \rightarrow \mathbb{R}^+$ => Probability measure

$\frac{f(X)}{\mu(X)}: \Omega \rightarrow \mathbb{R}$

More interestingly:

$E\left[\frac{f(X)}{\mu(X)}\right] = \int_S f(x)\ ds$

And by the central limit theorem:

$\frac{Y_1 + Y_2 + \dots + Y_N}{N} \rightarrow E\left[\frac{f(X)}{\mu(X)}\right]$ 

as $N$ gets larger where $Y_i ~ \frac{f(X)}{\mu(X)}$

And more interestingly, $\mu$ is free to choose! This is the idea of importance sampling.




