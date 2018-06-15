---
layout: post
title: GraphIt - A High-Performance Graph Domain Specific Language
---

<meta name="Description" content="GraphIt - A High-
Performance Graph Domain Specific Language">

Simit is a new programming language that makes it easy to compute on sparse
systems using linear algebra.  Simit programs are typically shorter than Matlab
programs yet are competitive with hand-optimized code and also run on GPUs.

With Simit you build a graph that describes your sparse system (e.g. a spring
system, a mesh or the world wide web). You then compute on the system in two
ways: locally or globally. Local computations apply update functions to each
vertex or edge of the graph that update local state based on the vertex or the
edge and its endpoints. This part of the language is similar to what you find
in graph processing framework such as GraphLab and its descendants.

Global computations also start locally with functions applied to each vertex or
edge. However, instead of updating local state the local functions now assemble
a piece of a global vector or matrix. After this assembly, you can compute
using normal linear algebra on the global vectors and matries, and resulting
vectors can be stored back to graph fields. The Simit compiler takes care of
exploiting sparsity and other optimizations, leading to code that is often
simpler than Matlab yet comparable in performance to hand-optimized C++.

Simit graphs are more flexible than what you find in other graph frameworks. A
Simit graph consists of any number of sets. Some of these are vertex sets while
others are edge sets. Both vertex and edge sets can have data fields, but the
edges of an edge set also connects vertices/edges from other sets. Finally,
edges can connect any number of vertices/edges, which means that graphs in
Simit are hypergraphs.

Simit is open source, under a [commercially permissive MIT
license](https://github.com/simit-lang/simit/blob/master/LICENSE). We encourage
you to use it in open source or commercial projects!


