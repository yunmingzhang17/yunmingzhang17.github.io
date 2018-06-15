---
layout: post
title: Simit - A Language for Physical Simulation
---
Simit: A Language for Physical Simulation
=========================================

{% assign authorids=site.data.papers.tog16.authors %}
{% assign lastauthorid=authorids|last %}
{% assign num_authors=authorids|size %}
{% assign num_authors_minus_1=num_authors|minus: 1 %}
{% for authorid in authorids limit:num_authors_minus_1 %} {% assign author=site.data.authors[authorid] %}[{{ author.name }}]({{ author.site }}), {% endfor %} and 
[{{site.data.authors[lastauthorid].name}}]({{site.data.authors[lastauthorid].site}})

{% if site.data.papers.tog16.image %}
![Simit]({{site.data.papers.tog16.image}})
{% endif %}

## Abstract
With existing programming tools, writing high-performance simulation code is labor intensive and requires sacrificing readability and portability. The alternative is to prototype simulations in a high-level language like Matlab, thereby sacrificing performance. The Matlab programming model naturally describes the behavior of an entire physical system using the language of linear algebra. However, simulations also manipulate individual geometric elements, which are best represented using linked data structures like meshes. Translating between the linked data structures and linear algebra comes at significant cost, both to the programmer and to the machine. High-performance implementations avoid the cost by rephrasing the computation in terms of linked or index data structures, leaving the code complicated and monolithic, often increasing its size by an order of magnitude.

In this article, we present Simit, a new language for physical simulations that lets the programmer view the system both as a linked data structure in the form of a hypergraph and as a set of global vectors, matrices, and tensors depending on what is convenient at any given time. Simit provides a novel assembly construct that makes it conceptually easy and computationally efficient to move between the two abstractions. Using the information provided by the assembly construct, the compiler generates efficient in-place computation on the graph. We demonstrate that Simit is easy to use: a Simit program is typically shorter than a Matlab program; that it is high performance: a Simit program running sequentially on a CPU performs comparably to hand-optimized simulations; and that it is portable: Simit programs can be compiled for GPUs with no change to the program, delivering 4 to 20× speedups over our optimized CPU code.

{% if site.data.papers.tog16.movie %}
<iframe width="500" height="281" src="{{site.data.papers.tog16.movie}}" frameborder="0" allowfullscreen></iframe>
<br/>
{% endif %}

## Files
Paper: [[pdf]]({{site.data.papers.tog16.paper}})

## BibTex
    @article{kjolstad:2016,
     author = {Kjolstad, Fredrik and Kamil, Shoaib and Ragan-Kelley, Jonathan and Levin, David I. W. and Sueda, Shinjiro and Chen, Desai and Vouga, Etienne and Kaufman, Danny M. and Kanwar, Gurtej and Matusik, Wojciech and Amarasinghe, Saman},
     title = {Simit: A Language for Physical Simulation},
     journal = {ACM Trans. Graph.},
     issue_date = {May 2016},
     volume = {35},
     number = {2},
     month = may,
     year = {2016},
     issn = {0730-0301},
     pages = {20:1--20:21},
     articleno = {20},
     numpages = {21},
     url = {http://doi.acm.org/10.1145/2866569},
     doi = {10.1145/2866569},
     acmid = {2866569},
     publisher = {ACM},
     address = {New York, NY, USA},
     keywords = {Graph, matrix, simulation, tensor},
    }
