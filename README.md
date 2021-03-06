## Notice:

I'm seen a few people forking this. Please keep in mind all code here is licensed under the AGPLV3. I did this because I have absolutely no clue how this code is useful beyond making pretty graphs. If you contact me and tell me how you are using this code I'll be happy to give you a less strictly licensed version.

# columbus: Software for Computing Approximate Solutions to the Traveling Salesman's Problem

This repository contains cuda code for solving the traveling salesman problem using a GPU. See the paper in the docs folder for a detailed explanation.


The purpose of this project is to use clever computational techniques to implement Metropolis Hastings algorithms on a GPU. To example the problem, we solve the traveling salesman problem using simulated annealing. The heart of implementing Metropolis Hastings on a GPU is to use the GPU to make hundreds of thousands of samples at each step, while checking and updating the objective and loss function in a purely parallel manner. Instead of basing updates on the best sample of a particular set of draws we simply take the last sample that was a success. This allows the update step to be made purely in parallel, with the only wait time being the write of the first success in a block.

The most useful part of a GPU implementation of Metropolis Hastings is the ability to dramatically increase the number of proposal steps made during each iteration. On a GTX 780 GPU there are 8 SM's that can handle 16 blocks of size 1024 at a time. This means we can draw and evaluate 131,072 proposal steps at each initialization of the kernel. Leveraging high throughput allows the algorithm to explore a much larger space than it would be able to search on a serial CPU. 

The code has so far only been tested on trip sizes no larger than 744,710 cities. But because of an efficient schema to calculate the distances necessary in the proposal step the algorithm should be capable of handle trip sizes in the millions.

Cloning this repo and typing `make` will create the tsp_cuda program which can be accessed through something like

```
./columbus data/earring200K993 -maxiter= 100 -global_search= .01 -local_search= .1
```

The following are the flags and inputs you can use to run the program

```
Inputs: 
(Required) input_file.tsp: [char()] 
 - The name of the tsp file, excluding .tsp at the end, containing the cities to travel over. 
(Optional) -trip: [char()] 
 - The name of the csv file, excluding .csv, containing a previously found trip. If missing, a linear route is generated as the starting trip. 
(Optional) -temp: [float(1)] 
 - The initial starting temperature. Default is 1000 
(Optional) -decay: [float(1)]  
 - The decay rate for the annealing schedule. Default is .99 
(Optional) -maxiter: [integer(1)]  
 - The maximum number of iterations until failure. 
  Default is -1, which runs until temperature goes to the minimum.
(Optional) -global_search: [float(1)]  
 - A parameter that controls the variance of the second city search space,
   such that the variance is [30 + exp(global_search/Temp) * N]. default is .01.
  See An example of what this controls here:
(Optional) -local_search: [float(1)]  
 - A parameter that controls the variance of the second city search space,
   such that the variance is [30 + exp(local_search/Temp) * N]. default is 1.
```

The program will output a csv of the best trip found throughout the simulated annealing process.

Several tsp datasets are given, but you can download more of them at the [TSPLIB](http://comopt.ifi.uni-heidelberg.de/software/TSPLIB95/) library and the University of Waterloo's [TSP Data](http://www.math.uwaterloo.ca/tsp/data/) webpage.

This project is still very much in beta, but running the algorithm on the classic [mona lisa](http://www.math.uwaterloo.ca/tsp/data/ml/monalisa.html) problem for 1.7 hours yielded a trip length of 5,971,924 and made a very nice picture which you can also generate with the given jupyter notebook.

Below is an example of Venus, with a total of 140K cities we achieve a result of 6,986,033 which is only 3% worse than the best found score of 6,810,665. While normal solvers may have taken a month to find this answer our algorithm only took 20 hours.


![](https://i.imgur.com/GjaKFAJ.jpg)
