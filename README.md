**University of Pennsylvania, CIS 565: GPU Programming and Architecture,
Project 1 - Flocking**

* My name: Zibo Ye
  * [LinkedIn](https://www.linkedin.com/in/zibo-ye/)
  * [CMU ETC Intro Page](https://www.etc.cmu.edu/blog/author/ziboy/)
  * [Twitter](https://twitter.com/zibo_ye)
  * This is a guy trying to self-learn ways to optimize GPU codes.
* Tested on: Windows 11, AMD R7-5800H @ 3.2GHz (Up to 4.4Ghz) 32GB, RTX 3080 Mobile 16GB

## Introduction

In this project, I simulated the flocking behavior of scattered boids on GPU by using CUDA.
The Algorithm is based on Craig Reynold's artificial life program published on SIGGRAPH 1989.

the following three behaviors are implemented:

1. cohesion - boids move towards the perceived center of mass of their neighbors
2. separation - boids avoid getting to close to their neighbors
3. alignment - boids generally try to move with the same direction and speed as their neighbors

In the simulation results, the color of each particle is a representation of its velocity direction.

![Coherent Grid Flocking with 1M boids](results\coherent_grid_1M_short.gif)

_Coherent Grid Flocking with 1M boids_

## Implementation and Results
### Part 1. Naive Boids Simulation

The first simulation is a naive neighbor search, where each boid just iterates on the whole boids array to check whether they are within distance for cohesion, separation, or alignment. If a non-self boid is within any such distance, then its position and velocity will be taken into account for the respective rule. 

![Naive Boids Simulation](results\naive_5k.gif)

_Naive Grid Flocking with 5,000 boids_

### Part 2. Uniform Grid Boids

Naive boid simulation sucks, because each boid needs to check every other boid in the simulation space. This introduces $O(n^2)$ complexity, which is not scalable. To address this issue, I implemented a uniform grid boid simulation. In this simulation, a uniformed cube grid is built ahead of neighbor search. Each boid is assigned to a cell in the grid based on its position. Similar to the bounding box optimization in AABB traversal, with these cubes, Each boid only needs to check the cubes that overlap with its spherical neighborhood. Each boid calculates the extremities of its reach by using its own radius and position. With these extremities, I can calculate the maximum and minimum of my desired cells to scan. The time complexity of this algorithm is still $O(n^2)$, but in a much smaller scale that a modern GPU can easily handle.

![Uniform Boids Simulation](results\scattered_50k.gif)

_Uniform Grid Flocking with 50,000 boids_

### Part 3. Coherent Grid Boids

The third simulation builds on the second simulation. This time, we also rearrange the position and velocity information such that boids that are in a cell together are also contiguous in memory. This would help GPU to reach a higher cache hit rate, improving memory access efficiency.

![Coherent Grid Flocking with 1M boids](results\coherent_grid_1M_short.gif)

_Coherent Grid Flocking with 1M boids_

## Performance Analysis

**For each implementation, how does changing the number of boids affect performance? Why do you think this is?**

Across all three implementations, increasing the number of boids will increase compute time and therefore decrease FPS. For every kernel, the number of operations is proportional to the number of boids it need to process to decide whether the boid is its neighbor. For every method, increasing the number of boids will increase the number of operations, thus decreasing performance. However, if the grid is larger, the increasing boids might not increase density, and therefore the performance might not decrease as much. Noted that we are discussing this kernel wide. Globally speaking, increasing the number of boids will introduce more kernel calls, which will also decrease performance.

**For each implementation, how does changing the block count and block size affect performance? Why do you think this is?**

As seen from the graph, smaller block count usually results in poorer performance before a certain blockSize is hit. For example, there is a big difference between blockSize == 16 and blockSize == 4, however,above blockSize 32 is relatively similar in FPS performance. This behavior is not easily seen in simulations with fewer boids (N = 5,000). This is because NVIDIA GPUs are scheduling work in warps, each containing 32 threads. Any blockSize smaller than 32 will not be able to fill up a warp, therefore wasting performance.

**For the coherent uniform grid: did you experience any performance improvements with the more coherent uniform grid? Was this the outcome you expected? Why or why not?**

TODO: charts

Cache hit rate comparison

Calculation & Nsight

**Did changing cell width and checking 27 vs 8 neighboring cells affect performance? Why or why not? Be careful: it is insufficient (and possibly incorrect) to say that 27-cell is slower simply because there are more cells to check!**
