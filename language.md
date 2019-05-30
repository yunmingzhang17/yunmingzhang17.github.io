---
layout: post
title: The GraphIt Programming Language 
---
The GraphIt Programming Language
==============================

{:.no_toc}

This guide introduces GraphIt language features and shows how they can be used in programs. 


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

For vector for vertices, the user can access with a vertexid, in this case `age[v]`. For vector for edges, the user need to access with both src and destinaton vertexids, `edge_vector[src,dst]`.


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

### Utility functions 

* __getVertices(): -> output : vertexset{Vertex}__ This returns a vertexset that is the union of source and destination nodes in the edgeset. 
* __getOutDegrees() -> vector{Vertex}(int):__ This returns a vector of outdegrees for all the vertices in the graph. 
 
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

# Extern Functions

The users can call functions implemented in C++ from GraphIt. These can be used in cases where the user needs some external functionalities that are not currently supported in the GraphIt language. For example, certain solvers or more complex operations that are not necessarily related to graphs. Another example could be a distance estimator used in AStar search. 

The user needs to define a prototype of the function in the GraphIt program with the `extern` keyworkd. The example below defined a extern function named `extern_func` that takes as input a vertex and outputs a double. 
```
extern func extern_func(v: Vertex) -> output:double; 
```

The users can use the extern function inside any function and it can be supplied as arguments to vertexset and edgeset operators. For exmaple, the extern_func can be supplied to the apply operator on a vertexset. 

```
vertexset.apply(extern_func);
```

The users can also use vectors defined in the original GraphIt program inside the extern function. In the C++ definition of the extern programs, the user simply needs to declare an extern variable. The following example shows the definition of an extern function that reads a graphit_vector that is defined in the GraphIt program, and adds the value one to the vector for vertex v.  

```
extern double * graphit_vector;

double extern_func(v: NodeID){
  return graphit_vector[v] + 1;
}
```

To compile GraphIt programs, the user will first need to generate the C++ program with the GraphIt compiler. And later compile the generated file along with the other C++ file with the extern function. 

```
python graphitc.py -f graphit_file.gt -o graphit_generated.cpp 
g++ -std=c++11 -O3 -g -I ../../src/runtime_lib/ graphit_generated.cpp extern_func.cpp -o executable 
```

# Export Functions

The GraphIt compiler generate a stand alone executable with a main function by default. However, the user can also generate a library version of the function that can be used in the Python binding or just a regular C++ function using the export functions.  

To declare an export function, the user simply uses the `export` keyword before the function that is going to be exported. The following example shows the definition of an export function `export_func` that takes as input a graph named `input_edges`.

```
export func export_func(input_edges : edgeset{Edge}(Vertex,Vertex))
  ...
end
```

If the graph is passed in as argument to the exported function, then the vertexsets and vectors cannot be initialized in the const declarations. This is because the size of the vertexset and edgesets are not known at compile time. In this case, the user would need to manually initialize vertexsets and vectors. If the graph is not loaded in as an argument to the export function, then the user can continue to load the graph, initialize the vertexsets and vectors in const declarations. 

Here we show an example to initilize the vertexsets and vectors when the graph is an input argument to the `export_func`.

```
const edges : edgeset{Edge}(Vertex,Vertex);
const vertices : vertexset{Vertex};
const float_vector : vector{Vertex}(float);

func initVector(v : Vertex)
     float_vector[v] = 0.0;
end

export func export_func(input_edges : edgeset{Edge}(Vertex,Vertex))
  edges = input_edges;
  vertices = edges.getVertices();
  float_vector = new vector{Vertex}(float)();
  vertices.apply(initVector);
end

```

# Library Functions 

* __getRandomOutNgh(v: Vertex)__: This function returns a random outgoing neighbor of the Vertex v. 
* __getRandomInNgh(v: Vertex)__: This function returns a random incoming neighbor of the Vertex v. 
* __serialMinimumSpanningTree(graph : edgeset{Vertex, Vertex, int}, start_vertex : Vertex) -> output : vector{Vertex}(int)__: This function computes a serial Minimum Spanning Tree computation on the weighted graph `graph` from the `start_vertex`. It returns a vector of VertexIDs (integers). This vector contains the parent VertexID for each Vertex v. 
* __serialSwepCut(g : edgeset{Edge}(Vertex, Vertex), vset : vertexset{Vertex}, val_array : vector{Vertex}(double)) -> output : vertexset{Vertex}__: This function computes a sweep cut on the vertices based on values supplied with the `val_array`, and returns a subset of the vertexset that belongs to one side of the cut. This is an operator for local graph clustering operations as documented [here](https://arxiv.org/abs/1604.07515).  
* __load(graph_file_name : string) -> output : edgeset{Edge}(Vertex, Vertex)__: This function returns an edgeset loaded from the external file. This can load either an unweighted or weighted file.    
* __startTimer()__: This function starts the timer. 
* __stopTimer()__ -> elapsed_time : float__: This function stops the timer and returns the elapsed time in floats
* __fabs()__: This function returns the absolute value of a float. 
* __atoi()__: This function converts a string to an integer. 

__NOTE__: If you need some library routine that is not included in the list, you can use the [extern functions](language#extern-functions) to provide your customized functionalities. 

# Scheduling Language

In this section, we provide some heuristics for tuning the schedules for improved performance. The scheduling language specified separately can tune the performance of the algorithm. The example below first described the algorithm with edgeset, vertexset, and the `main` function. The user can then use the `schedule:` keyword to mark the beginning of schedule spcification. 

The label `#s1#` is used in the scheduling commands to identify the apply operator the schedule is applied on. The label `#s1` must be specified before the edgeset apply operator if the user intend to tune the performance of the operator. 

```
const edges : edgeset{Edge}(Vertex,Vertex);
const vertices : vertexset{Vertex};
const float_vector : vector{Vertex}(float);
...

func main ()
  ...
  #s1# edges.apply(updateEdge);
  ...
end

schedule:
  program->configApplyDirection("s1", "DensePull");
  ...

```

The full set of schedules are listed in the table below. We refer users to the Section 5 of the [arxiv report](https://arxiv.org/abs/1805.00923) for details of scheduling language. 

<img src="gallery/SchedulingApply.png" alt="Scheduling Functions">


Here are some general guidelines for selecting a set of schedules

* __Step 1:__ Select a direction from SparsePush, DensePush, DensePull, SparsePush-DensePull with `configApplyDirection`. For large social networks that uses a frontier, SparsePush-DensePull is usually a good choice. For PageRank, DensePull is always the best. For road networks, SparsePush often is the best.       
* __Step 2:__ Select a parallelization strategy with `configApplyParallelization`. Usually dynamic-vertex-parallel is the best. For road networks, it might better to switch to static-vertex-parallel. Edge-aware-dynamic-vertex-parallel is only better for PageRank and Collaborative Filtering in some cases.   
* __Step 3:__ Select a layout for Dense Vertex Set with `configApplyDenseVertexset`. if a DensePull direction is used, then you can potentially to use bitvector for the vertexset when the graph has a large number of vertices (currently only working for the pull direction).   
* __Step 4:__ If a DensePull direction is used, then you can use `configNumSSG` (currently only working for the pull direction) to partition the graph for cache efficiency. This is mostly useful for applications that spend a large amount of time processing all the edges (PageRank, PageRankDelta, and Collaborative Filtering). This optimization is usually hurtful for application that only touch a subset of vertices (BFS, SSSP, BC).  Calculations for the number of segments is based on the last level cache (LLC) size.   
* __Step 5:__ `fuseFields` fuses fields that are accessed together into array of structurs to reduce the number of random accesses. Only needed for PageRankDelta so far. 

Some existing schedules 
There are many predfined schedules in the [__input_with_schedules__ directory](https://github.com/GraphIt-DSL/graphit/tree/master/test/input_with_schedules). 
For a specific graph and application, the [script](https://github.com/GraphIt-DSL/graphit/blob/master/graphit_eval/eval/table7/benchmark.py#L16) shows the best schedule. 

In general, the following schedule achieves reasonable perforamnce on social networks (assuming the relevant edgeset apply operator is labeled with `#s1`). 
``` 
schedule:
    program->configApplyDirection("s1","SparsePush-DensePull");
    program->configApplyParallelization("s1", "dynamic-vertex-parallel");
```

# Python Binding

GraphIt provides bindings for the user to load and call GraphIt functions in python. The algorithm with it's schedule can be written in a GraphIt file which can then be compiled and loaded using the GraphIt python module. 
In this section we describe how to build such an interface and how the types from GraphIt translate to python types and vice-versa. 


## GraphIt language extensions

The main difference between writing GraphIt programs and writing functions to be called from python is that GraphIt now compiles as a library instead of an executable program. So it doesn't contain a `main` function. Instead users can define any number of custom functions which can be separately invoked from python. 

To separate internal helper functions from the function which are exposed as a part of the library, we add the `export` keyword. This keyword can be added to any function declaration before the `func` keyword. This tells the compiler that this function is supposed to be a part of the library to be called from python and sufficient wrappers should be generated for the same. 

```
export func do_pagerank(edges: edgeset{Edge}, damp: double) -> ranks: vector{Vertex}(float)
  ...
end
```

Notice how unlike the `main` function which doesn't take any arguments, export functions can take arguments for the graph and the required parameters for the algorithm which can be directly passed from the python code. 

## Type mappings 

The bindings allow user to pass and return python objects over to the GraphIt functions. Python types are translated to GraphIt types and vice-versa using the following mapping. This mapping has been chosen to keep data copy to the minimum to ensure high performance. 
Just like regular python and GraphIt functions, the arguments are passed by reference and any modifications to non scalar types will reflect back in the python program. This feature can also be used if the user wishes to return multiple values. 

### Scalar types mappings

The scalar types in GrapIt directly map to their counterparts in python

| GraphIt type | python type |
|--------------|-------------|
| `int`          | `int`         |
| `float`        | `float`       |
| `long`         | `long`        |
| `double`       | `double`      |
| `bool`         | `bool`        |

GraphIt currently doesn't have a `string` type. Hence `strings` cannot be passed or returned at this point. 


### Non-scalar types mappings

Following non-scalar types are currently supported as arguments and return values. 

| GraphIt type | python type |
|--------------|-------------|
| `edgeset{Edge}` | `scipy.sparse.csr_matrix` |
| `vector{Vertex}(X)` | `numpy.array(dtype=X)` (shape = `(num_vertices)`) | 
| `vector{Vertex}(vector[n](X))` | `numpy.array(dtype=X)` (shape = `(num_vertices, n)`) | 

Here `X` is any scalar type mapped according to the mappings in the previous section. 

## Python module API

To fascilate the user to load GraphIt libraries inside python and call into them we provide a module named `graphit` that can be imported on installed systems as 

```
import graphit
```

This module provides mainly the `compile_and_load` function which returns a `graphit.graphit_module` object. This object has all the functions which are exported from the GraphIt program. 

### `graphit.compile_and_load`

```
graphit.compile_and_load(graphit_source_file, extern_cpp_files=[], linker_args=[], parallelization_type=graphit.PARALLEL_NONE)
```
*Returns `graphit.graphit_module`*

The `compile_and_load` function is the primary function for using the python bindings for graphit. This function compiles the algorithm in the supplied `.gt` file and loads it as a python module with all the exported functions. 

`compile_and_load` takes a compulsory argument `graphit_source_file`. This is the relative/absolute path to the .gt file. The .gt file should contain both the algorithm and schedule if any. The schedule should be specified after the algorithm after the `schedule:` tag. 

This function also takes an optional argument `extern_cpp_files`. If the algorithm needs to be linked with user provided .cpp files, they should be provided here as a list. This argument is necessary when using `extern` functions whose implementation is provided in a .cpp file. The list of files provided here will be compiled with appropriate GraphIt flag and linked with the module before loading. Note, you can only provide .cpp files here. Object files or static and shared libraries should not be provided here. They should be added to the `linker_args`. 

The function also takes another optional argument `linker_args`, which is useful for providing extra arguments to the linker. These arguments are appended to the link command at the very end. Any extra object files or static/shared libraries can also be provided here. You should not provide .cpp files here because this command does not have appropriate compile flags. Source files should be provided with the `extern_cpp_files` argument. 

Finally, the function takes an optional `parallelization_type` argument which can have the value `graphit.PARALLEL_NONE`, `graphit.PARALLEL_CILK` or `graphit.PARALLEL_OPENMP` (default is `graphit.PARALLEL_NONE`. This option allows the user to choose the parallelization runtime to use while compiling the GraphIt program. 

The function returns a `graphit.graphit_module` object that has all the functions exported in the algorithm. 

__Example with linker args__

```
import graphit
pagerank_module = graphit.compile_and_load("pagerank.gt", linker_args=["-lm"])
```

### `graphit.graphit_module`

The `graphit.graphit_module` type object provides an interface to call into exported GraphIt functions. The functions have the exact same name as it had in the GraphIt file. The arguments follow the same name and order. 

```
from scipy.sparse import csr_matrix
my_graph = load_npz("road-usad.npz")

ranks = pagerank_module.do_pagerank(edges=my_graph, damp=0.85)

```
The ranks which is of type `numpy.array(dtype=float)` can now be iterated over and its values used for further processing. 
 


__Example with extern cpp files__
```
import graphit
import scipy.io
from scipy.sparse import csr_matrix
src_add_one_module = graphit.compile_and_load("export_extern_simple_edgeset_apply.gt", extern_cpp_files=["extern_src_add_one.cpp"])

my_graph = csr_matrix(scipy.io.mmread("4.mtx"))
sum_returned = src_add_one_module.export_func(graph)
```

In this example, the `export_extern_simple_edgeset_apply.gt` file declares an `extern` function `extern_src_add_one` which takes a source and destination and performs an operation on the source node's data. This function is called from the `edgeset.apply` operator. But the implementation of this function is not provided in the .gt file. It is provided as a cpp function in the cpp file `extern_src_add_one.cpp`. So we have provide a list containing the name of this file as the argument `extern_cpp_files`. This file is compiled separately when the module is compiled and linked in to the generated graphit module. If this argument is not provided, the compilation will fail with the linker error - *Undefined symbol: `extern_src_add_one`*.  

__Example with the parallelization type flag__

```
import graphit
pagerank_module = graphit.compile_and_load("pagerank.gt", linker_args=["-lm"], parallelization_type=graphit.PARALLEL_CILK)
```

In this example we provide the `parallelization_type` as `PARALLEL_CILK`. This instructs the GraphIt compiler to make use of the CILK runtime while compiling the module. If we had used the graphit.PARALLEL_NONE option (which is the default option), the compiled code would have ran serially. 


