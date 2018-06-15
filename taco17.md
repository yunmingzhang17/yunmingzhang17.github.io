---
layout: post
title: The Tensor Algebra Compiler 
---
The Tensor Algebra Compiler
=========================================

{% assign authorids=site.data.papers.taco17.authors %}
{% assign lastauthorid=authorids|last %}
{% assign num_authors=authorids|size %}
{% assign num_authors_minus_1=num_authors|minus: 1 %}
{% for authorid in authorids limit:num_authors_minus_1 %} {% assign author=site.data.authors[authorid] %}[{{ author.name }}]({{ author.site }}), {% endfor %} and 
[{{site.data.authors[lastauthorid].name}}]({{site.data.authors[lastauthorid].site}})

{% if site.data.papers.taco17.image %}
![Simit]({{site.data.papers.taco17.image}})
{% endif %}

## Abstract
Tensor and linear algebra is pervasive in data analytics and the physical sciences. Often the tensors, matrices or even vectors are sparse. Computing expressions involving a mix of sparse and dense tensors, matrices and vectors requires writing kernels for every operation and combination of formats of interest. The number of possibilities is infinite, which makes it impossible to write library code for all. This problem cries out for a compiler approach. This paper presents a new technique that compiles compound tensor algebra expressions combined with descriptions of tensor formats into efficient loops. The technique is evaluated in a prototype compiler called taco, demonstrating competitive performance to best-in-class hand-written codes for tensor and matrix operations.

{% if site.data.papers.taco17.movie %}
<iframe width="500" height="281" src="{{site.data.papers.taco17.movie}}" frameborder="0" allowfullscreen></iframe>
<br/>
{% endif %}

## Files
Paper: [[pdf]]({{site.data.papers.taco17.paper}})

## BibTex
    @techreport{kjolstad17:tensor-compiler-techreport,
     author={Fredrik Kjolstad, Shoaib Kamil Stephen Chou, David Lugato and Saman Amarasinghe},
     title={{The Tensor Algebra Compiler}},
     institution={Massachusetts Institute of Technology},
     year={2017},
     month={February},
     publisher={CSAIL},
     source={http://hdl.handle.net/1721.1/107013},
     keywords = {Tensor, tensor algebra, linear algebra, C++ library}
   }
