---
layout: post
title: (Original work) Latent space Bayesian Waveform Inversion
date: 2023-07-21 08:00:00-0000
description: Waveform inversion on parsimonious models
tags: InverseProblems
categories: projects
related_posts: false
urlcolor: "blue"
---

### Introduction 
Waveform inversion concerns reconstructing velocity fields from remotely measured wave signals. It has been common practice to optimize for velocity fields parameterized as indicator functions. Considering the ill-posed nature of inversion, the trend is to turn to Bayesian inversion methods, which can be formulated as well posed problems. In addition, Bayesian methods provides one, an estimate of the uncertainity in prediction, which is ever so crucial, should waveform inversion be used for resource sensitive applications.

Despite all evidences pointing towards adopting Bayesian parameter estimation for waveform inversion, it is still not common practice in the FWI community; largely because Bayesian methods are extremely expensive for models with large parameters.

In this work, we use the time honored practice of seeking latent space representations for families of velocity fields. Subsequently we demonstrate that Bayesian waveform inversion in these latent spaces becomes computational tractable.

### Latent spaces

We begin with some observations. More specifically, we consider some velocity fields that are commonplace in the inversion literature -- seismic and otherwise. 

#### Fourier space representations
Initially, we seek Fourier space latent representation of the fields. Initial observations indicate that one can do away with more than 80% of the Fourier bases functions and still obtain a qualitatively good estimate of the velocity field at the cost of mapping to and back from the Fourier space. (This is realized with **fft**.)

##### Primitives

<style>
    .column {
  float: left;
  width: 50.00%;
  margin : 0 0 0px 0px;
  padding: 20px;
}

/* Clear floats after image containers */
.row::after {
  content: "";
  clear: both;
  display: table;
}
</style>

<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/circle.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/square.svg" alt="spline-sur" style="width:100%">
  </div>
</div>


<div class="row">
   <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/2circle.svg" alt="spline-sur" style="width:100%">
    </div>
   <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/circle4.svg" alt="spline-sur" style="width:100%">
  </div>
</div>

##### Velocity classes

<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/simple.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/complex.svg" alt="spline-sur" style="width:100%">
  </div>
</div>


<div class="row">
   <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/Csimple.svg" alt="spline-sur" style="width:100%">
    </div>
   <div class="column">
    <img style="border:1px solid black;" src="/assets/latent/Ccomplex.svg" alt="spline-sur" style="width:100%">
  </div>
</div>

One should note that, for discontinuous functions, high frequency Fourier bases functions become increasingly relevant. While, this was not so debilitating for the examples in the Primitive section, it is clear that seismic velocity fields cannot be robustly parameterized. 

#### Learned latent space representations

This leads us to learning the latent space representations using [variational autoencoders](https://arxiv.org/abs/1906.02691).