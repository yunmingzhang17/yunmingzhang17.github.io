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

The variable can be initialized with the following syntax.

```
var foo : float = 0.0;
```

The `const` keyword creates a variable that cannot be modified after
initialization.  Individual elements of const vectors can still be updated. 

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


# Elements, Vectors, VertexSets, EdgeSets
Elements, vertexsets, edgesets and vectors form GraphIt's _data model_.

## Elements
An element is a type that stores one or more data fields, much like a struct in
C/C++. For example, a vertex in a social network representing a person can have a Person Element type. In the future, we plan to support fields in the Element. Currently, fields associated with an Element are expressed as separate Vectors descrived below. 

```
element Person
end
```

## Vectors

Vectors are associated with an element. Essentially they act as fields of the elements. The following code says that `age` is a vector for `Person` Element type. Each Person would have an associated age field, and the field is initialized to 0. 

```
const age : vector {Person}(int) = 0;

```




## Edgesets
Edgesets are have connectivity information. In particular, edge
set definitions specify the type of elements from which each edge's endpoints come.
The following declares a set of spring elements that each connect two points
from the `edges` set:

```
const edges : edgeset{Follow}(Person,Person);
```

There is no explicit graph type in GraphIt; rather, graphs are formed implicitly
from the edgesets. This is similar to how graphs are
often defined in mathematical papers (i.e. as an ordered pair `G = (V,E)`).



## Vertexsets

Vertexsets are sets of vertices of a specific Element Type. `people` is a vertexset made up of endpoint elements of Person Type from the `edges` edgeset. 

```
const people : vertexset{Person} = edges.getVertices();

``` 

# Set Opeartors 
## Edgeset Operators


### from, to and filter 
`from` and `to` Filters out edges whose source vertex is in the input set. 

```
const people_age_over_40 : vertexset{Person} = ... 
const people_age_over_60 : vertexset{Person} = ...

func main()
  ...
  % find edges between people over 40 years old to people over 60 years old
  var filtered_edges : edgeset{Friend}(Person, Person} = 
       friend_edges.from(people_age_over_40).to(people_age_over_60);
end 

```

`filter` simply supplies a boolean function that checks every edge. 

### srcFilter and dstFilter

`srcFilter` and `dstFilter` filters out edges where the input boolean filtering function `filter_func` returns true. 

```
func filter_func(v : Vertex) -> output : bool
    output =  age[v] > 40;
end

func main()
  ...
  filtered_edges = edges.srcFilter(filter_func);
  ...
end

```

### apply

This operator applies a function `updateEdge ` to every edge. In one iteration of PageRank,

```
func updateEdge(src : Vertex, dst : Vertex)
    new_rank[dst] += contrib[src];
end

func main()
  ...
  edges.apply(updateEdge);
  ...
end
```

### applyModified
This operator applies a function (`updateEdge`) to every edge. Returns a vertexset that contains destination vertices whose entry in the vector has been modified in `updateEdge`. The programmer can optionally disable deduplication within modfieid vertices. Deduplication is enabled by default.

```
func updateEdge(src : Vertex, dst : Vertex)
    parent[dst] = src;
end

func main()
  ...
  edges.applyModified(updateEdge,parent, true);
  ...
end

``` 
 
### combining edgeset operators

The various operators can be and often are chained together. 

```
frontier = edges.from(frontier).to(toFilter).applyModified(updateEdge,parent, true);

```

## Vertexset Operators

### filter
The filter operator is similar to the edgeset filter, except for it is applied on a single vertex and not edge. 

### apply
The apply operator is similar to the edgeset apply operator, but applied to a vertex. 

# Scheduling Language

For now, we refer users to the Section 5 of the [arxiv report](https://arxiv.org/abs/1805.00923) on how to use the scheduling language.  




# Python Binding

## Python Types

## GraphIt 

The programmer uses the "exported" keyword. 

## Python API


