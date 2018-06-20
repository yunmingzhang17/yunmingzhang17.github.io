---
layout: post
title: The GraphIt Programming Language 
---
The GraphIt Programming Language
==============================
{:.no_toc}

This guide introduces GraphIt language features and shows how they can be used in
programs. 


* auto-gen TOC:
{:toc}

# Basics
GraphIt is an imperative language with statements, control flow, and high-level operators on sts of vertices ane edges. In this section, we describe some of the language's basic constructs.

## Functions
Functions can take any number of parameters, including none. Each parameter
must be declared with its name followed by its type (separated by a `:`). In
the following example, the function `add` takes two parameters named `a` and
`b`, both of which are of type `float`, and returns a single result (named `c`)
that is also a `float`. The function result is separated from the list of
parameters by a `->` and the function declaration is delimited by the `end`
keyword.

```
func add(a : float, b : float) -> c : float
  c = a + b;
end
```

Like functions in MATLAB, GraphIt functions can return any number of results,
including none. In the next example, the function `minMax` takes two `float`
parameters and return two `float` results: the smaller of the two inputs as the
first result and the larger of the two inputs as the second result.

```
func minMax(a : float, b : float) 
    -> (c : float, d : float)
  if a < b
    c = a;
    d = b;
  else
    c = b;
    d = a;
  end
end
```

Note that the list of function results must be surrounded by parentheses if the
function returns more than one result. (The parentheses are optional if the
function returns just a single result.)


## Variables
Variables are declared in function bodies or in the global scope using the
`var` keyword. The following example declares an integer variable named `foo`:

```
var foo : int;
```

If the variable declaration has an initializer, then the variable's type can be
inferred from the initializer value. The following are equivalent ways to
define a floating-point variable that is initialized to `0.0`:

```
var foo : float = 0.0;
var foo = 0.0;
```

The `const` keyword creates a variable that cannot be modified after
initialization.  The following example shows three ways to declare constant
variables that are initialized to `0.0`:

```
const foo = 0.0;
const bar = foo;
bazz = 0.0;
```

Individual elements of const vectors can still be updated. 

## Comments
Single-line comments start with `%`:

```
h = 0.01;  % h is the time-step size.
```

Multi-line comments are surrounded by `%{` and `%}`:

```
%{
h is the time-step size.
h is initialized to one millisecond.
%}
h = 0.001;
```


# Control Flow Statements
GraphIt supports a variety of control flow constructs, including `if` statements,
`while` loops, `do`-`while` loops and `for` loops.

## If Statements

In GraphIt, a simple `if` statement looks something like this:

```
if x < 1
  print "x is less than 1";
end
```

An `if` statement can optionally include an `else` clause as well as an any
number of `elif` (else-if) clauses:

```
if x < 1
  print "x is less than 1";
elif x > 5
  print "x is greater than 5";
else
  print "x is between 1 and 5";
end
```


## While Loops

A `while` loop in GraphIt looks like this:

```
while x < 100
  x = 2 * x;
end
```

As with `if` statements, logical operators and comparison operators can be used
to construct more complex conditions. Note that if the condition of a `while`
loop is false when a GraphIt program first encounters the loop, the loop body
will not be executed at all.



## For Loops

GraphIt `for` loops are more like those found in MATLAB, Julia and Python than
those available in C. You can use a `for` loop to iterate over the set of all integers between two values, as shown in the following example.

```
for i in 0:10
  print i;
end
```

Note that the lower bound is _inclusive_ while the upper bound is _exclusive_
(like Python), so the above example prints all integers between 0 and 9 but
omits 10.


# Elements, VertexSets, EdgeSets, Vectors
Elements, vertexsets, edgesets and vectors form GraphIt's _data model_.

## Elements
An element is a type that stores one or more data fields, much like a struct in
C/C++. For example, an element representing a point may store a position vector
`x` and a velocity vector `v`, while an element represent a spring may store a
scalar mass:

```
element Point
  x : vector[3](float);
  v : vector[3](float);
end

element Element
  m : float;
  l : float;
end
```

To read from or write to a field of an Element `e` or a Point `p`, you use the
`.` operator:

```
e.m = 2.0;
print e.m;  % 2.0

p.x = [0.0, 0.0, 1.0]';
print p.x;  % [0.0, 0.0, 1.0]'
```

## Sets
Unlike C structs, elements live in sets. So Point elements must be stored in
some set, such as `points`:

```
extern points : set{Point};
```

The `extern` keyword simply means that the `points` set comes from outside the
GraphIt program. Typically they have been assembled using the [GraphIt
C++API](api).

The best ways to work with sets are to
[apply stencil update functions](#apply-stencil-update-functions) and to
[assemble system vectors or matrices](#assemble-system-vectors-and-matrices).

## Edge Sets
Edge sets are sets that also have connectivity information. In particular, edge
set definitions specify the list of sets from which each edge's endpoints come.
The following declares a set of spring elements that each connect two points
from the `points` set:

```
extern springs : set{Element}(points,points);
```

There is no explicit graph type in GraphIt; rather, graphs are formed implicitly
from the combination of sets and edge sets. This is similar to how graphs are
often defined in mathematical papers (i.e. as an ordered pair `G = (V,E)`).

GraphIt's graphs are hypergraphs, which just means that edges can have more (or
less) than two endpoints. More precisely, a GraphIt graph is a _k_-uniform
hypergraphs; in other words, each edge can (and must) connect _k_ vertices,
where _k_ is some non-negative integer constant. Thus, we can declare
additional edge sets that contain triangle, tetrahedral or even hexahedral
elements, as demonstrated below:

```
extern triangles  : set{Element}(points,points,points);
extern tetrahedra : set{Element}(points * 4);
extern hexahedra  : set{Element}(points * 6);
```

The `tetrahedra` and `hexahedra` sets are _homogeneous_ edge sets. This means
that all of their endpoints are elements from the same set. Because of this we
could use a syntactic shortcut to declare their endpoint lists, which freed us
from writing out `point` four or six times.  The two ways to declare
homogeneous edge sets shown above are equivalent.

That said, the more verbose syntax also lets us declare _heterogeneous_ edge
sets, which are edge sets that can connect two or more _different_ sets. For
instance, the `links` edge set below connects a set of triangles to a set of
tetrahedra.

```
extern links : set{Link}(triangles, tetrahedra);
```


# VertexSet Operators

# EdgeSet Operators


A stencil update function is any function that takes as arguments an element
and (if the element is an edge) its endpoints. A stencil update function that
writes to the input element is called a _gather_ (or _pull_) stencil, while a
stencil update function that writes to the endpoints is called a _scatter_ (or
_pull_) stencil. The following stencil function moves a point one unit in the x
direction. The `inout` keyword declares that `p` can be written to.

```
func move(inout p : Point)
  p.x(0) = p.x(0) + 1.0;
end
```

Stencil update functions are applied to every element of a set concurrently
using an `apply` statement. The following statement moves every point in the
`points` set one unit in the x direction:

```
apply move to points;
```

Since `move` only takes a single point as input, the stencil can access just
that one point and therefore has a completely local effect.

A stencil update function that takes an edge from an edge set as input can
access the element as well as the endpoints corresponding to that edge. As an
example, the following stencil takes a spring, computes its length and stores
the length into the `l` field of the corresponding element:

```
func length(inout s : Element,  p : (src : Point, dst : Point))
  s.l = norm(p.dst.x - p.src.x);
end
```

In the function definition above, `p` is a tuple containing the two endpoints
of the spring. To be more precise, `p` is a _named_ (or _heterogeneous_) tuple,
meaning its elements can be accessed by name using the `.` operator. Note that
since all elements of `p` are actually of the same element type, we could have
instead declared `p` as an _unnamed_ (or _homogeneous_) tuple that is indexed
by integral indices using parentheses, as the following equivalent definition
of `length` demonstrates:

```
func length(inout s : Element, p : (Point*2))
  s.l = norm(p(1).x - p(0).x);
end
```

Now to update the lengths of all spring elements in the `springs` set, we again
use an apply statement:

```
apply length to springs;
```

