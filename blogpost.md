<script type="text/javascript"> window.MathJax = { tex: { inlineMath: [['$', '$'], ['\\(', '\\)']], displayMath: [['$$', '$$'], ['\\[', '\\]']] }, svg: { fontCache: 'global' } }; </script> <script type="text/javascript" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"> </script>

# Nice Code Bro, Too Bad It's Just a Wrapper Over Math: Solving Leetcode in Lambda Calculus


## Background and motivation

Like any other field of formal logic, the formal study of lambda calculus is littered, and for good reason, with involved formal definitions. These are very important, but also difficult to digest. This blog post is not an academic resource, hence, I opt for an intuitive natural language explanation. To maintain disambiguity and completeness I will link all the involved discussions, and definitions are the end in an appendix of sorts. Forgive me for this math sacrilige. 


## Introduction and definitions


$$
\lambda x.x
$$

here $\lambda$ is an operator that denotes a function in the language, the first $x$ denotes the input parameter and the second $x$ denotes the output as a 'function' of the input parameter. For example, a simple function like $f(x)=x+1 $ is written as $$\lambda x.x+1$$ or the polynomial $f(x)=x^2+x+1$ is $$\lambda x.x^2+x+1$$ which is to say that that the $\lambda$ operator allows you to __abstract__ over $x$. To evalute the function:
$$
\begin{align*}
(\lambda x. x^2 + x + 1) \space 2 & \quad \implies \quad 2^2 + 2 + 1 & \quad (\text{substitute for } 2) \\
                            & \quad \implies \quad 7           & \quad (\text{arithmetic})
\end{align*}
$$

Notice that we __substitute__  $2$ for all occurances of $x$ in the output. More formally:
$$
(\lambda x.M)N \implies M[x \coloneqq N]
$$

There is no restriction on the _type_ of $N$ because types dont exist in (untypes) lambda calculus. It couldve been a string, an int, a float, bool, etc. however, lambda calculus does not recognise anything other than a function, so in that sense $N$ is also a lambda. We will see soon how to encode things like integers into lambda functions. For now, take lite.

$M[x \coloneqq N]$ denotes the substituion of $N$ for the free occurances of $x$ in $M$. If you are familiar with the definition of free and bound variables in context of first order logic, then the same intuitive idea applies here: an occurrence of $x$ in $φ$ is free in $φ$ if and only if it is a leaf node in the parse tree of $φ$ such that there is no path upwards from that node $x$ to a node $∀x$ or $∃x$. But of course quanitifers do not exist in $\lambda$ calculus but the $\lambda$ operator is analogous to the logical quantifiers in that it __binds__ the variable appearing right after it (the input parameter) in term $M$ (the output), which is what the quantifiers also do (hence the 'no path upwards...' clause in the definition). Dont worry if it doesn't make sense, in my experience relying on a intuitive understanding of substitution works fine in most cases. 

To maintain disambiguity in our work we will maintain a "_hygeine condition_", i.e., we are only allowed to substitute into a lambda if the replacement term ($N$) does not contain the bound variable ($x$), otherwise we can simply rename our bound variable, becasue, notice that, the point of the lambda operator is to _abstract_ away from the bound variable, i.e.:
$$
(\lambda x.x^2)3
$$
and $$(\lambda t.t^2)3$$ are equivalent expressions (evaluate to the same quanitity for all substitutions).

__This is called $\beta$-reduction and is central to _lambda_ calculus__

The cool thing about lambda calculus is that you could directly test it in code. For example, in python $\lambda x.x+1$ is written as:

<p align="center">
  <img src="photos/Screenshot 2024-06-18 at 1.29.33 AM.png">
</p>
and evalued as:
<p align="center">
  <img src="photos/Screenshot 2024-06-18 at 1.27.10 AM.png" alt="alt text">
</p>

### Unary but Higher Order

As you may have noticed by now: lambda functions are unary! Which seems like a problem because if the claim is that in can do everything that a programming langugage can then it must be able to encode functions like:
```python3
def sum(a, b):
    return a+b
```
which takes in two arguments. We can in fact do this, the key is that even though the functions are unary, they are higher order, i.e., they can (and in a langugae like lambda calculus which only has the notion of functions, MUST) return other lambdas. The lambda function for the above python code if:
$$
\lambda x. \lambda y.x+y
$$

Let's test this by $\beta$-reducing it:
$$
    (\lambda x. \lambda y.x+y)2 \space 3 \\ \implies (\lambda y.2+y) 3 \\ \implies 2+3 \\ \implies 5
$$

This is indeed correct and testable:

<p align="center">
  <img src="photos/Screenshot 2024-06-18 at 1.54.42 AM.png" alt="alt text">
</p>

Again, for testing, the only constraints on the types of the inputs are that they must be summable in Python (they cant be int and a string). Again, in lambda calculus the notion of types is not inherent and the only thing we can work with are other lambdas.

<p align="center">
  <img src="photos/Screenshot 2024-06-18 at 1.57.17 AM.png" alt="alt text">
</p>

Also notice that the type of the value returned by $
(\lambda x. \lambda y.x+y)2 \space 3
$ is an int, but the type returned by $
(\lambda x. \lambda y.x+y)2
$ is a function, i.e., the first lambda, which binds x$$, return another lambda  which binds $y$. In code this is analogous to:
```javascript 
function lambda1(x){
    return function lambda2(y){
        return x+y
    }
}
```
<p align="center">
  <img src="photos/Screenshot 2024-06-18 at 2.04.38 AM.png" alt="alt text">
</p>

## A note on grammar

I initially opted to skip this section and instead write it in the appendix, but as you will see soon ahead, it serves us very well to develop and define an unambiguous parsing grammar for $\lambda$ calculus.

The complete BNF grammar is:
$$

\begin{array}{rcl}
\langle \lambda\text{exp} \rangle & ::= & \langle \text{var} \rangle \\
                                  & \mid & (\langle \lambda\text{exp} \rangle) \\
                                  & \mid & \lambda \langle \text{var} \rangle . \langle \lambda\text{exp} \rangle \\
                                  & \mid & ( \langle \lambda\text{exp} \rangle )\langle \lambda\text{exp} \rangle 
\end{array}

$$

This recursively defines all the valid expressions in the language

1) Variable: The first condition denotes that a variable like $x$ is a valid expression.
2) if M is valid then (M) is also valid.
3) Lambda Abstraction: A $\lambda$ expression of the type $\lambda x.M$ is valid where $M$ is either a $\langle \text{var} \rangle$ as in $\lambda x.x^2$ or another lambda abstraction. Note '.' sperates the input and the function.
4) Application or Substitution: The third rule corresponds to the substituion phase where the abstraction in the first brackets denotes to the function that needs to be evaluated and the second abstraction is the argument passed to it (note that the language only has functions, so the argument is also a function as opposed to a number like 1).

__Note: also  (by convention) application is left associative: ABC means (AB)C not A(BC), and application/substituion has higher precedence than abstraction: λx.AB means λx.(AB), not (λx.A)B.__ 

## But how do you program with this?

The first thing we will encode for is Booleans. Again, booleans as a data type don't exist in our langugage yet so the best thing we can do is _emulate_ their behavior. 

The fundamental function of booleans in progrgamming is control flow as an `if-else` clause:
```cpp
if(condition is true){do something}
else{do something else}
```

So, the encoding for True as a lambda function is:
$$
  \lambda x.\lambda y. x
$$
and the eoncding for False is:
$$
  \lambda x.\lambda y. y
$$

Notice, that True is basically a function that takes in two arguments and returns the first (corresponding to the first branch in an `if-else` statement) and the vice-versa for False.

To see how this emulates `if-else` consider a lambda function $F$ that returns either the True lambda function or the False lambda function, and depending on its truth value we want to return either $M$ or $N$, i.e.,
```cpp
if(F){
    return M
}
else{
    return N
}
```

is the same as saying:
$$
\lambda F. \lambda m.\lambda n. F \space m\space n
$$

Consider the $\beta$-reduction when True is passed (F is True):
$$
  (\lambda F. \lambda m.\lambda n. F \space m\space n)\space True \space M \space N
  \\
  \implies (\lambda m.\lambda n.True \space m \space n)\space M \space N
\\
\implies (\lambda n. True \space M \space n)\space N
\\
\implies True \space M \space N
\\
\implies (\lambda x. \lambda y. x )M  \space N
\\
\implies (\lambda y.M)N
\\
\implies M
$$

and the $\beta$-reduction when the condition $F$ is False:
$$
  (\lambda F. \lambda m.\lambda n. F \space m\space n)\space False \space M \space N
  \\
  \implies (\lambda m.\lambda n.False \space m \space n)\space M \space N
\\
\implies (\lambda n. False \space M \space n)\space N
\\
\implies False \space M \space N
\\
\implies (\lambda x. \lambda y. y )M  \space N
\\
\implies (\lambda y.y)N
\\
\implies N
$$

![alt text](<photos/Screenshot 2024-06-19 at 4.30.14 PM.png>)

Also note that we can name our lambda abstractions and use these names as aliases for the expression. We do this by replace $\lambda$ by the name of the function, i.e., a $\lambda$ function is an anonymous function, which is consistent with the use of lambda functions in languages like Java where it means an anonymous function.

Hence:
$$
True \space x. \lambda y. x \\ \text{and} \\ False \space x. \lambda y. y
$$

but in interest of readability I will use the following aliasing convention:

$$
True = \lambda x. \lambda y. x \\ \text{and} \\ False = \lambda x. \lambda y. y
$$

Now that we have defined the True and False, we can do a lot of interesting things with just these.

For example:
$$
  AND = \lambda x. \lambda y. \space x \space y \space False
$$

The currying steps when $x$ is True and $y$ is False:
$$
(\lambda x. \lambda y. \space x \space y \space False) True \space False 
\\
\implies (\lambda y. True \space y \space False) False
\\
\implies True \space False \space False
\\
\implies (\lambda x. \lambda y. x) False \space False
\\
...
\\
\implies False
$$

You could try to substitute $x$ and $y$ for all permutations of $True$ and $False$ and verify that the truth table is correct.

Intuitively, if $x$ is False, it will choose the second input to it which is False, and if it is True it will choose $y$.

If we are able to encode for $NOT$ we will have the full range of logical operators since {$ \land, \lnot $} make the universal set of logical operators.

$$
NOT = \lambda x. x \space False \space True
$$

Verification of the truth table is fairly simple in this case and left as an exercise :p but intuitvely, if $x$ is True it will select the first input which is False, and if not, then the second which is True.

Now all other logical operators can be defined in terms of $AND$ and $NOT$.

$$
OR \equiv \lor \equiv \lnot(\lnot A \land \lnot B)
\\
OR = NOT(AND(NOT(A) \space \space NOT(B)))
\\
OR = (\lambda x. x \space False \space True) AND(NOT(A) \space \space NOT(B))
\\
OR = (AND(NOT(A) \space \space NOT(B))) False \space True
\\
OR =  AND(\lambda x. x \space False \space True(A) \space \space \lambda x. x \space False \space True(B)) False \space True
\\
OR = AND((A \space False \space True) \space \space (B\space False \space True)) False \space True
\\
OR = (\lambda x. \lambda y. \space x \space y \space False ((A \space False \space True) \space \space (B\space False \space True))) False \space True
\\
OR = (\lambda y. \space (A \space False \space True) \space y \space False (B\space False \space True)) False \space True
\\
OR = ((A \space False \space True) \space (B\space False \space True) \space False) False \space True
\\[0.3in]
\text{Hence,}
\\
OR = \lambda A. \lambda B. ((A \space False \space True) \space (B\space False \space True) \space False) False \space True
$$

You could try making the truth table for this expression to verify that this is indeed correct. However, we could think of a simpler expression for $OR$. When the first argument is True we want to return True, otherwise we want to return whatever the second input is:

$$
OR = \lambda x. \lambda y. x \space True \space y
$$

But at least we know for sure that it is possible to write any logical operator by some combination of AND and OR. The proof of why AND and OR make the universal set is something that I discovered the existence of in the appendix of my logic text book 10 minutes before the final paper of the course, and never bothered to understand. 

## Numerals

Numbers don't exist. But we don't need them. What does a number even do? It just tells you a quantity, and then you can do _something_ quantity number of times. And you can do arithmetic on those quantity numbers to make new quantities. Big deal. 

This is Zero: $$
\lambda f. \lambda x. x$$ what is even going on here? Semantically: Zero takes in an operation $f$ and a starting value $x$ and applies $f$ to $x$ zero number of times, by just returning $x$. 

This is One:
$$
  \lambda f. \lambda x. f x
$$

This is Two:
$$
  \lambda f. \lambda x. f(fx)
$$

Note that the encodings for Zero is the same as False, and that for One is the same as True, which is so surprising. It occurs naturally without us having intended it and corresponds to how booleans are dealt with in formal logic and most programming languages (0 for False and 1 for True).

You could go one defining all numbers by hand, and I think there is something to pretty and artisnal about it: defining the very basics of a language by hand. But we could also define a successor function, that takes in a number, a function, and a starting value and apply the the function one more time than the input number to the starting value.

$$
succ = \lambda n. \lambda f. \lambda x. f(n\space f\space x)
$$

## Our first program:

We have done enough groundwork to build actual programs already. Most of these programs naturally Boolean functions, built by encoding for predicates and then using an `if-else` style control flow. First we will encode for the predicate `isZero()` which looks like this in cpp:
```cpp
bool isZero(int n){
  if(n==0){
    return true;
  }
  else{
    return false;
  }
}

int main(){
  if(isZero(0)){
    printf("Statement is true");
  }
  else{
    printf("Statement is false");
  }
}
```

This translates to:
$$
  isZero = \lambda x. x \space (\lambda n. False)\space True
$$

Why is this correct? It is so, because Zero is a lambda (like False) that returns it's second argument so the second argument should be True for isZero, and the first should return False however many times it is applied to itself.

![alt text](<photos/Screenshot 2024-06-19 at 4.34.12 PM.png>)

We could also write a function that tells if a number is odd or even:
$$
  isEven = \lambda x. x \space NOT \space True
$$

If we $x$ is `zero` it returns the second argument which is True, otherwise if $x$ is `n` it applies the $NOT$ lambda $n$ times to True.

![alt text](<photos/Screenshot 2024-06-19 at 6.59.19 PM.png>)

Using `isEven` we can build other functions like one which takes in two arguments and checks if their parities are equal.

```cpp
bool parityEqual(int n, int m){
    if(!(isEven(n) ^ isEven(m))){ // ^ is XOR
        return true;
    }
    else{
        return false;
    }
}
```

So first let's define the XOR lambda:
$$
  XOR = \lambda x. \lambda y. x(NOT \space y) \space y
$$

i.e., if $x$ is false return whatever $y$ is, otherwise return $\lnot y$.

So, parityEqual:
$$
  parityEqual = \lambda x. \lambda y. XOR(isEven(x) \space isEven(y)) \space False \space True
  \\[0.3in]
  \text{Substituting isEven(x) and isEven(y) in XOR}:
  \\[0.3in]
  \implies(\lambda x. \lambda y. x(NOT \space y) \space y)\space isEven(x) \space isEven(y)
  \\
  \implies (\lambda y. isEven(x) (NOT \space y) \space y)\space isEven(y)
  \\
  \implies isEven(x) (NOT \space isEven(y))\space  isEven(y)
  \\[0.3in]
  Hence,\\
  parityEqual = \lambda x. \lambda y. isEven(x) (NOT \space isEven(y))\space  isEven(y) \space False \space True
$$

![alt text](<photos/Screenshot 2024-06-19 at 7.28.58 PM.png>)

Another important function we should define is `areEqual` which takes in two numerals (their lambda expressions) are returns if they are equal or not. One way to do it is two subtract the two and pass the subtraction to the `isZero` function. Which means we need to encode for $+$, $-$ and other arithemtic operations.

## Arithmetic

### Addition

An addition lambda must take $m$ and $n$, two numerals as input and since, $m$ and $n$ correspond to applying a function $f$ (first input) to a starting value $x$ (second input) $m$, and $n$ number of times respectively, the output should apply $f$ to $x$ $\space$ $m+n$ number of times:
$$
add = \lambda m. \lambda n. \lambda f. \lambda x. m(f)(n(f)(x))
$$

Fundamentally `add` applies $f$ to itself with $x$ as the starting value $n$ times then applies $f$ to itself with $n(f)(x)$ as the sarting value $m$ times, so that $f$ is applied to itself with $x$ as the starting value $m+n$ times.

### Multiplication

$m*n$ is the same as adding $n$ to itself $m$ times. So the multiplication lambda must apply $n$ to $f$ and $x$ $m $ times and that's it.
$$
mult = \lambda m. \lambda n. \lambda f. \lambda x. m(n(f))(x)
$$

To define the subtraction function we need to define the predecessor function, which is a function, which applies $f$ to itself one _less_ time, which is way trickier than `succ`.

### Predecessor

The funny thing about the predecessor function is that it was given by god, no one knows how it works, men far beyond my stature have driven themselves insane trying to foolishly decipher this divine arrangemenr of symbols. Don't try to understand it, just know that this exists, and quickly abstract upwards (If youre so smart go through the links at the end to try to understand it).

$$
pred=λn.λf.λx.(n(λg.λh.h(gf))(λu.x))(λu.u)
$$

A super helpful discussion on the semantics of `pred` is here: https://stackoverflow.com/questions/8790249/lambda-calculus-predecessor-function-reduction-steps, with the lower answers providing a better explanation, especially the ones by https://stackoverflow.com/users/12120531/dmitri-gekhtman and https://stackoverflow.com/users/418966/cyker 

For the sake of completeness, here is dmitri's answer:

>I'll add my explanation to the above good ones, mostly for the sake of my own understanding. Here's the definition of PRED again: 
>
>PRED := λnfx. (n (λg (λh.h (g f))) )  λu.x λu.u
The stuff on the right side of the first dot is supposed to be the (n-1) fold composition of f applied to x: f^(n-1)(x).
>
>Let's see why this is the case by incrementally grokking the expression.
>
>λu.x is the constant function valued at x. Let's just denote it const_x.
>
>λu.u is the identity function. Let's call it id.
>
>λg (λh.h (g f)) is a weird function that we need to understand. Let's call it F.
>
>Ok, so PRED tells us to evaluate the n-fold composition of F on the constant function and then to evaluate the result on the identity function.
>
>PRED := λnfx. F^n const_x id
>Let's take a closer look at F:
>
>F:= λg (λh.h (g f))
>F sends g to evaluation at g(f). Let's denote evaluation at value y by ev_y. That is, ev_y := λh.h y
>
>So
>
>F = λg. ev_{g(f)}
>Now we figure out what F^n const_x is.
>
>F const_x = ev_{const_x(f)} = ev_x
>and
>
>F^2 const_x = F ev_x = ev_{ev_x(f)} = ev_{f(x)}
>Similarly,
>
>F^3 const_x = F ev_{f(x)} = ev_{f^2(x)}
>and so on:
>
>F^n const_x = ev_{f^(n-1)(x)}
>
>Now,
>
>PRED
>     = λnfx. F^n const_x id
>
     = λnfx. ev_{f^(n-1)(x)} id

     = λnfx. id(f^(n-1)(x))

     = λnfx. f^(n-1)(x)
>which is what we wanted.
>
>Super goofy. The idea is to turn doing something n times into doing f n-1 times. The solution is to apply F n times to const_x to obtain ev_{f^(n-1)(x)} and then to extract f^(n-1)(x) by evaluating at the identity function.

Ok! yay! That was something. Good thing is youll probably never need to come up with this on your own :D

### Subtraction

$$
sub = \lambda n. \lambda m. \lambda f. \lambda x. (m(pred)(n))f \space x
$$

Hence, this corresponds to $n- m$.

![alt text](<photos/Screenshot 2024-06-19 at 11.59.45 PM.png>)

Notice that the sub function is built using pred, which semantically means applying a function one less number of times to some value. But what happens when you pass zero to pred? You could try $\beta$-reducing it to find that it returns zero. Hence, the subtraction of a bigger number from a smaller number will equal 0.

![alt text](<photos/Screenshot 2024-06-20 at 12.03.52 AM.png>)

## Equality

We have now done enough groundwork to define a very important predicate: `isEqual`.

Notice that two numbers are only equal when their difference is 0, and we already have an `isZero`. But having a simple lambda like this:

$$
\lambda m. \lambda n. (isZero(sub(m \space n))) \space True \space False
$$

will not work because sub(m n) will be zero when n is greater than m. The key to overcoming this is to notice that if two numbers $m$, $n$ are equal, both $m-n$ and $n-m$ are Zero, and when they are not equal only one of the differences is Zero (corresponding to smaller - greater).

Hence the predicate we want to check for is:
$$
isZero(m-n) \land isZero(n-m)
$$

Translating to lambda expression:

$$
isEqual = \lambda m. \lambda n. AND(isZero(sub(m \space n)) \space isZero(sub(n \space m)))\space True \space False
$$



## loops

## FizzBuzz

while loop. division. 