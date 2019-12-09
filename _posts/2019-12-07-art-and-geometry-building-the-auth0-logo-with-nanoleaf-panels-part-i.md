---
layout: post
title: "Art and geometry - Building the Auth0 Logo with Nanoleaf Panels - Part I"
date:   2019-12-07 12:42-0800
categories: geometry auth0
comments: true
author: Eugenio Pace
---

A colleague of mine had a really nice backwall light art and I asked him what was it. It turned out to be a design based on [nanoleaf light panels](https://nanoleaf.me/en/consumer-led-lighting/products/smarter-series/nanoleaf-light-panels-smarter-kit/)

Each triangular panel can be connected to build nice patterns. Each panel also can switch colors and it is all programmable.

I figured I could build the Auth0 Logo using these so I ordered a kit. In this post I cover how the basic flat geometry will not work, and what to do to solve it.

### Auth0 Logo Geometry

![](https://cdn.auth0.com/website/press/resources/auth0-glyph.svg)

The Auth0 "shield" is built around a regular pentagon with all sides of the same length. Or you can also build it with 5 paralelograms around a "concave decagon" = a "star". And a paralelogram can be decomposed in 2 triangles. The angles of a star are:

![](https://docs.google.com/drawings/d/e/2PACX-1vS4v9e5v-qmfyaieV43HHAB50vGs5Yo_LAVgFUD8XHQRmoAj2elnwTugP6gKHqLQmXD-VVAEwwF5mEy/pub?w=496&h=289)

### The problem

The problem is that each triangle for the paralelogram we need is **not** equilateral, otherwise the angles would be exactly 60°. We need a way of turning equilateral triangles into exactly what we need. And cutting is not an option for obvious reasons :-).

### Enter 3D!

If you don't limit yourself to a plane, you can turn the triangle into the shape we need:

![](https://docs.google.com/drawings/d/e/2PACX-1vQAL0ijdNO27_G6YlcRDc61O8CFQbghojzm0zF_WBaqNNWbyU_eSKQQPo4MDKCldiCeC9J86sL2QsrX/pub?w=582&h=300)

We need to find the angle β that will make the base 54° (from 60°). 

From above:

![](https://latex.codecogs.com/svg.latex?\inline&space;\large&space;\cos(\beta)&space;=&space;h&space;/&space;L&space;,&space;\tan(54)&space;=&space;2h&space;/&space;L,&space;\cos(\beta)&space;=&space;\tan(54)/2,&space;\beta&space;=&space;\arccos(tan(54)/2))

Which then yields:

![](https://latex.codecogs.com/svg.latex?\inline&space;\large&space;\beta&space;=&space;46.5)

### A prototype

With the above info, I built a simple model using the always awesome [TinkerCAD](https://www.tinkercad.com/) to render an approximation, which looks pretty good:

![](https://docs.google.com/drawings/d/e/2PACX-1vSIRnoD-ANAZKPoftY3sKk0tGRJMIFA-KQaTATyPhm2q6aQYVTs-isGmTOcNRrl8clYQmcCucoskmdr/pub?w=581&h=170)

### Next steps  

Now I need to design the support structure for the Nanoleafs, and figure out how to connect them all.

### Credits

The math formulas are rendered with [Codecogs](https://www.codecogs.com/latex/eqneditor.php)






