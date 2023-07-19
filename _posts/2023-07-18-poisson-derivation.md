---
title: A Derivation of the Poisson Distribution from the Binomial Distribution
author: tim
date: 2023-07-18 11:33:00 +0800
categories: [non-fiction, math]
tags: [math, poisson distribution, bernoulli, binomial distribution, statistics ]
pin: true
math: true
mermaid: true
image:
  path: /img/poisson-title.jpeg
  alt: AI still can't do math (yet)
---

> Life is good for only two things, discovering mathematics and teaching
> mathematics.
>
> Siméon Denis Poisson


## Pick your Poisson 

Short post today. Here's how to derive the Poisson Distribution from the
Binomial distribution, in the form of a recipe

<center>The Poisson Distribution, mean = $\lambda$ </center>

$$Pr(X = k) =  \frac{\lambda ^ k  e^{-\lambda}}{k!}$$

### Ingredients

1. Knowledge of the [binomial distribution](https://en.wikipedia.org/wiki/Binomial_distribution) 
    * The probability for $k$ out of $n$ successful trials, with each trial having
      a probability $p$ of success, is 

    $${n \choose k} p^k (1-p)^{n-k}$$

2. [Stirling's approximation:](https://en.wikipedia.org/wiki/Stirling%27s_approximation)

$$n! \sim \sqrt{2\pi n} \left( \frac{n}{e} \right)^n $$

which is syntactic sugar for

$$\lim_{n \to \infty} \frac{n!}{\sqrt{2\pi n} \left( \frac{n}{e} \right)^n} = 1$$

### Main Idea

The Poisson distribution answers the following question:

> A radioactive substance is decaying. If, on average, $\lambda$ atoms split every hour,
> what is the probability that $k$ atoms will decay in the next hour?

For example, if on average $5$ atoms split every hour, the probability that $3$
atoms will split is, according to the Poisson distribution,


$$Pr(X = 3) =  \frac{5 ^ 3  e^{-5}}{3!} \approx 14.04\%$$
 
![poisson distribution with mean 5](/img/poisson-mean5.png)
_Poisson distribution with mean = 5. Notice that the largest probability also occurs at 5_

There are a couple of hidden assumptions at play:

1. The events occur independently: i.e. the splitting of one atom does not
   "cause" another atom to split
     - In this model scenario, this assumption can be argued to be false. This
       is because when an atom splits in radioactive decay, it can hit another
       atom which can cause it to split. We wouldn't have to worry about this if
       the density, the amount of radioactive substance, or the rate of decay is
       low enough.
       
2. An atom can decay at any time. In other words, given two different small slices of
   time, an atom is equally likely to split in both slices of time.
      - For example, we cannot use the Poisson distribution to model the
        probability that you will hit $3$ red lights on your way to work if you
        have $5$ stoplights in between. This is because you cannot be stopped
        between stoplights. In this case, a binomial distribution is more
        appropriate.
        
The derivation will make extensive use of this second assumption

### The Derivation

Suppose, on average, $\lambda$ atoms split every hour. That means
$\frac{\lambda}{2}$ atoms split every half hour, $\frac{\lambda}{4}$ atoms split
every $15$ minutes, and so on and so forth. Generalizing this, 

$$\text{the probability that an atom will split every } \frac{1}{n} \text{ hours is }\frac{\lambda}{n}$$

(This is assuming $n$ is sufficiently large so $\frac{\lambda}{n}$ is a
probability. i.e. $n \gg \lambda$)

> In a way, the Poisson distribution is like throwing $n$ coins with each coin
> showing heads with a probability $\frac{\lambda}{n}$ and measuring the
> probability that $k$ coins will show heads, as $n$ tends to infinity!
{: .prompt-tip }

From this, we can work on the formula of the Poisson distribution from the
binomial distribution with probability of success $\frac{\lambda}{n}$

$$P(X = k) = \lim_{n \to \infty} {n \choose k} \left( \frac{\lambda}{n}\right)^k
\left(1 - \frac{\lambda}{n}\right)^{n-k}$$

--- 

We know that, from Stirling's approximation

$$n! \sim \sqrt{2\pi n} \left( \frac{n}{e} \right)^n $$

How can we use this to find an asymptotic for the binomial coefficient $n
\choose k$ ? We can replace all but the $k!$ instance of a factorial with
Stirling's approximation:

$$
\begin{aligned}
{n \choose k} = \frac{n!}{k! (n-k)!} &\sim
\frac{\sqrt{2\pi n} \left( \frac{n}{e} \right)^n}
{k! \sqrt{2\pi (n-k)} \left( \frac{n-k}{e} \right)^{n-k}} \\
&=\lim_{n \to \infty}\frac{1}{k!} \frac{ n^n (n-k)^k\sqrt{n}}{e^k (n-k)^n\sqrt{n - k} } \\
&=\lim_{n \to \infty}\frac{1}{k!} \frac{e^{-k}}{\left( 1 - \frac{k}{n}\right)^n}
n^k\left(1 - \frac{k}{n}\right)^k & \text{here we use } \lim_{n \to \infty}\left(1 -
\frac{k}{n}\right)^n = e^{-k}\\
&=\frac{n^k }{k!}
\end{aligned}
$$

therefore, asymptotically,

$$
{n \choose k } \sim \frac{n^k}{k!} 
$$

we apply this:

$$
\begin{aligned}
P(X = k) &= \lim_{n \to \infty} {n \choose k} \left( \frac{\lambda}{n}\right)^k
\left(1 - \frac{\lambda}{n}\right)^{n-k} \\
&= \lim_{n \to \infty} \frac{n^k}{k!} \left( \frac{\lambda}{n}\right)^k
\left(1 - \frac{\lambda}{n}\right)^{n-k}\\
&= \lim_{n \to \infty} \frac{\lambda^k}{k!} \left(1 - \frac{\lambda}{n}\right)^{n-k}\\
&= \lim_{n \to \infty} \frac{\lambda^k}{k!} \left(1 -
\frac{\lambda}{n}\right)^n\left(1 - \frac{\lambda}{n}\right)^{-k} \\
&=  \frac{\lambda^k\ e^{-\lambda}}{k!}
\end{aligned}
$$

--- 

There you have it! A complete derivation of the Poisson distribution from the
Binomial distribution and Stirling's approximation. I think the part where you
think about the Poisson distribution being an infinite amount of coin tosses is
pretty interesting.
