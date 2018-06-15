---
layout: post
title: The Simit Programming Language 
---
The Simit Programming Language
==============================
{:.no_toc}

This guide introduces Simit language features and shows how they can be used in
programs. We assume that you are already familiar with imperative programming
and some linear algebra.

Simit is a linear algebra language with control flow. The main difference
between Simit and other linear algebra languages, such as Matlab and Julia, is
its [hypergraph data structure](#elements-sets-and-graphs) that is used to
represent sparse systems, its [vector/matrix type
system](#system-vectors-and-matrices) that ties them to the hypergraph, and the
[vector/matrix assembly](#assemble-system-vectors-and-matrices) that lets you
assemble system vectors and matrices from hypergraphs.

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

In fact, we can write the test even more succinctly by chaining comparisons:

```
if 1 <= x <= 5
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

## Do-While Loops

A `do`-`while` loop in Simit looks like this:

```
do
  x = 2 * x;
end while x < 100
```

In contrast to `while` loops, the body of a `do`-`while` loop is guaranteed to
be executed at least once even if the condition is never true.

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


# Vectors and Matrices
Vector and matrix variables have types, and in the simplest case these just
define the size of each dimension:

```
% A column vector of three floats
var row : vector[3](float);

% A row vector of three floats
var col : vector[3](float)';

% A 3 x 3 matrix of floats
var mat : matrix[3,3](float);
```

Vectors and matrices can also be blocked:

```
% A 3-vector with 2-vector blocks
var vec : vector[3](vector[2](float));
```

There are two types of vector/matrix dimensions. The examples above show
vectors and matrices with simple integer range dimensions. These are much like
one- and two-dimensional arrays in languages like C. However, vector and matrix
dimensions can also be Simit sets; this is explained in more detail in the
[System Vectors and Matrices](#system-vectors-and-matrices) section.

## Literals
The syntax for declaring vector literals in Simit is similar to that of MATLAB,
as the following examples illustrate. Note that vector literals are _row-major_
by default; column vector literals can be created by transposing row vector
literals.

```
% A column 3-vector containing the gravity constant (g)
const gravity : vector[3](float) = [0.0, 0.0, 9.8]';

% Same as above, but with comma separators omitted
const gravity : vector[3](float) = [0.0 0.0 9.8]';

% Same as above, but with the type inferred
gravity = [0.0 0.0 9.8]';
```

Simit also supports the MATLAB syntax for declaring matrix literals in addition
to an alternative more regular syntax:

```
% 2 x 2 matrix of integers
mat = [1, 2; 3, 4];

% Same as above, but with comma separators omitted
mat = [1 2; 3 4];

% Same as above, but expressed in alternative syntax
mat = [[1, 2], [3, 4]];

% 2 x 1 matrix of floats
mat21 = [1.0; 2.0];

% 1 x 2 matrix of floats
mat12 = [[1, 2]];
```

## Linear Algebra
Simit supports a wide variety of basic linear algebra operations, as shown
below. (In the following examples, `b` and `c` are column vectors and `r` is 
a scalar.)

```
a = b  + c;   % Vector addition
a = b  - c;   % Vector subtraction
a = b';       % Vector transpose
a = b' * c;   % Inner product
a = b  * c';  % Outer product

A = B + C;    % Matrix addition
A = B - C;    % Matrix subtraction
A = B * C;    % Matrix multiplication
A = B';       % Matrix transpose

a = r  * b;   % Scalar-vector multiplication
a = b  * r;   % Vector-scalar multiplication
A = r  * B;   % Scalar-matrix multiplication
A = B  * r;   % Matrix-scalar multiplication
A = B  * c;   % Matrix-vector multiplication
A = b' * C;   % Vector-matrix multiplication

a = b .* c;   % Vector component-wise multiplication
a = b ./ c;   % Vector component-wise division
A = B .* C;   % Matrix element-wise multiplication
A = B ./ C;   % Matrix element-wise division

x = A \ b;    % Solve Ax = b
```

Linear algebra operations can be composed to form complex expressions. For
instance, the following example multiplies the sum of two matrices `B` and `C`
by the component-wise product of two column vectors `d` and `e`.

```
A = (B + C) * (d .* e);
```

## Indexing
Vectors and matrices can be indexed using parentheses, but unlike MATLAB
indices are zero-based.  Note that initializing a vector or a matrix by a
scalar literal sets every component to the scalar value.

```
var a : vector[2](float) = 0.0;
a(0) = 1.0;  
print a;  % a == [1.0, 0.0]'

var A : vector[2,3](float) = 0.0;
A(0,2) = 1.0;
print A;  % [0.0, 0.0, 1.0; 0.0, 0.0, 0.0]
```

Blocked vectors and matrices can be indexed using multiple parentheses.

```
var a : vector[3](vector[2](float)) = 0.0;
a(1) = [1.0, 2.0]';
a(2)(1) = 3.0;
print a';  % [0.0, 0.0, 1.0, 2.0, 0.0. 3.0]'
```

## Slicing
You can also use the `:` operator to select a single row or column from a
matrix:

```
A : matrix[2,2](float) = [1.0, 2.0; 3.0, 4.0];
print A(:, 1);  % [2.0, 4.0]

```

## Generic Range Dimensions

In all of the examples above, vectors and matrices have range dimensions that
are integer literals. Very often though, you want to define functions that work
with vectors and matrices of arbitrary sizes as opposed to some fixed sizes,
similar to how operators like `+` and `*` can operate on vectors and matrices
of any size. This can be done with generics as demonstrated by the following
function that takes two vectors of any length and returns their element-wise
maximum:

```
func max<0:N>(a : vector[N](int), b : vector[N](int)) 
    -> c : vector[N](int)
  for k in 0:N
    if a(k) >= b(k)
      c(k) = a(k);
    else
      c(k) = b(k);
    end
  end
end
```

The prefix `0:` in the declaration of the generic parameter tells the compiler
that `N` must represent an integer range (as opposed to a Simit set). This
allows us to use `N` anywhere within the function body as a constant integer
whose value is the upper bound of the range represented by the generic
parameter, as we did on the first line of the `for` loop in `max`.

We can now call `max` in the rest of our program with integer-dimensioned
vectors of any length as inputs:

```
a = [0, 2, 4, 6, 8]';
b = [9, 7, 5, 3, 1]';
print max(a, b);  % == [9, 7, 5, 6, 8]'

u = [1, 3, 5]';
v = [2, 3, 4]';
w = max(u, v);  % w == [2, 3, 5]'
```

However, the following program would not compile because `max` can only accept 
two vectors that are of the same length:

```
a = [1, 2, 3]';
b = [5, 6, 7, 8]';
c = max(a, b);
```

Observe that we never had to explicitly specify the value of `N` when calling
`max`, since the compiler can deduce what `N` has to be from the dimensions of
the inputs. Occasionally it might not be possible for the compiler to deduce
what the generic arguments to a function call are. Consider this next example,
which is a function that outputs an arithmetic sequence:

```
func arithSeq<0:N>(a0 : int, d : int)
    -> a : vector[N](int)
  for k in 0:N
    a(k) = a0 + k * d;
  end
end
```

None of the inputs to `arithSeq` has a type that is parameterized by `N`, so in
order to call `arithSeq` the value of `N` has to be explicitly specified as
part of the function call:

```
a = arithSeq<5>(1,2);  % a == [1, 3, 5, 7, 9]'
```

# System Data Structures
So far we have focused on basic constructs and data types. We will now look at
the advanced types that are used to describe the structure of whole physical
systems as well as the _global_ properties of these systems. This is the part
where Simit starts to look and feel very different from other languages like
MATLAB and Julia.

In Simit, the topolog of a system is expressed with a _hypergraph_ data
structure and global properties are described using system vectors and
matrices. A _system_ vector or matrix is a vector or matrix whose dimension
sizes are proportional to the size of the whole system, as opposed to an
_element_ vector or matrix whose sizes are integer ranges.

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

## Assemble System Vectors and Matrices
The key construct that ties sets to system vectors and matrices is the vector
or matrix assembly. An assembly maps a set to a system vector or matrix. Its
semantics is very intuitive and lets the programmer describe system vectors and
matrices as sums of contributions from the elements of a set. An assembly map
expression, like the stencil apply, applies a local stencil function (called an
assembly function) to each element of a set. However, each assembly function
invocation returns a system vector and/or matrix, which are then added together
by the assembly map. That is, an assembly function is a stencil function that
returns one or more system vectors and matrices.

To assemble a force vector, each assembly function call returns a system vector
with the force produced by a single element:

```
func tet_force(tet : Element, p : (Point*4))
    -> f : vector[points](vector[3](float))
  for i in 0:4
    f(p(i)) = compute_tet_force(tet,p,i);
  end
end
```

Note that each `tet_force` invocation only has access to the four points of one
tetrahedra and can therefore only write to four locations in the force vector.
This is a very important constraint, which both prevents indexing errors and
lets the compiler produce fast code since it knows how matrices relate to the
graph. The assumption is that your sparse system only directly interacts
locally. That is, if you want two elements to directly interact through a
matrix then you need to connect them with an edge. (Of course, elements will
often indirectly interact in your time stepper, for example by repeated
matrix-vector multiplications or as a result of a linear solve.)

The assembly function is applied to the elements of the tetrahedra set using an
assembly map expression to assemble the force vector for the whole system:

```
f = map tet_force to tetrahedra reduce +;
```

To assemble a system stiffness matrix each assembly function call returns an
element stiffness matrix, which is a system stiffness matrix with the
contribution of just one element:

```
func tet_stiffness(tet : Element, p : (Point*4))
    -> K : matrix[points,points](matrix[3,3](float))
  for i in 0:4
    for j in 0:4
     K(p(i),p(j)) = compute_tet_stiffness(tet,p,i,j);
    end
  end
end
```

The assembly function is applied to the elements of the tetrahedra set using an
assembly map expression to assemble the stiffness matrix for the whole system:

```
K = map tet_stiffness to tetrahedra reduce +;
```

## Coordinate-Based Access
In
<!-- not supported yet: [applying stencil update functions]
     (#apply-stencil-update-functions) and -->
[assembling system matrices and vectors](#assembly-system-vectors-and-matrices)
using grid edge sets, Simit also supports coordinate-based access. Enabling
coordinate-based access requires supplying a grid edge set using the `through`
syntax. The following demonstrates a vector assembly with grid coordinates:
<!-- not supported yet:
The following demonstrates a stencil update application with grid
coordinates enabled: -->

```
v = map average to points through springsGrid;
```

Using coordinate-based access within the assembly function requires accepting
the grid edge set as additional argument to the function. The underlying node
set may then by indexed by coordinate indices _relative_ to the origin node
(the first argument) using brackets and constant integer offets. The following
demonstrates assembling an average value vector incorporating elements one hop
away in all cardinal directions.

```
func average(p : Point, links : grid[3]{Element}(points))
    -> v : vector[points](float)
  v(p) = (0.167)*(points[1,0,0].val + points[-1,0,0].val +
                  points[0,1,0].val + points[0,-1,0].val +
                  points[0,0,1].val + points[0,0,-1].val);
end
```

The links passed for coordinate-based access not only provide structure but may
also be accessed as part of the stencil function. To access a link, one must
specify both the source and sink of the link in brackets, separated by a
semicolon. The following demonstrates assembling a force vector for a grid of
springs with individual spring constants:

```
func spring_force(p : Point, points : set{Point},
                  springs : grid[2]{Spring}(points))
    -> f : vector[points](vector[3](float))
  force1 = (points[1,0].x - p.x) * springs[0,0; 1,0].k;
  force2 = (points[0,1].x - p.x) * springs[0,0; 0,1].k;
  force3 = (points[-1,0].x  - p.x) * springs[0,0; -1,0].k;
  force4 = (points[0,-1].x - p.x) * springs[0,0; 0,-1].k;
  f(p) = force1 + force2 + force3 + force4;
end
```

This assembly function is applied to the elements of the point set using an
assembly map expression with the `through` keyword:

```
f = map spring_force to points through springsGrid;
```

## Generic Set Dimensions
Generics in Simit also work with vectors and matrices whose dimensions are sets
as opposed to integer ranges. As an example, the following function takes any
system (or even non-system) matrix of any size and finds the largest element in
the matrix (albeit in a suboptimal fashion for sparse matrices):

```
func max<M,N>(A : matrix[M,N](float)) -> b : float
  b = -1.0 / 0.0;  % Initialize b to -inf
  for m in M
    for n in N
      if A(m,n) > b
        b = A(m,n);
      end
    end
  end
end
```

Observe that iterating over a generic set is allowed. However, it is illegal to
access the field of an element of a generic set.

Generics can also be used to define assembly functions that can be mapped to
arbitrary sets (as long as they contain the same types of elements). For
instance, the assembly function `tet_force` from the previous section can be
redefined as follows:

```
func tet_force<S>(tet : Element, p : (Point*4))
    -> f : vector[S](vector[3](float))
  for i in 0:4
    f(p(i)) = compute_tet_force(tet,p,i);
  end
end
```

The above function can then be used to assemble the force vector as before:

```
f = map tet_force to tetrahedra reduce +;
```

Once again, observe that we did not have to explicitly state what set `S`
corresponds to when mapping `tet_force` to the `tetrahedra` set. Since elements
of tuple `p` are used to index into the result vector `f` in the body of
`tet_force`, the compiler would actually be able to deduce that `S` must
correspond to the set that contains the elements of `p`, which in the case of
the assembly map must be the `points` set (the edge set of `tetrahedra`).


# Built-in Functions
The following functions are built into Simit:

## Scalar Math

| Name    | Arguments              | Result      |
| ------- | ---------------------- | ----------- |
| `mod`   | `a : int, b : int`     | `c : int`   |
| `sin`   | `a : float`            | `b : float` |
| `cos`   | `a : float`            | `b : float` |
| `tan`   | `a : float`            | `b : float` |
| `asin`  | `a : float`            | `b : float` |
| `acos`  | `a : float`            | `b : float` |
| `atan2` | `a : float, b : float` | `c : float` |
| `sqrt`  | `a : float`            | `b : float` |
| `log`   | `a : float`            | `b : float` |
| `exp`   | `a : float`            | `b : float` |
| `pow`   | `a : float, b : float` | `c : float` |


## Linear Algebra

| Name      | Arguments                                           | Result                   |
| --------- | --------------------------------------------------- | ------------------------ |
| `det`     | `A : matrix[3,3](float)`                            | `b : float`              |
| `inv`     | `A : matrix[3,3](float)`                            | `A : matrix[3,3](float)` |
| `norm<N>` | `a : vector[V](float)`                              | `b : float`              |
| `dot<N>`  | `a : vector[V](float),` <br> `b : vector[V](float)` | `c : float`              |

## Solvers

| Name          | Arguments                                                                        | Result                 |
| ------------- | -------------------------------------------------------------------------------- | ---------------------- |
| `chol`        | `A : matrix[V,V](float)`                                                         | `solver : opaque`      |
| `cholfree`    | `solver : opaque`                                                                |                        |
| `lltsolve`    | `solver : opaque,` <br> `A : matrix[V,V](float),` <br> `b : vector[V](float)`    | `x : vector[V](float)` |
| `lltmatsolve` | `solver : opaque,` <br> `A : matrix[V,V](float),` <br> `B : vector[V,V](float)`  | `x : vector[V](float)` |

## Complex Math

| Name             | Arguments              | Result        |
| ---------------- | ---------------------- | ------------- |
| `createComplex`  | `r : float, i : float` | `c : complex` |
| `complexNorm`    | `c : complex`          | `n : float`   |
| `complexGetReal` | `c : complex`          | `r : float`   |
| `complexGetImag` | `c : complex`          | `r : float`   |
| `complexConj`    | `a : complex`          | `b : complex` |

