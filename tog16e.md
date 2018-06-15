---
layout: post
title: Why New Programming Languages for Simulation?
---
Why New Programming Languages for Simulation?
=============================================

{% assign authorids=site.data.papers.tog16e.authors %}
{% assign lastauthorid=authorids|last %}
{% assign num_authors=authorids|size %}
{% assign num_authors_minus_1=num_authors|minus: 1 %}
{% for authorid in authorids limit:num_authors_minus_1 %} {% assign author=site.data.authors[authorid] %}[{{ author.name }}]({{ author.site }}), {% endfor %} and
[{{site.data.authors[lastauthorid].name}}]({{site.data.authors[lastauthorid].site}})

## Abstract
Writing highly performant simulations requires a lot of human effort to optimize for an increasingly diverse set of hardware platforms, such as multi-core CPUs, GPUs, and distributed machines. Since these optimizations cut across both the design of geometric data structures and numerical linear algebra, code reusability and portability is frequently sacrificed for performance.

We believe the key to make simulation programmers more productive at developing portable and performant code is to introduce new linguistic abstractions, as in rendering and image processing. In this perspective, we distill the core ideas from our two languages, Ebb and Simit, that are published in this journal.

## Files
Paper: [[pdf]]({{site.data.papers.tog16e.paper}})

## BibTex
    @article{bernstein-kjolstad:2016,
     author = {Bernstein, Gilbert Louis and Kjolstad, Fredrik},
     title = {Why New Programming Languages for Simulation?},
     journal = {ACM Trans. Graph.},
     issue_date = {May 2016},
     volume = {35},
     number = {2},
     month = may,
     year = {2016},
     issn = {0730-0301},
     pages = {20e:1--20e:3},
     articleno = {20e},
     numpages = {3},
     url = {http://doi.acm.org/10.1145/2930661},
     doi = {10.1145/2930661},
     acmid = {2930661},
     publisher = {ACM},
     address = {New York, NY, USA},
     keywords = {Physical simulation languages, compilation, performance},
    }
