<!-- 
.. title: Interpolations
.. slug: interpolations
.. date: 2015-08-11 13:47:04 UTC-07:00
.. tags: mathjax, interpolation, functions
.. category: 
.. link: 
.. description: 
.. type: text
-->

Interpolation is a great technique to convert discrete data into continuous data.  It's useful if you want to sample the domain at points other than the original data, or make continuous plots, or do operations convenient on a function (like integration or differentiation).  I'll talk about two types of interpolation of 1-D functions: the standard cubic spline and the less-known Steffen interpolation.  Along the way we'll encounter interesting relations in linear algebra and for the Lucas sequence (a generalization of the Fibonacci sequence).

<!-- TEASER_END -->

It's useful to distinguish interpolation from regression -- interpolation provides a functional form passing through the original data, and uses a standard functional form (e.g., a cubic polynomial) that is ideally a sufficient approximation to the entire domain, while regression fits a model to the data.  In regression, the model is often specialized to the case at hand and usually doesn't pass through the original data points.  If you have a candidate model for the data, you should use regression and not interpolation.  Or, if your data is very noisy, interpolation is usually not a great technique (as we'll see later).

Here's a case where I use interpolation in my own work.  Let's say I have a smooth function $f(x)$ which I want to evaluate over some compact interval.  Suppose $ f(x) \equiv \int_a^b dy \, g(x, y) $, where $ g(x, y) $ is a very complex function and the integral can only be done numerically.  If I want to avoid doing this integral every time I evaluate $ f $, I can tabulate the function over a large number of points (once and for all) and interpolate them to get any $ f(x) $.  I can evaluate the numerical error from interpolation, and ensure that this error falls below the accuracy I need.

Getting to the interpolations, both the standard cubic spline and the Steffen interpolation use cubic polynomials to interpolate between data points, where the polynomial coefficients are determined by boundary conditions at the data points.  The [Steffen interpolation](http://adsabs.harvard.edu/full/1990A%26A...239..443S "M. Steffen, Astron. Astrophys. 239, 443-450 (1990)") has the nice feature that it is monotonic between data points, which avoids wild excursions of the interpolation that can happen with other methods.  Many other types of interpolation exist, and frameworks such as gsl or scipy have rather broad implementations of interpolation routines (with 2D interpolations in scipy or available as an extension in gsl).  I use 2D cubic (or bicubic) interpolation a lot in my own work.

Suppose we have a set of points $\{(x_0 , y_0) ,\, \ldots ,\, (x_n, y_n) \}$ to interpolate.  We define a set of cubic polynomials $\{ g_0 (x), \, \ldots ,\, g_{n-1} (x) \}$, where

$$
g_k (x) = c_k^{(0)} + c_k^{(1)} (x - x_k) + c_k^{(2)} (x - x_k)^2 + c_k^{(3)} (x - x_k)^3 \text{ covers the domain } [x_k, x_{k+1}].
$$

There are $4n$ coefficients to determine.  These can be expressed in terms of the values ($y_k$) and derivatives ($y_k'$) at each of the data points via the linear system

$$
\begin{align}
g_k (x_k) &= c_k^{(0)} = y_k , \\\
g_k' (x_k) &= c_k^{(1)} = y_k' , \\\
g_k (x_{k+1}) &= c_k^{(0)} + c_k^{(1)} d_k + c_k^{(2)} d_k^2 + c_k^{(3)} d_k^3 = y_{k+1} , \\\
g_k' (x_{k+1}) &= c_k^{(1)} + 2 c_k^{(2)} d_k + 3 c_k^{(3)} d_k^2 = y_{k+1}' , \\\
\end{align}
$$

where $d_k \equiv x_{k+1} - x_k$.  We also define $m_k \equiv (y_{k+1} - y_k) / d_k$.  Solving yields

$$
\begin{align}
c_k^{(0)} &= y_k , \\\
c_k^{(1)} &= y_k' , \\\
c_k^{(2)} &= \frac{1}{d_k} (3m_k - 2y_k' - y_{k+1}') , \\\
c_k^{(3)} &= \frac{1}{d_k^2}(-2m_k + y_k' + y_{k+1}') .
\end{align}
$$

Enforcing that the interpolation passes through the data points (and hence requiring continuity) has removed $2n$ variables, and enforcing continuity of the first derivative has further removed $n-1$ variables.  The $n+1$ remaining variables are the $n+1$ derivatives $y_k'$ of the interpolation at each data point.

At this point, different interpolation schemes make different choices to fix the remaining variables.  Let's talk about these in turn, starting with the usual cubic spline.

# <span style="color:blue">Cubic spline</span>

The standard cubic spline makes an obvious choice: enforcing the interpolation is $C_2$, plus boundary conditions at the endpoint:

* Continuity of the second derivative: $g_k'' (x_{k+1}) = g_{k+1}'' (x_{k+1})$. <span style="color:red">[$n - 1$ conditions]</span>
* Vanishing second derivative at the boundary: $g_0''(x_0) = 0$, $g_{n-1}''(x_n) = 0$ <span style="color:red">[2 conditions]</span>

The endpoint boundary conditions could be chosen differently, and it's useful to consider the case where we ask that the first derivatives vanish instead of the second:

* Vanishing first derivative at the boundary, ($g_0'(x_0) = 0$, $g_{n-1}'(x_n) = 0$) <span style="color:red">[2 conditions]</span>

The second derivatives are

$$
\begin{align}
g_k'' (x_{k+1}) &= 2 c_k^{(2)} + 6 c_k^{(3)} d_k , \\\
g_{k+1}'' (x_{k+1}) &= 2 c_{k+1}^{(2)} ,
\end{align}
$$

so that continuity implies

$$
g_k'' (x_{k+1}) = g_{k+1}'' (x_{k+1}) \; \Rightarrow \; d_{k+1} y_k' + 2(d_k + d_{k+1}) y_{k+1}' + d_k y_{k+2}' = 3(d_k m_{k+1} + d_{k+1} m_k) ,
$$

while a vanishing second derivative at the endpoint gives the conditions

$$
\begin{align}
g_0'' (x_0) &= 0 \; \Rightarrow \; 2 y_0' + y_1' = 3 m_0 , \\\
g_{n-1}'' (x_n) &= 0 \; \Rightarrow \; 2 y_n' + y_{n-1}' = 3 m_{n-1} .
\end{align}
$$

This leads to a linear system which can be represented as

$$
\left(\begin{matrix}
2d_0 & d_0 & 0 & 0 & 0 & \ldots & 0 \\\ 
d_1 & 2(d_1 + d_0) & d_0 & 0 & 0 & \ldots & 0 \\\
0 & d_2 & 2(d_2 + d_1) & d_1 & 0 & \ldots & 0 \\\
 & & & \ddots & & & \\\
0 & \ldots & 0 & d_{n-2} & 2(d_{n-2} + d_{n-3}) & d_{n-3} & 0 \\\
0 & \ldots & 0 & 0 & d_{n-1} & 2(d_{n-1} + d_{n-2}) & d_{n-2} \\\
0 & \ldots & 0 & 0 & 0 & d_{n-1} & 2 d_{n-1} \end{matrix} \right)
\left( \begin{matrix}
y_0' \\\
y_1' \\\
y_2' \\\
\vdots \\\
y_{n-2}' \\\
y_{n-1}' \\\
y_n'
\end{matrix} \right) = \left( \begin{matrix}
3 d_0 m_0 \\\
3(d_0 m_1 + d_1 m_0) \\\
3(d_1 m_2 + d_2 m_1) \\\
\vdots \\\
3(d_{n-3} m_{n-2} + d_{n-2} m_{n-3}) \\\
3(d_{n-2} m_{n-1} + d_{n-1} m_{n-2}) \\\
3(d_{n-1} m_{n-1})
\end{matrix} \right) .
$$

The matrix is tridiagonal, and by inverting it we obtain the derivatives at each point and can define the cubic spline.  The form becomes far simpler if the $x$ points are equidistant, so that the $d_k$ scale out:

$$
\left(\begin{matrix}
2 & 1 & 0 & 0 & 0 & \ldots & 0 \\\
1 & 4 & 1 & 0 & 0 & \ldots & 0 \\\
0 & 1 & 4 & 1 & 0 & \ldots & 0 \\\
 & & & \ddots & & & \\\
 0 & \ldots & 0 & 1 & 4 & 1 & 0 \\\
 0 & \ldots & 0 & 0 & 1 & 4 & 1 \\\
 0 & \ldots & 0 & 0 & 0 & 1 & 2 \end{matrix} \right) 
\left( \begin{matrix} 
y_0' \\\
y_1' \\\
y_2' \\\
\vdots \\\
y_{n-2}' \\\
y_{n-1}' \\\
y_n'
\end{matrix} \right) = \left( \begin{matrix} 
3 m_0 \\\
3(m_1 + m_0) \\\
3(m_2 + m_1) \\\
\vdots \\\
3(m_{n-2} + m_{n-3}) \\\
3(m_{n-1} + m_{n-2}) \\\
3(m_{n-1}) 
\end{matrix} \right) .
$$

If we instead chose vanishing first derivatives and the endpoints, we would just have $y_0' = y_n' = 0$, and our linear system would become (for equidistant $x_k$)

$$
\left(\begin{matrix} 
4 & 1 & 0 & 0 & 0 & \ldots & 0 \\\
1 & 4 & 1 & 0 & 0 & \ldots & 0 \\\
0 & 1 & 4 & 1 & 0 & \ldots & 0 \\\
 & & & \ddots & & & \\\
 0 & \ldots & 0 & 1 & 4 & 1 & 0 \\\
 0 & \ldots & 0 & 0 & 1 & 4 & 1 \\\
 0 & \ldots & 0 & 0 & 0 & 1 & 4 
 \end{matrix} \right) \left( \begin{matrix} 
 y_1' \\\
 y_2' \\\
 y_3' \\\
 \vdots \\\
 y_{n-3}' \\\
 y_{n-2}' \\\
 y_{n-1}' 
 \end{matrix} \right) = \left( \begin{matrix} 
 3(y_2 - y_0) \\\
 3(y_3 - y_1) \\\
 3(y_4 - y_2) \\\
 \vdots \\\
 3(y_{n-2} - y_{n-4}) \\\
 3(y_{n-1} - y_{n-3}) \\\
 3(y_n - y_{n-2}) 
 \end{matrix} \right) ,
 $$

which is also tridiagonal.  At this point, we only need to invert the matrix to solve for the derivatives and calculate the function.  It may be surprising, but the derivatives will generally depend on the values of all of the data points -- so your interpolation can be adversely affected by including garbage data at the ends of the domain.  Before moving on to the Steffen interpolation I'll talk about inverting the tridiagonal matrices here, which is interesting on its own.

# <span style="color:blue">Inverting tridiagonal matrices </span>

Guess what? Inverting tridiagonal matrices is a lot of fun!  There's a pretty slick form for the inverse that runs into some interesting sequences (see [this paper](http://www.sciencedirect.com/science/article/pii/0024379594904146 "R. Usmani, Comput. Math. Appl. 212/213, 413-414 (1994)")).

For a tridiagonal matrix of the form

$$
T = \left( \begin{matrix}
a_1 & b_1 & & & \\\
c_1 & a_2 & b_2 & & \\\
& c_2 & \ddots & \ddots & \\\
& & \ddots & \ddots & b_{n-1} \\\
& & & c_{n-1} & a_n 
\end{matrix} \right) ,
$$

the inverse $T^{-1}$ has entries

$$
T_{ij}^{ -1 } = \begin{cases} 
(-1)^{i+j} b_i \cdots b_{j-1} \theta_{i-1} \phi_{j+1} / \theta_n & i \le j \,, \\\ 
(-1)^{i+j} c_j \cdots c_{i-1} \theta_{j-1} \phi_{i+1} / \theta_n & i > j 
\end{cases}
$$

where $\theta_k$ and $\phi_k$ satisfy the recursion relations

$$
\theta_k = a_k \theta_{k-1} - b_{k-1} c_{k-1} \theta_{k-2} ,\, k = 2, \ldots, n \text{ with } \theta_0 = 1 ,\, \theta_1 = a_1 , \\\
\phi_k = a_k \phi_{k+1} - b_k c_k \phi_{k+2} ,\, k = n-1, \ldots, 1 \text{ with } \phi_{n+1} = 1,\, \phi_n = a_n .
$$

Note that if (as in our case), $b_k = c_k = 1$, we have a much simpler form:

$$
T_{ij}^{ -1 } = \begin{cases} 
(-1)^{i+j} \theta_{i-1} \phi_{j+1} / \theta_n & i \le j \,, \\\
(-1)^{i+j} \theta_{j-1} \phi_{i+1} / \theta_n & i > j 
\end{cases}
$$

with recursion relations

$$
\begin{align}
\theta_k &= a_k \theta_{k-1} - \theta_{k-2} ,\, k = 2, \ldots, n \text{ with } \theta_0 = 1 ,\, \theta_1 = a_1 , \\\
\phi_k &= a_k \phi_{k+1} - \phi_{k+2} ,\, k = n-1, \ldots, 1 \text{ with } \phi_{n+1} = 1,\, \phi_n = a_n .
\end{align}
$$

This further simplifies if the $a_k$ are constant; in this case the recurrence relation becomes a Lucas sequence $U(a_k, 1)$ (as in the case of the modified cubic spline, where $a_k = 4$).  Let's take another side track and prove a nice identity for the Lucas sequence $U(P, 1)$ that seems to be not well known.

### <span style="color:blue">The Lucas sequence $U(P, 1)$</span>

We're discussing a special case of the Lucas sequence $U(P, Q)$, setting $Q = 1$.  Let's use the notation $U_n$ as the $n^{\rm th}$ term of $U(P, 1)$.  The closed form expression for $U_n$ is given by

$$
U_n = \frac{1}{\sqrt D} (a^n - b^n), \text{ where } D = P^2 - 4,\, a = \frac12(P + \sqrt{D}), \text{ and } b = \frac12(P - \sqrt{D}).
$$

We will show $U_n^2 = U_{n+k} U_{n-k} + U_k^2$ for any natural numbers $n$ and $k$.  We will make use of the defining recursion relation for $U_n:$

$$
U_n = P U_{n-1} - U_{n-2}, \text{ where } U_1 = 1 \text{ and } U_2 = P.
$$

First, we need to prove another relation, $U_{n+k} = U_n U_{k+1} - U_{n-1} U_k,$ which we do by induction on $k$.  For $k = 1$, we recover the defining recursion relation above.  Assuming the case for $k$, for $k+1$ we have

$$
\begin{align}
U_{n+k+1} &= P U_{n+k} - U_{n+k-1} \\\
&= P ( U_n U_{k+1} - U_{n-1} U_k ) - U_n U_k + U_{n-1} U_{k-1} \\\
&= U_n ( P U_{k+1} - U_k ) - U_{n-1} ( P U_k - U_{k-1} ) \\\
&= U_n U_{k+2} - U_{n-1} U_{k+1},
\end{align}
$$ 

which completes the induction.  Now to show the main relation, we take the difference $U_{n+k}^2 - U_{n+2k} U_n$ and apply the relation we have already show to separate the $n$ and $k$ dependence:

$$
\begin{align}
U_{n+k}^2 - U_{n+2k} U_n &= (U_n U_{k+1} - U_{n-1} U_k)^2 - (U_n U_{2k+1} - U_{n-1} U_{2k}) U_n \\\
&= (U_n^2 + U_{n-1}^2) U_k^2 - U_n U_{n-1} U_k (U_{k+1} + U_{k-1}) .
\end{align}
$$

Using $U_{k+1} + U_{k-1} = P U_k,$ (which uses $Q = 1$ for the general Lucas sequence relation) we can put the above into the form $U_k^2 ( U_n^2 - U_{n-1} U_{n+1} ),$ so that if $U_n^2 - U_{n-1} U_{n+1} = 1$, we have proven the relation we set out to (note that we have just shifted the indices by $k$).  This can be done using $U_n$ in terms of $a$ and $b$, noting two things: $a b = \frac14 (P^2 - D) = 1,$ and $a - b = \sqrt{D}.$  Then

$$
\begin{align}
U_n^2 - U_{n-1} U_{n+1} &= \frac{1}{D} \Bigl( (a^n - b^n)^2 - (a^{n-1} - b^{n-1})(a^{n+1} - b^{n+1}) \Bigr) \\\
 &= \frac{1}{D} (ab)^{n-1} (a-b)^2 = 1,
\end{align}
$$

and therefore we have $U_n^2 = U_{n+k} U_{n-k} + U_k^2$!  For the Fibonacci sequence, a very similar relation holds and is called Catalan's identity; we've shown the extension to the general Lucas sequence $U(P, 1)$.

OK, that was an interesting diversion.  Now let's talk about the Steffen interpolation, which doesn't have any matrices to invert!

# <span style="color:blue">Steffen interpolation</span>

The Steffen interpolation was motivated by the fact that many interpolation routines can have undesirable behavior between data points.  One way to think about this (for the case of the cubic spline) is that it "uses up" its degrees of freedom by requiring the interpolation is $C_2$, and so there are no more degrees of freedom left to ensure that the function is at all well-behaved between the data points -- we are just relying on the smoothness of the cubic polynomials to not do anything too crazy.
The Steffen interpolation solves this problem by using the remaining degrees of freedom to ensure that for each segment (cubic function between adjacent data points), the interpolation is monotonic.  It sets the derivatives to values depending on the data in the neighborhood:

$$
y_k' = \begin{cases} 
0 & \text{if } m_k m_{k-1} \le 0, \\\
2a \min(\lvert m_{k-1} \rvert , \lvert m_k \rvert) & \text{if } \lvert p_k \rvert > 2 \min( \lvert m_{k-1} \rvert, \lvert m_k \rvert) \\\
p_k & \text{otherwise,} 
\end{cases}
$$

where

$$
\begin{align}
a &= \text{sign}(m_k) = \text{sign}(m_{k-1}), \\\
p_k &= \dfrac{m_{k-1} d_k + m_k d_{k-1}}{d_{k-1} + d_k}.
\end{align}
$$

For $a$, note that the signs must be equal when the middle condition is met, as the first condition is not satisfied.

The first case for $y_k'$ is obvious -- enforcing monotonicity of the interpolation between data points requires that any local extremum has to be at an original data point, so that if the derivative is undergoing a sign change between $x_{k-1}$ and $x_{k+1}$ (signaled by $m_{k-1}$ and $m_k$ having different signs) it has to be zero at $x_k$.  The fact that local extrema can only exist at the original data points can be seen as a feature/bug of the Steffen interpolation, depending on your point of view.  Personally, I like it because I often deal with applications where I want the interpolating function to be well-behaved over the whole domain more than I want it to be $C_2$ (although that does have its advantages in applications where the derivative of the function is important, as it will have kinks where the second derivative fails to be continuous).

The other cases defining $y_k'$ set a maximum absolute value for the slope.  In fact, there is an interesting relation to the standard cubic spline.  Recall the relation defining the derivatives $y_k'$,

$$
d_{k+1} y_k' + 2(d_k + d_{k+1}) y_{k+1}' + d_k y_{k+2}' = 3(d_k m_{k+1} + d_{k+1} m_k).
$$

If we set $y_k' = y_{k+1}' = y_{k+2}' = p_{k+1}$, then we recover the defining relation for $p_{k+1}$.  The Steffen interpolation has a maximum between this value and twice the (pointwise-difference) slope of adjacent points.


