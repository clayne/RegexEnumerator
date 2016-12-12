# Regex Enumerator

Enumerate Regular Expressions the Fun Way.

<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/aa4d414dd40b418624756ba65b24c190.svg?invert_in_darkmode" align=middle width=672.31395pt height=39.45249pt/></p>

<p align="center">
<img src="http://i.imgur.com/sRo5tQz.png?invert_in_darkmode"/>
</p>

<sub>*Or how I learned to stop worrying and start counting things with calculus*</sub>

-----

### Table of Contents

* [Regex Enumerator](#regex-enumerator)
     * [Table of Contents](#table-of-contents)
     * [Installation](#installation)
     * [Usage](#usage)
        * [Regular Expression Syntax](#regular-expression-syntax)
        * [Library Functions](#library-functions)
        * [Caveat](#caveat)
     * [Justification](#justification)
        * [Fibonacci Redux](#fibonacci-redux)
     * [Additional Examples](#additional-examples)


-----

Have you ever wondered about how many different strings you can form that fits your favorite regex?

Yeah, chances are you probably haven't. But it's on your mind now.

Here's one of my favorite regular expressions:
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/120bcbe220479f3fb301392b145130a5.svg?invert_in_darkmode" align=middle width=65.09151pt height=18.020145pt/></p>
It specifies the class of languages that are comma-separated list of strings of zeros. For example,
`000, 0, 00000` belongs to this language, but `0,,0` and `0,0,` does not.

Now, it might seem like a masochistic endeavor, but if you enumerate every possible word in this language, you'll
find that there are no empty strings, 1 single letter string, 1 two letter string, 2 three letter strings, 3 four letter
strings, and so on. This pattern actually looks like

    0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, ...

Why, that is the fibonacci sequence! How did it end up in such a mundane place?

Now, I could give you a combinatorial interpretation for this amazeballs result, but I still get shivers up my spine
whenever I think back to my undergrad Combinatorics course. Instead, I'll give a more general way to compute these
enumerations.

However, that's not the end of it. It turns out that this algorithm can also compute a closed-form formula
for this sequence.
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/c709d2b386c7588133c620efa335e165.svg?invert_in_darkmode" align=middle width=514.0443pt height=52.667175pt/></p>
where <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/c9c53a99901c4a67544997f70b0f01bc.svg?invert_in_darkmode" align=middle width=18.19125pt height=23.24256pt/> is the number of comma-separated lists of size <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.36144pt height=14.93184pt/>.

Now, this might not look very pretty, but it's still pretty cool that there is a (computable) closed form expression that
counts every regular expression.

-----------------------------------

### Installation

This library is just meant to be a demonstration. 

You will need Python 2.7 or up, though it seems to be most stable on Python 3+.

Note that you will need to install `numpy`, `scipy`, and `sympy` in order to support solving a few
linear equations and to translate numerically computed roots into algebraic forms, if they are available.

```bash
git clone https://github.com/leegao/RegexEnumerator.git
cd RegexEnumerator
sudo python setup.py develop
```

To uninstall, run

```bash
pip uninstall RegexEnumerator
```

### Usage

#### Regular Expression Syntax

We are using vanilla regular expression, so the standard `*`, `+`, `?`, `|` variety. Note that for `+` and `?`, we've
encoded them using just `*` and `|` instead:

* <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/b44b0a181653840433cbcd08a7638cb7.svg?invert_in_darkmode" align=middle width=78.053415pt height=13.9105725pt/></p>
* <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/197232bbcfbca260e636689189e10f3b.svg?invert_in_darkmode" align=middle width=88.823955pt height=16.438356pt/></p>

Here, `%` denotes the "empty" transition <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/7ccca27b5ccc533a2dd72dc6fa28ed84.svg?invert_in_darkmode" align=middle width=6.1668915pt height=14.93184pt/> in formal languages. In effect, it acts as the
identity element of concatenation, so that <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/ffbd759215219ed839b38b1f27aedd2f.svg?invert_in_darkmode" align=middle width=58.917705pt height=15.38856pt/>. For example, the regular expression of
comma delimited language <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/4052b2961cf5ab33404b96cd5e12aae9.svg?invert_in_darkmode" align=middle width=71.423715pt height=26.95407pt/> can be encoded as
```python
e = '0' # or any other regular expression
regex = '({e}{e}*,)*{e}{e}*'.format(e = e)
```

#### Library Functions

`regex_enumerate` offers a few library functions for you to use.

* `enumerate_coefficients`: Runs the magical algorithm to give you an algorithm that can compute
  the count of words of size `n` in time that is only proportional (linearly) to the number of terms in your
  regular expression.
  
  ```python
  from regex_enumerate import enumerate_coefficients
  from itertools import islice
  
  print(list(islice(enumerate_coefficients('(0+1)*0+'), 10)))
  # [0.0, 1.0, 0.99999999999999989, 1.9999999999999998, 2.9999999999999996, 4.9999999999999982, 7.9999999999999982, 12.999999999999998, 20.999999999999993, 33.999999999999986].
  ```

* `exact_coefficients`: Uses a dynamic program to compute the same coefficients. Useful for validation
  and pure computation, but does not reveal any algebraic structure within the problem.
  
  ```python
  from regex_enumerate import exact_coefficients
  from itertools import islice
  
  print(list(islice(exact_coefficients('(0+1)*0+'), 10)))
  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
  ```

* `algebraic_form`: Computes the algebraic closed form of a regular expression.

  ```python
  from regex_enumerate import algebraic_form, evaluate_expression
  from sympy import latex, pprint
  
  formula = algebraic_form('(0+1)0+')
  
  # Normal Form
  print(formula)
  # 2.0*DiracDelta(n) + 1.0*DiracDelta(n - 1) + binomial(n + 1, 1) - 3
  
  # Latex
  print(latex(formula))
  # 2.0 \delta\left(n\right) + 1.0 \delta\left(n - 1\right) + {\binom{n + 1}{1}} - 3
  
  # ASCII/Unicode pretty print
  print(pprint(formula))
  #                                             /n + 1\
  # 2.0*DiracDelta(n) + 1.0*DiracDelta(n - 1) + |     | - 3
  #                                             \  1  /
  
  print(evaluate_expression(formula, 10))
  # 8
  ```

  The magic behind this will be discussed in the next section. The <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/c068b57af6b6fa949824f73dcb828783.svg?invert_in_darkmode" align=middle width=41.681475pt height=23.24256pt/> code looks like 
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/1fc3c5fe30de9b941a91921b8527493b.svg?invert_in_darkmode" align=middle width=267.8313pt height=39.45249pt/></p>
  Note that this differs from the above since we're enumerating <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/d2432a60d1dc5806cd53447ce48d2e43.svg?invert_in_darkmode" align=middle width=57.942225pt height=26.95407pt/> instead of <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/f9d2f9a74a3d1a9fc852220717fcbd49.svg?invert_in_darkmode" align=middle width=65.49939pt height=26.95407pt/>.

* `check_on_oeis`: This will search https://oeis.org for a potential combinatorial interpretation of your
  enumeration.
  
  ```python
  from regex_enumerate import check_on_oeis
  sequences = check_on_oeis("(0+,)*0+", start=5)
  for oeis in sequences:
    print('%s: https://oeis.org/%s' % (oeis.name, oeis.id))

  # Fibonacci numbers: https://oeis.org/A000045
  # Pisot sequences E(3,5), P(3,5): https://oeis.org/A020701
  # Expansion of (1-x)/(1-x-x^2): https://oeis.org/A212804
  # Pisot sequence E(2,3): https://oeis.org/A020695
  # Least k such that the maximum number of elements among the continued fractions for k/1, k/2, k/3, k/4 : https://oeis.org/A071679
  # a(n) = Fibonacci(n) mod n^3: https://oeis.org/A132636
  # Expansion of 1/(1 - x - x^2 + x^18 - x^20): https://oeis.org/A185357
  # Nearly-Fibonacci sequence: https://oeis.org/A264800
  # Pisot sequences E(5,8), P(5,8): https://oeis.org/A020712
  # a(n) = s(1)t(n) + s(2)t(n-1) + : https://oeis.org/A024595
  ```

* `disambiguate(regex)`: [EXPERIMENTAL] attempts to construct an unambiguous regular expression. In many cases,
  regular expressions are ambiguous. For example, <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/a5e47a69368560eb1f96acf425b6b4da.svg?invert_in_darkmode" align=middle width=42.416715pt height=25.43409pt/> is a classic example. Ambiguities is the source of
  redundancy, and unfortunately, our enumeration methods won't understand that the redundant components are already
  taken care of. Therefore, care must be taken to to ensure that the regular expression is unambiguous.
  
  This is an experimental algorithm that reduces any regular expression into an ambiguity free form. The cost is a
  potentially exponential blow-up in the size of your regular expression. However, for most of the simple cases, this
  is alright.
  
  ```python
  from regex_enumerate import disambiguate, enumerate_coefficients
  from itertools import islice
  
  # 0*0* is equivalent to just 0*
  print(list(islice(enumerate_coefficients('0*0*'), 10)))
  # [1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0]
  
  # Let's disambiguate this problem
  print(list(islice(enumerate_coefficients(disambiguate('0*0*')), 10)))
  # [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0]
  ```
  
  In general, this should work. However, it does require a fair bit of term-rewriting to
  ensure that some of the intermediate steps can be reduced properly. Therefore, if something
  seems fishy, you can always inspect the reconstructed DFA and its disambiguation form
  <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/d507b0de6241f92619c33714931ba5f0.svg?invert_in_darkmode" align=middle width=60.00258pt height=28.43346pt/>, which will be described below.
  
  ```python
  from regex_enumerate import compile_disambiguously, reduce
  from random import sample
  R, dfa, accepts, number_of_states = compile_disambiguously("0*0*")
  # The states are arbitrarily ordered, so we can say that a state u < v when its 'id' is less than that of the others.
  
  # R(u, v, k) is the regular expression that allows an automaton to transition from u to v using only nodes
  # [1, ..., k] in its intermediate steps.
  print(R(1, *sample(accepts), 3)) # 1 -> ...(<3) -> some random final state
  print(R(1, *sample(accepts), number_of_states)) # Regex describing all the ways of getting from start (node 1) to some final state
  # Additionally, R(u, v, k) is designed to be mutually orthogonal, so R(u, v, k) + R(u, v', k) is unambiguous
  ```

In addition, regular expressions correspond to the family of rational functions (quotient of two polynomials).
To see the generating function of a regular expression, try

```python
from regex_enumerate import generating_function
from sympy import latex

print(latex(generating_function("(0+1)*0+")))
```

which outputs
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/2f5a67f61cf674ae11beb062c1f892d6.svg?invert_in_darkmode" align=middle width=140.091435pt height=34.360095pt/></p>

#### Caveat

There are many regular expressions that are ambiguous. For example, the regular expression
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/c983706ca99ab3174d64641faec9442f.svg?invert_in_darkmode" align=middle width=30.13692pt height=16.438356pt/></p>
is inherently ambiguous. On encountering a `0`, it's not clear which side of the bar it belongs to. While
this poses no challenges to parsing (since we don't output a parse-tree), it does matter in enumeration.
In particular, the direct translation of this expression will claim that there are 2 strings of size 1
in this language.

To remedy this, you can try to use `regex_enumerate.disambiguate(regex)`, but it's not completely clear
that this is correct. Therefore, know that
for some regular expressions, this technique will fail unless you manually reduce it to an unambiguous form.
There is always a way to do this, though it might create an exponential number of additional states.

### Justification

Now, all of this might feel a little bullshitty. (Shameless plug, for more bullshitty math, check out http://bullshitmath.lol)
Is there any real justification for what you are doing here? Am I just enumerating a bunch of pre-existing cases
and running through a giant table lookup?

Well, it's actually a lot simpler than that. However, there's a bit of a setup for the problem.

#### Fibonacci Redux

Let's rewind back to our first example; that of enumerating comma-separated sequences of `x`es:
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/c6281cf4f962d25fbc36c57ce7850bbf.svg?invert_in_darkmode" align=middle width=109.31118pt height=16.438356pt/></p>
We've seen above that this follows a fibonacci-like sequence. Is there some-way that we can derive this
fact without brute-force enumeration?

Let's start with the sequence of `x`es: <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/4522248e54d76be0a26031c14eeb96a9.svg?invert_in_darkmode" align=middle width=17.108685pt height=16.07364pt/>. This language, in an infinitely expanded form, looks like
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/3381c5da3dc46afe9e4deb717a7f6664.svg?invert_in_darkmode" align=middle width=175.8834pt height=16.438356pt/></p>

Now, here's a trick. Let's pretend that our bar (<img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/57fe5a91a139252a27ed191b2680eda7.svg?invert_in_darkmode" align=middle width=4.0607325pt height=25.43409pt/>) is a plus sign (<img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.27996pt height=19.95477pt/>), so that
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/e6a8413df070790dcc80a7af6dc4e05b.svg?invert_in_darkmode" align=middle width=201.4551pt height=13.511025pt/></p>

This looks remarkably familiar. In fact, if you are working within a numerical field, then a little bit of
precalculus would also show that
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/a48f22fd8de4155848bc1a018da8b40e.svg?invert_in_darkmode" align=middle width=225.72825pt height=34.360095pt/></p>

Could there be some connection here? Well, let's find out. To do this, let's equate the two expressions:
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/bf7e136f136c7efe33bc173e1bf1a070.svg?invert_in_darkmode" align=middle width=484.71555pt height=34.360095pt/></p>
so <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/fe2120e91ad08218bdfb64ad6b47fa90.svg?invert_in_darkmode" align=middle width=36.303795pt height=21.96381pt/> and <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/6b424929bb1a83860737d2188f80b16f.svg?invert_in_darkmode" align=middle width=64.617795pt height=28.55226pt/> if we pretend that each regular expression has a numerical value.

In fact, this works for every regular expression. For any regular expressions <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/49f3694ae5275a3e33da3e17e4dd9528.svg?invert_in_darkmode" align=middle width=36.03567pt height=14.93184pt/> and for any letters <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/6ecf10ed1c08ba92db30119ef192228f.svg?invert_in_darkmode" align=middle width=40.51806pt height=14.93184pt/> we have
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/5f1add925794ecf4a2957a175b9ee8cc.svg?invert_in_darkmode" align=middle width=114.99972pt height=131.41953pt/></p>
As long as you don't need to invoke the axiom of multiplicative-commutativity, this reduction works.

For example, for the comma-separated list example, we have
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/52d12c6130dc0b733e761d30072694e5.svg?invert_in_darkmode" align=middle width=241.30095pt height=84.50739pt/></p>

Note here that <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/497403162f282a343c315086f75ba766.svg?invert_in_darkmode" align=middle width=4.0607325pt height=14.93184pt/> is a variable! It might be tempting to try to simplify this further. Letting <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/2404d424965cf131c13c2080768326c8.svg?invert_in_darkmode" align=middle width=11.043285pt height=14.93184pt/> denote the comma, 
we might try
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/d8c119d2514099b760bc51bb48b0e9fd.svg?invert_in_darkmode" align=middle width=260.3436pt height=84.74664pt/></p>

But this requires a crucial axiom that we do not have:

* We do not have multiplicative commutativity, so we couldn't merge <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/080bdd9337021a90b67fc7ed0a645b6e.svg?invert_in_darkmode" align=middle width=103.39791pt height=28.55226pt/>, since 
  no longer know whether this is <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/5a43d840c0717ad3ba88234ef1d697fb.svg?invert_in_darkmode" align=middle width=55.234245pt height=28.55226pt/> or <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/76f426704274ce73132925d7690385b3.svg?invert_in_darkmode" align=middle width=55.22682pt height=28.55226pt/>.
[](
This begs a natural question. If we can't take inverses or negate things, then why do we admit the expression <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/3f332093a7dbcc97f190d6d57fa8d322.svg?invert_in_darkmode" align=middle width=23.768085pt height=28.55226pt/>?
Well, in this language, that term is **atomic**. Therefore, we cannot break it down and look at it as a subtraction followed by
an inverse; it is just <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/982c522bfaa34942669aa7ebd7bbdca3.svg?invert_in_darkmode" align=middle width=23.768085pt height=28.55226pt/>. I'll clear this up later.
)

Now that we have this weird "compiler" taking us from regular expressions to numerical formulas, can you tell us what it means
for a regular expression to take a numerical value?

The answer: none. There is no meaning to assign a value of say <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/22f2e6fc19e491418d1ec4ee1ef94335.svg?invert_in_darkmode" align=middle width=20.499105pt height=21.96381pt/> to <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode" align=middle width=8.88954pt height=14.93184pt/>, or that <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/3f6a01eca3b8d6c424522f2e11ccb80e.svg?invert_in_darkmode" align=middle width=58.623015pt height=23.41515pt/>. It doesn't mean anything, 
it's just pure gibberish. Don't do it, except maybe values of <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode" align=middle width=7.713717pt height=21.96381pt/> or <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode" align=middle width=7.713717pt height=21.96381pt/>; we'll get to that later.

Okay. So why did we go on this wild goose-hunt if their values don't even mean anything? 
It turns out that the value of a formula is not what we are interested in; these objects are compact and have nice algebraic properties.
When we count things, we just care about how many objects there are that satisfies a certain property.
When we count all words of, say, size 5 in a language, we don't care whether these strings are `000,0` or `0,0,0`. The ordering
of the letters in these strings are extraneous details that we no longer care about. Therefore, it would be nice to be able to
forget these details. More formally, if the order of letters in a word doesn't matter, we would say that
*we want the concatenation operator to be commutative*. If there's a representational equivalence to the numerical "field",
then the translation would be that *we want the multiplication operator to be commutative.*

This is a huge game-changer. In the above example, we weren't able to fully simplify that ugly product of fractions precisely
because we lacked this crucial axiom. Luckily for us, it now allows us to fully simplify the expression
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/b5a38b664b97b87056dd23d6d12873bf.svg?invert_in_darkmode" align=middle width=196.581pt height=34.177275pt/></p>
Which tells us that our regular expression is isomorphic to the regular expression <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/82bb6a5e8e09489f748ec969fe5190f4.svg?invert_in_darkmode" align=middle width=69.026265pt height=25.43409pt/>. That is, for each
comma-separated list, you can map it to one of the words in <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/82bb6a5e8e09489f748ec969fe5190f4.svg?invert_in_darkmode" align=middle width=69.026265pt height=25.43409pt/>. In fact, not only are these two languages
isomorphic; they are the same! A moment of thought reveals that this new regular expression also matches only comma-separated list
of sequences as well.

That's a pretty cool trick to deduce equivalences between regular expressions, but is that all there is to it?

It turns out that each of these translated numerical expressions also admit an infinite series expansion (in terms of its free variables). So
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/0f96f40aed211541cae40f311b924b63.svg?invert_in_darkmode" align=middle width=459.71805pt height=34.177275pt/></p>
and in general, we have the multivariable expansion
<p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/ac3262648c32411a072d132fd3c8085f.svg?invert_in_darkmode" align=middle width=341.8173pt height=40.54809pt/></p>
where <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/0aae089ed20772138e327117bd8c6bac.svg?invert_in_darkmode" align=middle width=12.834525pt height=14.93184pt/> is the coefficient attached to the <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/1949bc43d509deddd1ed78695ad786ff.svg?invert_in_darkmode" align=middle width=66.720555pt height=30.61674pt/> term.

However, recall that each of the <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/1abf06af410e5fda80c06d2b5d246d77.svg?invert_in_darkmode" align=middle width=133.7292pt height=22.61622pt/> corresponds to exactly one of
the words in our language. Therefore, if there are 5 words of size 6 with just one comma in our language, the coefficient in front
of <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/60406b22dbf1f8660041fb24bb74e5e1.svg?invert_in_darkmode" align=middle width=31.184175pt height=27.5385pt/> in the series expansion must be 5.

Herein lies the key to our approach. Once we grant the freedom of commutativity, each of these regular expressions "generates"
a numerical function with some infinite series expansion. The coefficients of the <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/1a1ddd375cabb38c3e605c08d7df4181.svg?invert_in_darkmode" align=middle width=49.69437pt height=31.80408pt/> term in this
expansion is then the total count of all objects in this regular language that has `i` <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/e714a3139958da04b41e3e607a544455.svg?invert_in_darkmode" align=middle width=15.44202pt height=14.93184pt/>s, `j` <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/277fbbae7d4bc65b6aa601ea481bebcc.svg?invert_in_darkmode" align=middle width=15.44202pt height=14.93184pt/>s, and `k` <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/95d239357c7dfa2e8d1fd21ff6ed5c7b.svg?invert_in_darkmode" align=middle width=15.44202pt height=14.93184pt/>s.

This approach is called the generating function approach within elementary combinatorics. It is a powerful idea to create
these compact analytical (if a bit nonsensical) representations of your combinatorial objects of interest in order to
use more powerful analytical tools to find properties about them.
### Additional Examples
* `(00*1)*`: 1-separated strings that starts with 0 and ends with 1

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/1123f13a5cf59d4870e9a7320d2f869e.svg?invert_in_darkmode" align=middle width=127.30608pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/4ce1b82d1485300e9111c68da61e077b.svg?invert_in_darkmode" align=middle width=501.83265pt height=52.667175pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Fibonacci numbers: https://oeis.org/A000045
  1. Pisot sequences E(3,5), P(3,5): https://oeis.org/A020701
  1. Fibonacci numbers whose decimal expansion does not contain any digit 0: https://oeis.org/A177194
  1. Expansion of (1-x)/(1-x-x^2): https://oeis.org/A212804
  1. Pisot sequence E(2,3): https://oeis.org/A020695
  1. Least k such that the maximum number of elements among the continued fractions for k/1, k/2, k/3, k/4 : https://oeis.org/A071679
  1. a(n) = Fibonacci(n) mod n^3: https://oeis.org/A132636
  1. Expansion of 1/(1 - x - x^2 + x^18 - x^20): https://oeis.org/A185357
  1. Numbers generated by a Fibonacci-like sequence in which zeros are suppressed: https://oeis.org/A243063
  1. Fibonacci numbers Fib(n) whose decimal expansion does not contain any digit 6: https://oeis.org/A177247

* `(%|1|11)(00*(1|11))*0* | 1`: complete 1 or 11-separated strings

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/b8aeb10e24f0dcd276ca81740fa7667f.svg?invert_in_darkmode" align=middle width=248.3646pt height=37.147275pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 3, 4, 7, 13, 24, 44, 81, 149, 274, 504, 927, 1705, 3136, 5768, 10609, 19513, 35890, 66012, 121415

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/024a6cd2f5d2a9090ad4da3705edb0b6.svg?invert_in_darkmode" align=middle width=1286.4951pt height=19.789935pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Tribonacci numbers: https://oeis.org/A000073

* `(000)*(111)*(22)*(33)*(44)*`: complex root to <img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/6e5a3ffb1d1e0a7d544ce8768d90c76d.svg?invert_in_darkmode" align=middle width=112.325565pt height=28.55226pt/>

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/744d2e9b7793e8d7eeebd74a206f57ed.svg?invert_in_darkmode" align=middle width=558.4359pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 0, 3, 2, 6, 6, 13, 12, 24, 24, 39, 42, 63, 66, 96, 102, 138, 150, 196, 210

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/e3e434f9f80fd7bd07e85029a3b48196.svg?invert_in_darkmode" align=middle width=2908.026pt height=42.804135pt/></p>

  A list of OEIS entries that contains this subsequence.


* `1*(22)*(333)*(4444)*(55555)*`: number of ways to make change give coins of denomination 1 2 3 4 and 5

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/88cc6eed2a47cad72b2f75df37e3223f.svg?invert_in_darkmode" align=middle width=677.8431pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 1, 2, 3, 5, 7, 10, 13, 18, 23, 30, 37, 47, 57, 70, 84, 101, 119, 141, 164

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/a2d29ceab49c773111de5872c5afe7ee.svg?invert_in_darkmode" align=middle width=4482.1095pt height=59.178735pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Number of partitions of n into at most 5 parts: https://oeis.org/A001401
  1. Number of partitions of n in which the greatest part is 5: https://oeis.org/A026811

* `11* 22* 33* 44* 55*`: 5 compositions of n

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/2478a0938079a952ba396709115a9a3c.svg?invert_in_darkmode" align=middle width=419.4267pt height=37.147275pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      0, 0, 0, 0, 0, 1, 5, 15, 35, 70, 126, 210, 330, 495, 715, 1001, 1365, 1820, 2380, 3060

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/7e7aeb9917c208bc8ac08bbf703d1ff9.svg?invert_in_darkmode" align=middle width=466.39395pt height=39.45249pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Binomial coefficient binomial(n,4) = n*(n-1)*(n-2)*(n-3)/24: https://oeis.org/A000332

* `(11*)*`: all compositions of n

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/aec8780df1f8cf1561a4af35a738592a.svg?invert_in_darkmode" align=middle width=126.32202pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536, 131072, 262144

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/9d2cd0af8c8bd1f4a588934cfc61fddc.svg?invert_in_darkmode" align=middle width=118.418685pt height=32.9901pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Powers of 2: https://oeis.org/A000079
  1. Expansion of (1-x)/(1-2*x) in powers of x: https://oeis.org/A011782
  1. Zero followed by powers of 2 (cf: https://oeis.org/A131577
  1. Powers of 2, omitting 2 itself: https://oeis.org/A151821
  1. Orders of finite Abelian groups having the incrementally largest numbers of nonisomorphic forms (A046054): https://oeis.org/A046055
  1. a(n) = floor(2^|n-1|/2): https://oeis.org/A034008
  1. Smallest exponent such that -1+3^a(n) is divisible by 2^n: https://oeis.org/A090129
  1. Pisot sequences E(4,8), L(4,8), P(4,8), T(4,8): https://oeis.org/A020707
  1. Numbers n such that in the difference triangle of the divisors of n (including the divisors of n) the diagonal from the bottom entry to n gives the divisors of n: https://oeis.org/A273109
  1. a(n)=2*A131577(n): https://oeis.org/A155559

* `(.........................)* (..........)* (.....)* (.)*`: number of ways to make n cents with US coins.

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/db2edcc857a1573432da15c9c6371800.svg?invert_in_darkmode" align=middle width=944.8296pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 4, 4, 4, 4, 4, 6, 6, 6, 6, 6

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/9c329eacacd565a206f02bc5bbe8179f.svg?invert_in_darkmode" align=middle width=21417.99pt height=59.178735pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Highest minimal distance of any Type I (strictly) singly-even binary self-dual code of length 2n: https://oeis.org/A105674
  1. Number of ways of making change for n cents using coins of 1, 5, 10, 25 cents: https://oeis.org/A001299
  1. Number of ways of making change for n cents using coins of 1, 5, 10, 25, 50 cents: https://oeis.org/A001300
  1. Number of ways of making change for n cents using coins of 1, 5, 10, 25, 50 and 100 cents: https://oeis.org/A169718
  1. Number of ways of making change for n cents using coins of 1, 5, 10, 20, 50, 100 cents: https://oeis.org/A001306
  1. Repetition of even numbers, with initial zeros, five times: https://oeis.org/A130496
  1. Number of ways of making change for n cents using coins of 1, 5, 10 cents: https://oeis.org/A187243
  1. Coefficients of the mock theta function chibar(q): https://oeis.org/A260984

* `(00*1)*00*`: list of 0-sequences

  Its generating function is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/dff15208c98f51e2b1f62cf27a693988.svg?invert_in_darkmode" align=middle width=140.091435pt height=34.360095pt/></p>
  For words of sizes up to 20 in this language, their counts are:

      0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181

  Its closed form is
  <p align="center"><img src="https://rawgit.com/leegao/RegexEnumerator/svgs/svgs/841cce6bf5f0f977e25c033bc101dd71.svg?invert_in_darkmode" align=middle width=472.60785pt height=52.667175pt/></p>

  A list of OEIS entries that contains this subsequence.

  1. Fibonacci numbers: https://oeis.org/A000045
  1. Pisot sequences E(3,5), P(3,5): https://oeis.org/A020701
  1. Expansion of (1-x)/(1-x-x^2): https://oeis.org/A212804
  1. Pisot sequence E(2,3): https://oeis.org/A020695
  1. Least k such that the maximum number of elements among the continued fractions for k/1, k/2, k/3, k/4 : https://oeis.org/A071679
  1. a(n) = Fibonacci(n) mod n^3: https://oeis.org/A132636
  1. Expansion of 1/(1 - x - x^2 + x^18 - x^20): https://oeis.org/A185357
  1. Nearly-Fibonacci sequence: https://oeis.org/A264800
  1. Pisot sequences E(5,8), P(5,8): https://oeis.org/A020712
  1. a(n) = s(1)t(n) + s(2)t(n-1) + : https://oeis.org/A024595
