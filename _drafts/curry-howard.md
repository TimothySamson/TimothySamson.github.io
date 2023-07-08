---
title: Curry - Howard - Lambek and the Isomorphism 
author: tim
date: 2023-07-06 11:33:00 +0800
categories: [non-fiction, programming]
tags: [haskell, dependent type theory, math, programming, curry howard isomorphism, proof systems]
pin: true
math: true
mermaid: true
image:
  path: /img/turtles-recursion.jpeg
  alt: It's turtles all the way down. Certified GPT-Free
---

> God, inspired by the barber paradox, wanted to erase the existence of
> turtles by plotting a paradox. "Turtles cannot exist!" he said, "because who then
> shaves the barber turtle's head?" All the turtles synchronously blinked once,
> and moved on with their lives. The omniscient god saw his mistake at once;
> turtles don't have hair.

I've recently delved into a deep dive into exploring the Curry - Howard - Lambek 
isomorphism and I want to share my insights. I have played around with the Lean
theorem prover for a bit and got semi-good at formally proving mathematical
theorems in Lean. Theorem proving using formal verifiers can be very fun! But I had
this nagging feeling of not knowing what is going on under the hood. This
article is an attempt to fix that so we can explore what it
takes to build a theorem prover from the ground up

This article is about using the Curry - Howard 
isomorphism to build a proof verifier leveraging Haskell's type system from
scratch. I want to use as few language extensions as possible, only opting for
it if it increases convenience and if it is truly necessary. The goal
is to build Peano arithmetic into Haskell's type system and to generate proofs 
about arithmetic. Along the way, we'll get to see cool turtles.

## The Isomorphism

> Let it be said at once that there is no unanimity on the subject. This is a
> healthy situation, for each point of view suggests problems and methods
> which the others do not.
>
> Haskell Curry, *Foundations of Mathematical Logic*

### Types and Functions

Let's consider 3 types. 

The first one is the `Void` type. No object in the world
has this type, making it awkward (also impossible) to reference and/or produce an object of this
type. 

The other type is `()`. Only one object in the world can ever have the
type, namely `()`. 

The last is type `Bool`. Only two objects, named `True` and `False`,
can have the type `Bool`.

![Desktop View](/img/turtles-isom.jpeg) 
_a one-to-one correspondence between turtles and sets_
The next thing to consider is functions. How many can there be from one type to
another? Let's start with functions with type `Bool` as the argument. Here are
some examples:

```haskell
not :: Bool -> Bool 
not True = False
not False = True

cons :: Bool -> ()
cons _ = ()

```

How can we have a function which accepts a `Void` and returns `Bool`? There are
no objects with type `Void`. In the requirement for a function, every object
in the domain there is a unique mapping to an object in the range / codomain.
But what if there is no domain? In Haskell, the `absurd` (defined in
`Data.Void`) has the type signature

```haskell
`{-# LANGUAGE ExplicitForAll #-}`
absurd :: forall a. Void -> a
```

in other words, for any type `a`, there is one and only one function from `Void`
to `a`, namely `absurd`. 


| Function Type Signature | Number of functions | Functions                              |
|:------------------------|:--------------------|:---------------------------------------|
| `Void -> Void`          | 1                   | `id`                                   |
| `forall a. a -> Void`   | 0                   | None                                   |
| `forall a. Void -> a`   | 1                   | `absurd`                               |
| `Bool -> ()`            | 1                   | `_ -> ()`                              |
| `() -> ()`              | 1                   | `id`                                   |
| `() -> Bool`            | 2                   | `() -> True`, `() -> False`            |
| `Bool -> Bool`          | 4                   | `not`, `_ -> True`, `_ -> False`, `id` |
|                         |                     |                                        |

With some time to consider, you'll see that for sets $A, B$ with $m, n$ elements
respectively, the number of functions from `A` to `B` is 

$$n^m$$ 

This is one of the reasons why functions are elements of what is called an
exponential type. It is common in mathematics to denote the set of functions 
$A \rightarrow B$ by $B ^ A$. In this sense, functions in Haskell have their own type

### Sum types and Product types

How can we combine types? The product type is also known as the tuple type in
Haskell and is defined by two functions, `fst` and `snd`, which returns the left or right
object in the tuple

<!-- https://q.uiver.app/#q=WzAsMyxbMSwwLCIoYSwgYikiXSxbMCwxLCJhIl0sWzIsMSwiYiJdLFswLDEsIlxcdGV4dHtmc3R9IChhLCBiKSIsMl0sWzAsMiwiXFx0ZXh0e3NuZH0gKGEsYikiXV0= -->
<iframe class="quiver-embed" src="https://q.uiver.app/#q=WzAsMyxbMSwwLCIoYSwgYikiXSxbMCwxLCJhIl0sWzIsMSwiYiJdLFswLDEsIlxcdGV4dHtmc3R9IChhLCBiKSIsMl0sWzAsMiwiXFx0ZXh0e3NuZH0gKGEsYikiXV0=&embed" width="437" height="304" style="border-radius: 8px; border: none;"></iframe>
On the other hand, an example of a sum type is Haskell's `Either` type
<!-- https://q.uiver.app/#q=WzAsMyxbMSwwLCJcXHRleHQge0VpdGhlciBhIGJ9Il0sWzIsMSwiXFx0ZXh0e2J9Il0sWzAsMSwiXFx0ZXh0e2F9Il0sWzEsMCwiXFx0ZXh0e1JpZ2h0IGJ9IiwyXSxbMiwwLCJcXHRleHR7TGVmdCBhfSJdXQ== -->
<iframe class="quiver-embed" src="https://q.uiver.app/#q=WzAsMyxbMSwwLCJcXHRleHQge0VpdGhlciBhIGJ9Il0sWzIsMSwiXFx0ZXh0e2J9Il0sWzAsMSwiXFx0ZXh0e2F9Il0sWzEsMCwiXFx0ZXh0e1JpZ2h0IGJ9IiwyXSxbMiwwLCJcXHRleHR7TGVmdCBhfSJdXQ==&embed" width="503" height="304" style="border-radius: 8px; border: none;"></iframe>


The `Either` type is used to represent types that may come from the `Left` type
or the `Right` type. The `Maybe` type is also a special case of an `Either`
type. As an example, division with floats yield a `Maybe Float` because division
by zero yields a `Nothing`

```haskell
(/:) :: Float -> Float -> Maybe Float
a /: 0 = Nothing
a /: b = Just (a / b)
```

Using product, sum and exponential types, we can create a diverse array of types
and interpret its structure using the other side of the isomorphism

### The other side of the isomorphism

That's it for `types`. How does this relate to `propositions`?

```mermaid
flowchart TD;
  subgraph curry-howard
  types ---> propositions
  propositions ---> types
  end

```

Curry - Howard - Lambek noticed the similarity between the type system we
created above to the mechanisms of classical logic. To start with the
isomorphism, we will say that `()` is `True` and `Void` is `False`

```haskell
type False = Void
type True = ()
```

A proposition `P` is mapped to a type in our system. Here's the important bit

> `P` is true if it's corresponding type is non-empty, and it is false if its
> corresponding type is empty. 
{: .prompt-tip }


Here is a table of the corresponence 

| Types                      | Propositions |
|:---------------------------|:-------------|
| `Void`                     | False        |
| `()`                       | True         |
| `(a, b)`                   | a and b      |
| `Either a b`               | a or b       |
| Exponential type: `a -> b` | a implies b  |


```haskell

-- Proposition land -> Type land
type (:|) = Either 
type (:&) = (,)
-- We use Haskell's default `->` to denote implication
```


There is another way to denote a false proposition. Remember from the table in
the previous section that for any type `a` there are zero functions of type `a -> Void`.

| Function Type Signature | Number of functions | Functions |
|:------------------------|:--------------------|:----------|
| `Void -> Void`        | 1                   | `id`      |
| `forall a. a -> Void`   | 0                   | None      |
| ...                     | ...                 | ...       |

So, what if we are able to construct a function of type `a -> Void`? That would be
impossible, *unless* `a` itself was `Void`! That means that the existence of a
function of type `a -> Void` means `a` is uninhabited. 

This way, we can define that a proposition `P` is false if there is a function
of type `P -> Void`


```haskell
type Not a = a -> False
```

Putting it all together, here is our logical universe so far:

```haskell
-- Type operators allow types which are also infix operators
{-# LANGUAGE TypeOperators #-}
type False = Void
type True = ()
type Not a = a -> False
type (:|) = Either 

-- makes (:&) a product type
data a :& b= a :& b deriving Show
left :: (a :& b) -> a
left (a :& b) = a

right :: (a :& b) -> b
right (a :& b) = b

-- We use Haskell's default `->` to denote implication

-- Type variables
type P = ()
type Q = ()
type R = ()

```

Let's mess around a little bit

```haskell
ghci> a = () :& () :: P :& Q

ghci> :t a
a :: P :& Q

ghci> :t left 
left :: (a :& b) -> a

ghci> :t left a
left a :: P

ghci> :t right a
right a :: Q

ghci> b = Left () :: P :| R

ghci> :t b
b :: P :| R

```

### *Mise en place*

We now have all out ingredients out in the table. Let's stop here and have some
fun. Despite our simplistic setup (7 type synonyms, a data declaration, and 2 functions) 
we have at our fingertips the entirety of constructive logic. Let's prove a theorem.

--- 
A senator in the USA must be 30 years old. Let us denote 

`P = being a senator in the united states`

`Q = being at least 30 years old`.

Therefore, the implication `P -> Q` is true, so its type is inhabited.

`Not Q` is equivalent to `being younger than 30 years old`. What can we
deduce? If someone is younger than 30 years old, it's impossible for them to
be a senator. In other words, `Not Q -> Not P`. Putting it all together,
 we have that `P -> Q` implies `Not Q -> Not P`, or that a function
 
 ```haskell
 contrapositive :: (p -> q) -> (Not q -> Not p)
 ```

exists with this certain type, where `p` and `q` are arbitrary types

---

  * [ ] This is another level to our isomorphism. If propositions are types, that
means proofs of propositions are functions between types!

```mermaid
graph LR;
  subgraph ide2 [type level]
    d-->c
    c[types]-->d[propositions]
  end
  subgraph ide1 [element level]
    b[proofs]-->a[function application]
    a-->b
  end
  
  ide1 ---> ide2
  
  
```

How can we construct this type? `contrapositive` takes in a function and returns
another function. We can simplify the type signature by noticing that `Not q` is
actually `q -> False` in disguise. Unfolding our definition of `Not`, we get that
`contrapositive` should have a type signature of 

 ```haskell
contrapositive :: (p -> q) -> ((q -> False) -> ( p -> False))
 ```

> `(->)` is not associative but actually right associative, which means that
> 
> `a -> b -> c` is evaluated as `(a -> (b -> c))` which is not equal to `((a ->
> b) -> c)`
{: .prompt-warning }

because of the right - associativity of `->`, our type signature simplifies to
 ```haskell
contrapositive :: (p -> q) -> (q -> False) ->  p -> False
 ```

so our `contrapositive` function takes in three arguments: 
1. `pq :: p -> q`
2. `nq :: q -> False`
3. `p :: p`

Our goal is to produce a `False`. Stop here for a couple minutes and try to
figure out how to return a `False` from these 3 functions. Or, you could just
admire these turtles for a little bit

![Desktop View](/img/turtles-abstraction.jpeg) 
_above there are propositions, below there are types_


Welcome Back

Here's one way we can implement this. We first apply modus ponens to `p` and
`pq`, and apply the result of that computation to `nq`

```haskell
contrapositive :: (p -> q) -> (q -> False) ->  p -> False
contrapositive pq nq p  = nq . pq $ p
```

or more succinctly, using currying and using the `Not` type synonym

```haskell
contrapositive :: (p -> q) -> (Not q -> Not p)
contrapositive pq nq  = nq . pq
```

We have now proved our first theorem in Haskell! It compiles!
That's one of the features of type-level programming. In
regular programming, we are not sure if our program is correct even if the
compiler accepts our program. In type level programming, we rejoice when the
compiler accepts our offering


## Constructive logic

![Desktop View](/img/turtles-deconstruct.jpeg) 
_constructive logic_

Let's speedrun constructive logic with me! I will offer some function type
signatures, and our goal is to "prove" the proposition by constructing a
function with this type. Let's start with `cases`

### Cases

> "But one false statement was made by Barrymore at the inquest. He said that there were no traces upon the ground round the body. He did not observe any. But I did—some little distance off, but fresh and clear."
> 
> "Footprints?"
> 
> "Footprints."
> 
> "A man's or a woman's?"
> 
> Dr. Mortimer looked strangely at us for an instant, and his voice sank almost to a whisper as he answered:
> 
> "Mr. Holmes, they were the footprints of a gigantic hound!"
> 
> Conan Doyle, *The Hounds of Baskerville*

Sherlock Holmes arrives at the body of a murder victim. He deduces from the
footprints that the perpetrator must be a man's or a woman's. Because of
Sherlock Holmes' impeccable deductive skills, he concludes, because of the
type of the wound on the neck of the deceased, that if the perpetrator was a man,
he must have used a knife. If the perpetrator was a woman, then due to the
delicate nature and angle of the wound, he deduced that the instrument of murder
must have also been a knife.

Sherlock Holmes, adept in "deduction" but ignorant in mathematical logic,
neglected to see that the murder weapon was a knife. 


Let's consider a mathematical question. Can there be two irrational numbers $m,
n$ where $m^n$ is rational? Think about it for a minute.

---

<details><summary>Solution</summary>
Consider $\sqrt{2}$. We know that $\sqrt{2}$ is irrational, so $n = \sqrt{2} ^
\sqrt{2}$ is either rational or irrational. 

<br/>
If $n$ is rational, we have satisfied the conclusion

<br/>
If $n$ is irrational, then 

$$n ^ \sqrt{2} = \sqrt{2} ^ {\sqrt{2} \sqrt 2} = \sqrt{2} ^ 2 = 2$$ 

which is rational
<br/>

Note that, in this proof, we are uncertain whether $n = \sqrt 2 ^ \sqrt 2$ is
rational or not, but it is fine because both ways allow us to satisfy the
conclusion of our problem. It is known, using theory, that $n$ is irrational, but
that is not needed in this proof
</details>

---

Here's how we formalize this. 

$$
\begin{aligned}
a\ |\ b \\
a \implies c \\
b \implies c \\
\hline
c
\end {aligned}
$$

```haskell
-- also known as `or elimination`
cases :: (a :| b) -> (a -> c) -> (b -> c) -> c
cases (Left a) (ac) _  = ac a
cases (Right b) _ (bc) = bc b
```
Our logical universe is starting to look more cozy with all these theorems we're creating.

### Or introduction

Sherlock, after a course in first-order logic, have arrived at the conclusion
that the murder weapon was a knife. Inspired by his new knowledge of logic, he
concluded that the murder weapon was either a knife or a gun. He thought about
it for a second and realized that, although logically true, it was a logical
dead end.

Here's the logic written down mathematically:


$$
\begin{aligned}
a\\
\hline
a \text{  or  }  b
\end {aligned}
$$

This is leveraging the fact that `or` in mathematics is an inclusive or. Both
sides of `or` can be true.

```haskell
-- left or introduction
lorintro :: a -> a :| b
lorintro a = Left a
```

Elementary, my dear watson.

### Symmetry, `and` introduction and elimination

We already have the tools for **and introduction**: namely, our data constructor
`:&`. 

**and elimination** can be implemented using

```haskell
-- left and elimination
landelim :: a :& b -> a
landelim (a :& b) = a
```

This only eliminates the left part of the `:&` and `:|` type. To eliminate the right
side, we can implement the fact that `and` and `or` are symmetric operators. 

> If I am Tim and I am human, then I am human and I am Tim.
{: .prompt-tip }

```haskell
orsymm :: a :| b -> b :| a
orsymm (Left a)  = Right a
orsymm (Right b) = Left b

andsymm :: a :& b -> b :& a
andsymm (a :& b) = b :& a
```

then, we can implement right **and eliminations** and right **or introductions**
with

```haskell
-- right or introduction
rorintro :: b -> a :| b
rorintro = orsymm . lorintro

-- right and elimination
randelim :: a :& b -> b
randelim = landelim . andsymm
```

`rorintro`, `lorintro`, `randelim`, and `landelim` are all redundant because we
could use
  - `rorintro` = `Left`
  - `lorintro` = `Right`
  - we can use pattern matching instead of `randelim` and `landelim` 
  
so I will use Haskell's tools instead from now on. 

### The third side of the isomorphism

There are two sides to every isomorphism. But the best isomorphisms have three
sides. 

<!-- https://q.uiver.app/#q=WzAsMyxbMiwwLCJcXHRleHR7cHJvcG9zaXRpb25zfSJdLFswLDIsIlxcdGV4dHt0eXBlc30iXSxbNCwyLCJcXHRleHR7YWxnZWJyYX0iXSxbMCwyLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XSxbMiwxLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XSxbMSwwLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XV0= -->
<iframe class="quiver-embed" src="https://q.uiver.app/#q=WzAsMyxbMiwwLCJcXHRleHR7cHJvcG9zaXRpb25zfSJdLFswLDIsIlxcdGV4dHt0eXBlc30iXSxbNCwyLCJcXHRleHR7YWxnZWJyYX0iXSxbMCwyLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XSxbMiwxLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XSxbMSwwLCIiLDAseyJzdHlsZSI6eyJ0YWlsIjp7Im5hbWUiOiJhcnJvd2hlYWQifX19XV0=&embed" width="836" height="432" style="border-radius: 8px; border: none;"></iframe>

The third side is algebra. We called our three type operators sum, product, and
exponential. Could they possibly correspond to the sum, product, and exponential
we are used to in algebra?

| Types                      | Propositions | Algebra     |
|:---------------------------|:-------------|:------------|
| `Void`                     | False        | $0$         |
| `()`                       | True         | $1$         |
| `(a, b)`                   | a and b      | $a \cdot b$ |
| `Either a b`               | a or b       | $a + b$     |
| Exponential type: `a -> b` | a implies b  | $b ^ a$     |


Let's test it out. We know from algebra that

$$a \cdot (b + c) = a \cdot b + a \cdot c$$

putting in in the language of propositions:


$$a\ \&\ (b\ |\ c) = (a\ \&\ b)\ |\ (a\ \&\ c)$$

translating it into proposition land in Haskell, we have

```haskell
--         and  or
andor :: a :&(b :| c) -> (a :& b ) :| (a :& c)
```

is true. This is only one side of the implication; we have to consider the other
side.


```haskell
--             and    or    and
andorand :: (a :& b ) :| (a :& c) -> a :& (b :| c)
```
 
Here are the proofs:

```haskell
andor :: a :&(b :| c) -> (a :& b ) :| (a :& c)
andor (a :& (Left b))  = Left (a :& b)
andor (a :& (Right c)) = Right (a :& c)


andorand :: (a :& b ) :| (a :& c) -> a :& (b :| c)
andorand (Left  (a :& b)) = a :& Left b
andorand (Right (a :& c)) = a :& Right c
```

Hold on. Here's another theorem in logic:

$$a\ |\ (b\ \&\ c) = (a\ |\ b)\ \&\ (a\ |\ c)$$

which, using the isomorphism, translates in algebra to 

$$a + b \cdot c = (a + b) \cdot (a + c)$$

That is not true in regular algebra! How can we explain this? Well, since `a`, `b`, and `c`
are ultimately propositions, they are booleans and follow boolean arithmetic:


| $a$ | $b$ | $a + b$ | $a \cdot b$ |
|-----|-----|---------|-------------|
| 0   | 0   | 0       | 0           |
| 0   | 1   | 1       | 0           |
| 1   | 0   | 1       | 0           |
| 1   | 1   | 1       | 1           |

$$\begin{aligned}
(a + b) \cdot (a + c) &= a \cdot a + a\cdot c + b \cdot a + b \cdot c &
\text{distributive property}\\
                      &= a \cdot 1 + a\cdot c + b \cdot a + b \cdot c & \quad a \cdot
                      a = a = a \cdot 1 \\ 
                      &= a \cdot (1 +  c + b)  + b \cdot c & \quad 1 + x = 1\\
                      &= a + b \cdot c
\end{aligned}$$

Here is the proof of both sides

```haskell
--         or    and
orand :: a :| (b :& c) -> (a :| b) :& (a :| c)
orand (Left a) = Left a :& Left a
orand (Right (b :& c)) = Right b :& Right c

--            or    and   or
orandor :: (a :| b) :& (a :| c) -> a :| (b :& c)
orandor (Left a :& _)        = Left a
orandor (_ :& Left a)        = Left a
orandor (Right b :& Right c) = Right (b :& c)
```

This isomorphism in algebra is more profound when considering exponential types.
More on that maybe later.
