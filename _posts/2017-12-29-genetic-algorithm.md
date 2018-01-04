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

We will now explore how this algorithm can be put into code with python. The first thing we will need is importing numpy. I also imported the “random” library for convenience.

```python
import random
import numpy as np
```
Then we define a class for the optimizer, which in this case I called “customGA.” The following is the class declaration and the initialization member function:
````python
class customGA(): 
    # x0, lb, ub are numpy arrays 
    def __init__(self, costFunc, x0, lb, ub, popSize, numElites, kSelect, mutationP, maxGen): 
        self.costFunc = costFunc 
        self.oldGenList = [] 
        self.newGenList = [] 
        self.fitnessList = [] 
        self.maxGen = maxGen 
        # initial guess 
        self.x0 = x0 
        # upper bound 
        self.ub = ub 
        # lower bound 
        self.lb = lb 
        # population size 
        self.popSize = popSize 
        # number of elites 
        self.numElites = numElites 
        # number for tournament selection 
        self.kSelect = kSelect 
        # mutation percentage 
        self.mutationP = mutationP 
        # current generation counter 
        self.currentGen = 1
        # create initial population 
        # first element will be x0 
        self.oldGenList.append(self.x0) 
        for individualIndex in range(1,popSize): 
            # skipping first # append random vector between lb and ub 
            self.oldGenList.append(np.random.uniform(lb,ub)) 
            # set children equal to parents 
            self.newGenList = self.oldGenList 
````

We then create a member function to evaluate the fitness of the population of potential parents:

````python
def evalFitness(self):
  # clear fitness
  self.fitnessList = []
  for individual in self.newGenList:
      cost = self.costFunc(individual)
      self.fitnessList.append(cost)
````

We then create another member function to select the parents. This function is called tournamentSelection():

````python
def tournamentSelection(self):
    k = self.kSelect
    # select k, since list is already sorted lowest index wins
    # initialize choices list
    choicesSet = set(range(0,self.popSize))
    kSet = random.sample(choicesSet,k)
    minK1 = min(kSet)
    choicesSet.remove(minK1)
    # select again
    kSet = random.sample(choicesSet,k)
    minK2 = min(kSet)
    choicesSet.remove(minK2)
    # return arrays with indices minK1 and minK2
    arrayList = []
    arrayList.append(self.oldGenList[minK1])
    arrayList.append(self.oldGenList[minK2])
    
    return arrayList
````

We then implement the crossover operation. We use a vector of ones and zeros assigned randomly to select the genes from the parents:

````python
def crossOver(self, parentsList):
    arr1 = parentsList[0]
    arr2 = parentsList[1]
    # create array of zeros
    arr3 = np.zeros(arr1.size)
    # fill out arr3 with values from arr1 and arr2
    decisionArray = np.random.choice([0, 1], size=(arr1.size,))
    arr3 = decisionArray*arr1+(1-decisionArray)*arr2
````

The last component is adding mutations which we add as a prescribed percentage of the range of each item in the solution vector defined by the upper and lower bounds:

````python
def mutate(self, child):
    # Min and max bounds from mutation percentage
    maxMod = child + 0.5/100.0*self.mutationP*(self.ub-self.lb)
    minMod = child - 0.5/100.0*self.mutationP*(self.ub-self.lb)
    # Filtering values that exceed lb and ub
    modUB = np.minimum(self.ub,maxMod)
    modLB = np.maximum(self.lb,minMod)
    # mutating the vector
    mutant = np.random.uniform(modLB,modUB)
    
    return mutant
````

These methods, are put together into an “evolve” function:

````python
def evolve(self):
    # assemble fitnessList and newGenList together into list of tuples
    organizedPopList = []
    for i in range(self.popSize):
        organizedPopList.append((self.newGenList[i],self.fitnessList[i]))
    # sort populating using fitness values
    organizedPopList = sorted(organizedPopList, key=lambda x: x[1], reverse=True)
    # update member according to organized list
    self.newGenList = []
    self.fitnessList = []
    for i in range(self.popSize):
        self.newGenList.append(organizedPopList[i][0])
        self.fitnessList.append(organizedPopList[i][1])
    # move children to parents
    self.oldGenList = list(self.newGenList)
 
    # empty children list
    self.newGenList = []
 
    # select numElites elites individuals to retain
    for i in range(self.numElites):
        self.newGenList.append(self.oldGenList[i])
 
    for i in range(self.numElites, self.popSize):
        parentsList = self.tournamentSelection()
 
        child = self.crossOver(parentsList)
 
        mutantChild = self.mutate(child)
 
        self.newGenList.append(mutantChild)
````

Lastly, the algorithm is called from a simple function called run():
````python
def run(self):
    while self.currentGen < self.maxGen:
        self.evalFitness()
        print('Generation:',str(self.currentGen),' ','Best result:', str(self.fitnessList[0]))
        self.evolve()
        self.currentGen = self.currentGen + 1
````

We can now create the customGA class and test out the performance as follows:

````python
# define ost function
def costFunc(x):
    return np.square(x).sum()
 
x0 = np.array([-1,1])
lb = np.array([-2,-2])
ub = np.array([2,2])
popSize = 10
numElites = 1
k = 5
mutationP = 10
maxGen = 20
 
myGA = customGA(costFunc, x0,lb,ub,popSize,numElites,k,mutationP,maxGen)
myGA.run()
````

The solution goes to [0,0] which verifies that the algorithm indeed minimizes the function. The only missing functionality in my code is a convergence criteria, but that could be easily added. The parallelization can also be performed using the multiprocessing python library (https://docs.python.org/2/library/multiprocessing.html) when evaluating each population list for fitness. Just remember you must close the pool of threads or you might encounter a memory leak.

Let me know if you found this useful!