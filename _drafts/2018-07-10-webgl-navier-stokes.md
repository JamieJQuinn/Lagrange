---
title: "GeorgeGL - Part 1 - Simple Convection"
description: "Creating a webgl fluid simulation"
tags:
  - Mathematics
  - Code
---

Years ago I worked my way through Lorena Barba's [12 steps to Navier-Stokes](http://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/) in Python, but recently I've been getting more and more into GPU programming and figured that it would be an interesting exercise to redo the steps in WebGL. Really when I say GPU programming I mean using general purpose tech like CUDA, but CUDA and WebGL are similar enough, plus you get easy, automatic visualisation with WebGL!

## Who Is This For?

Just like Barba has for her course, I'm going to assume everyone reading this has a basic understanding of fluid mechanics, partial differential equations, and numerical methods. You do not have to in any way be an expert in any of these, even just hearing the words at some point in your life should be enough. 

The code will be fairly simple too, there's not too much software engineering that goes into these small numerical experiments, although I'll say now that the amount of code required to set up even a simple WebGL program seems scary. I'll be using a little bit of:

- Javascript - A good basic learning resource is the classic [w3schools](https://www.w3schools.com/jS/default.asp)
- WebGL - Check out <https://webgl2fundamentals.org/> for a good tutorial and reference.
- GLSL - This is the shader language used by WebGL, see <https://webgl2fundamentals.org/> again.

## The Fluid Mechanics

As in Prof. Barba's [first lesson](http://nbviewer.jupyter.org/github/barbagroup/CFDPython/blob/master/lessons/01_Step_1.ipynb) I'll take the 1-dimensional non-linear convection (or advection) equation, where we have a quantity $$u$$ sitting in a fluid that's advected at a speed $$c$$:

$$\frac{\partial u}{\partial t} + c\frac{\partial{u}}{\partial x} = 0.$$

For simplicity we'll only deal with this fluid between $$x=0$$ and $$x=1$$.

The simplest way to discretise these derivatives is to start by splitting our 1D space into $$N_x$$ points, each separated by $$\Delta x=1/N_x$$, and to split our time in a similar way, stepping forward every $$\Delta t$$. Then, the spatial derivative can be approximated as,

$$\left[\frac{\partial{u}}{\partial x}\right]_{x=i\Delta x}^{t=n\Delta t} \approx \frac{u_i^n - u_{i-1}^n}{\Delta x},$$

and the time derivative as,

$$\left[\frac{\partial{u}}{\partial t}\right]_{x=i\Delta x}^{t=n\Delta t} \approx \frac{u_i^{n+1} - u_{i}^{n}}{\Delta t}.$$

There is an entire field of mathematics dedicated to the development and analysis of these approximation of derivatives. This particular method, the first-order upwind method, a form of finite difference method, is a simple one. Although it's not used much in practise due to it not being terribly accurate it's certainly useful for playing about with.

With these approximations at hand, our partial differential equation becomes a finite difference equation,

$$\frac{u_i^{n+1} - u_{i}^{n}}{\Delta t} + c\frac{u_i^n - u_{i-1}^n}{\Delta x} = 0,$$

which can be rearranged to give,

$$u_i^{n+1} = u_i^{n} - c\frac{\Delta t}{\Delta x}(u_i^n - u_{i-1}^n).$$

So, if we know an initial state $$u^0$$ at every point $$i$$, we can work out $$u^1$$, then $$u^2$$, and so forth.

Now it should be quite clear that we've finally found a useful equation, something that ultimately helps us find out how a 1D fluid will behave, as long as we know the initial state $$u^0$$. It should be said our original PDE does actually have a known *analytical* solution, that is a solution we can write as a function $$u(x, t)$$ for any time $$t$$ and at any point in space $$x$$. However, this is a particularly simple case and as we add complexity to the fluid equation, it becomes usually impossible to find anything other than a *numerical* solution through the application of this kind of numerical method.

Coding that up, it looks a little like this:

<div style="text-align: center"><video src="/assets/videos/george-gl/simple-convection.webm" width="300" height="300" autoplay loop preload></video></div>

## The Code

Now technically, I could take that final equation, give myself a starting state, and manually work out every little calculation until I run out of time, food or any kind of semblance of sanity, but I won't because computers exists. I'm not going to go through the code in detail because, frankly, it's not very interesting and mostly copied from [WebGL2Fundamentals](https://webgl2fundamentals.org/webgl/lessons/webgl-image-processing.html) anyway. In short, the code does the following:

1. Setup WebGL context
2. Create rendering surfaces:
    1. Create two textures of simulation size
    2. Link to two framebuffers for rendering
3. Load initial conditions into a texture
4. Run main loop:
    1. Render using main simulation shader from one texture to another
    2. Render boundary conditions
    3. Render result to screen
    4. Swap textures

We're going to represent the grid of points as a texture in WebGL, with one texture representing the state at time $$n-1$$ and another at $$n$$, so by rendering between them and applying our finite difference formula we simulate the fluid. Of course a texture is a 2D grid of points, good for later when we finally move to interesting 2D simulations, but our problem right now is only D, so I'm only going to deal with the $$x$$ direction right now.

The only part that differs greatly from a more typical use of textures is that, because we're trying to run a decently accurate simulation, I've instructed the texture to be created with an internal format of `RGBA32I` which assigns 32 bits per colour channel. Not as hot as the standard of 64 bits in true high performance computing, but good enough for us!

Setting up these textures,the corresponding framebuffers for rendering to them is, and the vertex attribute objects that describe the geometry to be rendered (just a simple square, nothing fancy) is fairly straight forward. In fact, I copied the code almost exactly from [WebGL2Fundamentals](https://webgl2fundamentals.org/webgl/lessons/webgl-image-processing.html).

### The Shaders


