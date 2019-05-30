---
layout: post
title: Python Binding Getting Started
---
Getting Started with Python Binding
===============
{:.no_toc}
## Overview
This is a tutorial for getting started with GraphIt and using the python bindings to compile GraphIt programs as a library and call it from python. If you are looking for a tutorial on building GraphIt programs as standalone application, you can visit the [Standalone Getting Started](getting-started) tutorial. 

In this tutorial we will write the PagerankDelta algorithm in GraphIt and run it from python on a graph loaded in python. We will also measure the performance of this algorithm from python. We will learn how to separate initialization from actual execution to get appropriate timing numbers. 
This tutorial is broadly devided in the following sections - 

* auto-gen TOC:
{:toc}
 
## Downloading software
Make sure you have all the correct Open Source Software installed. First follow the [README](https://github.com/yunmingzhang17/graphit) file here to clone and install graphIt. You will need either CILK or OPENMP to allow you to run the C++ code in parallel. If you dont have either you can get both by simply downloading [GCC](https://gcc.gnu.org/). Alternatively if you already have CILK or OPENMP you can use those too. This tutorial will go through how to use GraphIt via both CILK and OPENMP.
        
## Cloning graphit
Clone graphit by going to [GraphIt](https://github.com/yunmingzhang17/graphit)
        *Something to note for the following tutorial.*
        *Everything will be done graphit/build/bin*
   
## Basic Variables, Constructs, and Functions

If you have not yet already please read the basic information on the [GraphIt Language.](language)

## PageRankDelta Example in GraphIt language

```
element Vertex end
element Edge end
const edges : edgeset{Edge}(Vertex,Vertex);
const vertices : vertexset{Vertex};
const cur_rank : vector{Vertex}(float);
const ngh_sum : vector{Vertex}(float);
const delta : vector{Vertex}(float);
const out_degree : vector {Vertex}(int);
const damp : float = 0.85;
const beta_score : float;
const epsilon2 : float = 0.1;
const epsilon : float = 0.0000001;
const init_delta: float;

func updateEdge(src : Vertex, dst : Vertex)
    ngh_sum[dst] += delta[src]/out_degree[src];
end

func updateVertexFirstRound(v : Vertex) -> output : bool
    delta[v] = damp*(ngh_sum[v]) + beta_score;
    cur_rank[v] += delta[v];
    delta[v] = delta[v]-1.0/edges.getVertices();
    output = (fabs(delta[v]) > epsilon2*cur_rank[v]);
    ngh_sum[v] = 0;
end

func updateVertex(v : Vertex) -> output : bool
   delta[v] = ngh_sum[v]*damp;
   cur_rank[v] += delta[v];
   ngh_sum[v] = 0;
   output = fabs(delta[v]) > epsilon2*cur_rank[v];
end

func initVectors(v : Vertex)
    cur_rank[v] = 0.0;
    ngh_sum[v] = 0.0;
    delta[v] = init_delta;
end


export func set_graph(edges_args: edgeset{Edge}(Vertex, Vertex))
    edges = edges_args;
    vertices = edges.getVertices();
    cur_rank = new vector{Vertex}(float)();
    ngh_sum = new vector{Vertex}(float)();
    delta = new vector{Vertex}(float)();
    out_degree = edges.getOutDegrees();
    beta_score = (1.0 - damp) / edges.getVertices();
    init_delta = 1.0 / edges.getVertices();
    vertices.apply(initVectors);
end

export func do_pagerank_delta() -> final_ranks: vector{Vertex} (float)
    var n : int = edges.getVertices();
    var frontier : vertexset{Vertex} = new vertexset{Vertex}(n);
    for i in 1:10
        #s1# edges.from(frontier).apply(updateEdge);
        var output : vertexset{Vertex};
        if i == 1
           output = vertices.filter(updateVertexFirstRound);
        else
           output = vertices.filter(updateVertex);
        end
        delete frontier;
        frontier = output;
    end
    delete frontier;
    final_ranks = cur_rank;
end
```

*Page Rank Delta in Graphit*

Here we will go through an example of writing GraphIt code for the Page Rank Delta application and then calling it from python using the python bindings. You can find the code used in this example along with a few other applicaitons under graphit/apps directory [here](https://github.com/yunmingzhang17/graphit/tree/master/apps). 

Additionally here is a link to the [GraphIt OOPSLA18 paper](https://dl.acm.org/citation.cfm?id=3276491) or [the arxiv report here](https://arxiv.org/pdf/1805.00923.pdf).  Sections 4 and 5 give the complete breakdown of the Page Rank Delta code. Please look here if you want a more detailed breakdown of the functionality of Graphit.

###      Algorithm Explanatation
```
element Vertex end
element Edge end
```

_element definitions_

Here we construct the basic Elements, `Vertex` and `Edge` with the `element` keyword that will be used by graphit. Note these basic Elements do not have to be named "Vertex" or "Edge". Most Graph Algorithms will require that you have both of these. GraphIt supports multiple types of user-defined vertices and edges, which is important for algorithms that work on multiple graphs.

[A quick refresher on Variables](http://graphit-lang.org/language#variables)

```
const edges : edgeset{Edge}(Vertex,Vertex);
const vertices : vertexset{Vertex};
```

*vertexset and edgeset definitions*
Once we have declared the basic element types, we can use these to construct the vertexsets and edgesets. These two lines declare these edgeset, `edges` and the vertexset `vertices`. Each element of the edgeset is of `Edge` type (specified between "{ }"), and the source and destination of the edge is type `Vertex`. The source and destinations can also be of different types (for instance in case of a bipartite graph). Here we have just declared these two sets but haven't initialized them to anything. This is because these sets will be created using the graph passed to GraphIt from the python code. 
We have added the `const` keyword before the declarations to indicate these vertexsets and edgesets are globally accessible. 

```
const cur_rank : vector{Vertex}(float);
const ngh_sum : vector{Vertex}(float);
const delta : vector{Vertex}(float);
const out_degree : vector {Vertex}(int);
```
*vector definitions*

Data for vertices and edges are defined as vectors associated with the element type denoted using the `{ }` syntax. For example, `cur_rank` is associated with `Vertex`, and is of type `float` (specified in with the `()` syntax). This is similar to fields of a struct in C or C++, but is stored as a separate vector. 

Notice again, that we have only delcared these vectors and haven't allocated them or initialized their data because the size and values of these will depend on the Graph passed in. We will be explicity initializing them in the in the `set_graph` function. 



```
const damp : float = 0.85;
const beta_score : float;
const epsilon2 : float = 0.1;
const epsilon : float = 0.0000001;
const init_delta: float;
```

*scalar constants declaration* 

Next we declare the constants and scalar variables required for the algorithm. We also initialize those scalars that have values independent of the graph (for example the `damp` float). We will initialize the variables that depend on the graph in the `set_graph` function. 

Next we move on to take a look at the functions used in the program. 

[A quick refresher on Functions](language#functions)

The algorithm uses three user-defined functions, `updateEdge`, `updateVertexFirstRound`, and `updateVertex`. 

```
func updateEdge(src : Vertex, dst : Vertex)
    ngh_sum[dst] += delta[src]/out_degree[src];
end
```

`updateEdge` takes in `src` and `dst` of an edge as arguments. The function adds to the current __ngh_sum__ of the destination vertex `dst`, the `delta` divided by the `out_degree` of the source vertex `src`. This function is later used in the main function and applied to every edge in the edgeset. 


```
func updateVertexFirstRound(v : Vertex) -> output : bool
    delta[v] = damp*(ngh_sum[v]) + beta_score;
    cur_rank[v] += delta[v];
    delta[v] = delta[v]-1.0/edges.getVertices();
    output = (fabs(delta[v]) > epsilon2*cur_rank[v]);
    ngh_sum[v] = 0;
end
```

`updateVertexFirstRound` takes in a vertex, `v`, and returns a boolean. It does this by multiplying the `ngh_sum` with the damping factor and adding the basescore. From this it computes the rank and using the delta it computes whether or not it exceeds a certain threshold. If this threshold is exceeded than it returns a boolean True and if not a boolean False. Then it sets the `ngh_sum` back to 0.

```
func updateVertex(v : Vertex) -> output : bool
   delta[v] = ngh_sum[v]*damp;
   cur_rank[v] += delta[v];
   ngh_sum[v] = 0;
   output = fabs(delta[v]) > epsilon2*cur_rank[v];
end
```

`updateVertex` also takes in a vertex and returning a boolean. However in this case it does not add the base score to `delta` when determining Delta. Similarly then by comparing if the delta exceeded the threshold of epilson times the rank it outputs a True or False. 

`updateVertexFirstRound` and `updateVertex` will be used later on to filter out the "active vertices". These are the vertices that will used in the next iteration of the algorithm. These active vertices are also known as the frontier. The reason for two functions is that the first time we update the vertexs some additional computation needs to be done as described above that isn't needed later on. Therefore the second function is run only once in the beginning of the algorithm.


```
func initVectors(v : Vertex)
    cur_rank[v] = 0.0;
    ngh_sum[v] = 0.0;
    delta[v] = init_delta;
end


export func set_graph(edges_args: edgeset{Edge}(Vertex, Vertex))
    edges = edges_args;
    vertices = edges.getVertices();
    cur_rank = new vector{Vertex}(float)();
    ngh_sum = new vector{Vertex}(float)();
    delta = new vector{Vertex}(float)();
    out_degree = edges.getOutDegrees();
    beta_score = (1.0 - damp) / edges.getVertices();
    init_delta = 1.0 / edges.getVertices();
    vertices.apply(initVectors);
end
```

*initialization functions*

We now create the functions that will initialize the vectors and other variables that depend on the graph. Firstly, the `set_graph` function. Notice this function is marked with the `export` keyword, which means that this function is to be called from python and should be a part of the module loaded into python. This function takes 1 argument, which is the edgeset passed from the python code (as `scipy.sparse.csr_matrix`) and does not return anything. 
You can read more about the `export` keyword and how the GraphIt types interface with the python types in the [export section](language#export-functions) of the language manual. 

The first thing we initialize is the edgeset `edges` by assigning the passed in argument to it. We then initialize the vertexset `vertices` by using the `getVertices()` function from `edges`. This function returns the vertices from the graph passed in as argument. 

We then allocate memory for the vectors associated with the vertices with  `new Vector{Vertex}(float)()`. This is okay to do now because the number of elements is known. We do the same for the variables `beta_score` and `init_delta`. The vector `out_degree` is initialized by calling the `getOutDegrees()` function from `edges`. 

We initialize the values for the vectors by using a user defined function `initVectors`. This function takes in an argument the vertex to initialize and sets the initial value of the vectors for that vertex. We invoke this function on all the vertices in the `set_graph` function by using the `vertexset.apply` operator. 

You should note here that we have created a separate `set_graph` function to isolate it from the actual algorithm in the `do_pagerank_delta` function. We do this so we can measure the performance of the actual algorithm and not the cost of initializing the graph related datastructures. If you are not timing the execution of the algorith, it is perfectly fine to add an arguement to the main `do_pagerank_delta` function and do all the initializations there. 

Now, all the data structures are initialized and we are ready to actuall perfom the pagerank_delta operation. 

```
export func do_pagerank_delta() -> final_ranks: vector{Vertex} (float)
    var n : int = edges.getVertices();
    var frontier : vertexset{Vertex} = new vertexset{Vertex}(n);
    for i in 1:10
        #s1# edges.from(frontier).apply(updateEdge);
        var output : vertexset{Vertex};
        if i == 1
           output = vertices.filter(updateVertexFirstRound);
        else
           output = vertices.filter(updateVertex);
        end
        delete frontier;
        frontier = output;
    end
    delete frontier;
    final_ranks = cur_rank;
end
```

*main function of PageRankDelta*

The `main` function is where your program comes together and runs together with all the user-defined functions. Similar to C and C++, you have to explicitly name the function `main`. 

GraphIt is designed to separate edge processing logic from edge traversal, edge filtering (from, to, srcFilter, and dstFilter), atomic synchronization, and modified vertex deduplication and tracking logic (apply and applyModified). This separation enables the compiler to represent the algorithm from a high level, exposing opportunities for edge traversal and vertex data layout optimizations. Moreover, it frees the programmer from specifying low-level implementation details, such as synchronization and deduplication logic. 

The algorithm iterates for 10 iterations to update each vertex's `cur_rank` value. The `cur_rank` is assumed to converge after 10 iterations, and represents the importance of each vertex based on its topological structure. In each iteration, the algorithm maintains the set of vertices whose rank has changed greatly from previous iterations, which is known as the `frontier`. We start with having all vertices in the frontier with frontier initalization ( `var frontier : vertexset{Vertex} = new vertexset{Vertex}(n);` ). The frontier generated by   
`output = vertices.filter(updateVertexFirstRound)`  
in the first iteration, which applies the `updateVertexFirstRound` function to all the `vertices`. In later iterations the frontier is generated by   
`output = vertices.filter(updateVertex)`.   
The user has to explicitly manage the memory and swapping of the frontier with `delete frontier; ` and `frontier = output;`

In each iteration we update the `delta` values using the `updateEdge` function in `#s1# edges.from(frontier).apply(updateEdge); `. We use the operator `from` to obtain the set of edges that comes out from the `frontier`. Then we use `apply `to apply the `updateEdge` function on these edges. The label `#s1` is used as a way to identify the edgeset operator for performance tuning using the scheduling. 

The vector `cur_rank` is returned by assigning it to the `final_ranks` variable. 

##  Scheduling Explanatation
To learn how to use the scheduling functions with GraphIt, please visit the [scheduling section](getting-started#performance-tuning-with-the-scheduling-language) of the main getting-started document. 

## Using GraphIt code from Python. 

Before you can run the code above you need to first follow these steps and build GraphIt

### Build Graphit
Before we start building GraphIt we need to make sure we have all the dependencies installed. Currently GraphIt requires the following packages to be installed - 
1. cmake (version >= 3.5)
2. g++ (version >= 5.4)
3. python 2.7 (for running test cases)
4. python 3.x (x >= 5, for python bindings)
5. pybind11
6. scipy 


To perform an out-of-tree build of Graphit do:

After you have cloned the directory:
```
cd graphit
mkdir build
cd build
cmake ..
make
```
To run the C++ test suite do (all tests should pass):
```
cd build/bin
./graphit_test
```
To run the Python end-to-end test suite:

start at the top level graphit directory cloned from Github, NOT the build directory
(All tests would pass, but some would generate error messages from the g++ compiler. This is expected.)
Currently the project supports Python 2.x for the basic tests and Python 3.x (x >= 5) for the pybind test cases
``` 
cd build
python2 python_tests/test.py
python2 python_tests/test_with_schedules.py
export PYTHONPATH=.
python3 python_tests/pybind_test.py 
```

### Invoke GraphIt program from python
We are now ready to invoke the GraphIt program we wrote above from python 3.x (x >= 5). We start by writing the following python program. 

```
import graphit
import scipy.io
from scipy.sparse import csr_matrix
import sys
import time

module = graphit.compile_and_load("pagerank_delta_export.gt")
graph = csr_matrix(scipy.io.mmread(sys.argv[1]))
module.set_graph(graph)
start_time = time.perf_counter()
ranks = module.do_pagerank_delta()
end_time = time.perf_counter()

print ("Time elapsed = " + str(end_time - start_time) + " seconds")
```

We start by importing the `graphit` module. This module has tall the helper functions to compile and invoke Graphit functions. We also import the supporting libraries required to load the graph and time the computation. 

We then ask the graphit module to load the GraphIt program we wrote using the `compile_and_load` function. We pass to it the path of the file we wrote above. This function returns a module that has all the functions we had exported form the GraphIt file. Notice that for the path, we have just provided `"pagerank_delta_export.gt"` because the file is in the same directory. But you should provide the full path here if it is in a different directory. 

Next we laod the graph in the `scipy.sparse.csr_matrix` format. Assuming the graph is stored in the MatrixMarket format, we use the `scipy.io.mmread` function which returns the graph in the `scipy.sparse.coo_matrix` format. We convert it to the `csr_matrix` format and store it in the `graph` variable. We will be specifying the input graph file name at run time and hence we have used the `sys.argv[1]` as input file name.

We then pass this graph to the pagerank_delta program by invoking the `set_graph` function we had defined. Finally we invoke the `do_pagerank_delta` function and save its return value in the ranks. We also time the call to this function by calling the `time.perf_counter()` function before and after the function call and substracting the time. 

### Running the python program

Save the above python program in a file named pagerank_delta.py

We are ready to run this program using python3. We can invoke the command - 

``` 
python3 pagerank_delta.py <path/to/graph.mtx>
```

If everything runs properly, this will output the time elapsed for computation. 

### Fix if python3 cannot find graphit

The build directory of Graphit is added to your PYTHONPATH already, but just in case the above command gives an error messsage like - 
```
ImportError: No module named 'graphit'
```

It means that python3 cannot find the `graphit` module. You can point to it using the command - 

```
export PYTHONPATH=<path/to/graphit/build/directory>
```
and then running the command again - 

```
python3 pagerank_delta.py <path/to/graph.mtx>
```


## Code examples
All the code used in this tutorial is available at the following urls
1. [pagerank_delta_export.gt](https://github.com/GraphIt-DSL/graphit/tree/master/apps/pagerank_delta_export.gt)
2. [pagerank_delta.py](https://github.com/GraphIt-DSL/graphit/tree/master/apps/pagerank_delta.py)

To reproduce these examples, you can just navigate to the [apps](https://github.com/GraphIt-DSL/graphit/tree/master/apps) directory and run the python command given below. Some example graphs are also provided in the [test/graphs](https://github.com/GraphIt-DSL/graphit/tree/master/test/graphs) directory. 

```
python3 pagerank_delta.py ../test/graphs/4.mtx
```
