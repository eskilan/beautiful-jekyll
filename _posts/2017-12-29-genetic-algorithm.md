---
layout: post
title: Implementing your own genetic algorithm with Python
subtitle: Taking control
image: /img/dna_orbit_animated_static_thumb.png
gh-repo: eskilan/customGA
gh-badge: [star, fork, follow]
---

Sometimes we need to perform a numerical optimization to minimize or maximize some discontinuous or non-smooth function.  There various options for solving these types of optimization problems:

* EGO: Efficient global optimization, also known as Bayesian optimization
* Particle swarm optimizers
* Genetic algorithms

In python, some optimization libraries already provide these methods: We have scipy’s differential_evolution (https://docs.scipy.org/doc/scipy/reference/optimize.html). We also have the pyOpt libraries (http://www.pyopt.org/) which allow for parallelization using MPI. The problem with these is a lack of control of the parallelization scheme which can hurt our program’s performance severely. This is why a few weeks ago, I was compelled to write my own custom genetic algorithm.

The basic idea behind a genetic algorithm can be better explained by a simple web app called BoxCar2d (http://boxcar2d.com/). If you go to that site you will start to understand how powerful evolution can be in leading to an optimal design. An interesting example of how evolution can lead to optimality can be found in the video below:

[![Bacteria evolving](https://img.youtube.com/vi/plVk4NVIUh8/0.jpg)](https://www.youtube.com/watch?v=plVk4NVIUh8 "Bacteria evolving")

The genetic algorithm consists of the following: First, a population of potential solutions is generated, where each of the solution is represented by a vector of numbers between pre-specified lower and upper bounds . Once the population is created, the fitness (or cost) of each solution is evaluated. We then select a few “elites” (the best solutions) and put them in a list of children.  Then, we perform something called tournament selection. In this step, we randomly select a number of solutions to “fight for the right to reproduce” as in natural selection. The winner of the tournament, which has the highest fitness becomes parent 1. The tournament is performed again, and the winner becomes parent 2. The next step is called crossover, and occurs when parent 1 and parent 2 exchange “genetic material.” By treating each item in a solution vector as a gene, we create “children” solution vectors with genes assigned from either parent randomly.

![alt text](/img/morgan_crossover_2_cropped.png "Thomas Hunt Morgan’s 1916 illustration of a double crossover between chromosomes.")

We repeat this process over and over until either we run out of time or the solution stops improving.

![alt text](/img/generic-pseudocode-of-a-genetic-algorithm.jpg "Image from Rashid, Mahmood & Newton, M A Hakim & Hoque, Md & Sattar, Abdul. (2013). Mixing Energy Models in Genetic Algorithms for On-Lattice Protein Structure Prediction. BioMed research international. 2013. 924137. 10.1155/2013/924137.")

