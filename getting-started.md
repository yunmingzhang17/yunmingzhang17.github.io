---
layout: post
title: Getting Started
---
Getting Started
===============
In Simit you compute on graphs using linear algebra on vectors and matrices.
For example, to write a physical simulation you model your physical system as a
graph, assemble global vectors and matrices from this graph, and then compute
on these using linear algebra.  The Simit programming system consists of a
programming language and a C++ API.  The design is similar to the OpenGL API,
with graphs instead of triangle meshes and Simit programs instead of shaders.

[The apps Directory](https://github.com/simit-lang/simit/tree/master/apps/springs)
in the Simit release contains examples of complete Simit programs.  These are
easy to build and should help you get started.

[The Simit Programming Language](language) lets you write programs
to compute on graphs using linear algebra.  It is an imperative language
similar to Matlab, with standard control flow and linear algebra statements.
However, it also has a novel assembly language construct that lets you assemble
global vectors and matrices from your graphs.  This construct is a key to
Simit's simplicity, performance and portability.  It makes it easy to assemble
vectors and matrices without indexing bugs.  It also lets the Simit compiler
and runtime reason about where matrices come from, what similarities they share
and how they relate to graphs, which they use to optimize for performance.

[The Simit C++ API](api) lets you interact with Simit. You use it to
construct graph data structures, to load and compile Simit programs, and to run
them on the graph data structures. Graphs are built from vertex and edge sets
and all set elements can have scalar or dense vector, matrix or higher-order
tensor fields.
