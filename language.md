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
Simit is an imperative language with statements, control flow and linear
algebra expressions. In this section, we describe some of the language's basic
constructs.

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

Like functions in MATLAB, Simit functions can return any number of results,
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

Exported functions can be called from C++ code through the [Simit C++
API](api). They are declared by prepending the function declaration with the
keyword `export`, as demonstrated below. Note that even though `main` does not
take any parameter, an (empty) parameter list must still be included in the
function declaration. Exported functions typically work on extern data
structures in global scope.

```
export func main()
  % Do something here
end
```

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

Note that variables are const by default! That is, variables that are declared
without the `var` or `const` keywords are treated as const.

## Basic Types
Simit is statically typed with type inference. This means that the type of
every variable is known at compile time, but that you do not always have to
explicitly specify it; the compiler will figure most out automatically.

The basic numeric types in Simit are `bool`, `int`, `float` and `complex`:

```
var mybool    : bool    = false;
var myint     : int     = 0;
var myfloat   : float   = 0.0;
var mycomplex : complex = <0.0, 0.0>;
```

The `float` and `complex` types are double-precision floating-point by default
when you use the CPU backend, but this can be changed to single-precision
floating-point when compiling a Simit program. The GPU backend currently only
supports single-precision floating-point `float` and `complex` types.

Simit also supports a `string` type that can be used for I/O and debugging. The
following example declares a variable containing the string "hello, world\n":

```
var mystr : string = "hello, world\n";
```

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
Simit supports a variety of control flow constructs, including `if` statements,
`while` loops, `do`-`while` loops and `for` loops.

## If Statements

In Simit, a simple `if` statement looks something like this:

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

Simit supports the following logical operators and comparison operators, which
can be used to construct more complex conditions:

```
x or y;   % Logical OR
x and y;  % Logical AND
x xor y;  % Logical XOR
not x;    % Logical NOT

a == b;   % Equality
a != b;   % Inequality
a < b;    % Less than
a <= b;   % Less than or equal to
a > b;    % Greater than
a >= b;   % Greater than or equal to
```

For example, if we only cared about the scenario in which the value of `x` is
between 1 and 5, then we could have simply written:

```
if x >= 1 and x <= 5
  print "x is between 1 and 5";
end
```


## While Loops

A `while` loop in Simit looks like this:

```
while x < 100
  x = 2 * x;
end
```

As with `if` statements, logical operators and comparison operators can be used
to construct more complex conditions. Note that if the condition of a `while`
loop is false when a Simit program first encounters the loop, the loop body
will not be executed at all.



## For Loops

Simit `for` loops are more like those found in MATLAB, Julia and Python than
those available in C. They can be used to iterate over elements in a Simit set.
For example (pun intended):

```
for p in points
  % Do some computation with element p in points set
end
```

You can also use a `for` loop to iterate over the set of all integers between
two values, as shown in the following example.

```
for i in 0:10
  print i;
end
```

Note that the lower bound is _inclusive_ while the upper bound is _exclusive_
(like Python), so the above example prints all integers between 0 and 9 but
omits 10.


## Elements, Sets and Graphs
Elements, sets and graphs form Simit's _data model_, which is the way you
represent your system.

### Elements
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

### Sets
Unlike C structs, elements live in sets. So Point elements must be stored in
some set, such as `points`:

```
extern points : set{Point};
```

The `extern` keyword simply means that the `points` set comes from outside the
Simit program. Typically they have been assembled using the [Simit
C++API](api).

The best ways to work with sets are to
[apply stencil update functions](#apply-stencil-update-functions) and to
[assemble system vectors or matrices](#assemble-system-vectors-and-matrices).

### Edge Sets
Edge sets are sets that also have connectivity information. In particular, edge
set definitions specify the list of sets from which each edge's endpoints come.
The following declares a set of spring elements that each connect two points
from the `points` set:

```
extern springs : set{Element}(points,points);
```

There is no explicit graph type in Simit; rather, graphs are formed implicitly
from the combination of sets and edge sets. This is similar to how graphs are
often defined in mathematical papers (i.e. as an ordered pair `G = (V,E)`).

Simit's graphs are hypergraphs, which just means that edges can have more (or
less) than two endpoints. More precisely, a Simit graph is a _k_-uniform
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

### Grid Edge Sets
Simit also supports declaring edge sets which connect points in a regular
grid pattern with a particular dimensionality. The following declares a
three-dimensional grid over the `points` set, meaning it contains links
connecting each point to six other points (in the six cardinal directions
one can move for a three-dimensional grid):

```
extern springsGrid : grid[3]{Element}(points);
```

While one could build such a structure using a generic edge set, specifying this
structure enables [coordinate-based accessing](#coordinate-based-access) when
assembling matrices based on grid edge sets.

Because these edge sets have very particular structure, extern grid edge sets
are assembled with different syntax in the [Simit C++ API](api).

## Apply Stencil Update Functions

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

## System Vectors and Matrices
In addition to integer ranges, the dimensions of vectors and matrices can be 
_sets_. We call such vectors and matrices _system_ vectors and matrices, and 
they can describe properties of an entire physical system. A system vector can 
be thought of as a dictionary, while a system matrix can be thought of as a
two-dimensional dictionary (or a dictionary of dictionaries):

```
% A vector with one float per point in the points set
var a : vector[points](float);

% A vector with one 3-vector block per point 
% in the points set
var b : vector[points](vector[3](float));

% A matrix with one 3x3 matrix block non-empty 
% per pair of points in the points set
var K : matrix[points,points](matrix[3,3](float));
```

Note that system matrices can be sparse matrices and do not have to store 
values for all |`points`|<sup>2</sup> combinations of indices.

In the [Elements](#elements) section, we showed how an element field can be 
accessed using the `.` operator. Sets also have fields, corresponding to the 
fields of the set elements, that can be accessed using the same syntax. The 
result of a set field read is a system vector that is constructed by 
concatenating the corresponding fields of all elements in the set. For example, 
if set `springs` contains elements `e0`, `e1` and `e2` (in that order) and if 
each element has a field `m` of type `float`, then `springs.m` would correspond 
to the system vector 

```
[e0.m, e1.m, e2.m]'
``` 

and would be of type `vector[springs](float)`.

