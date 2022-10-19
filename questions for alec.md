# Questions for Alec

## Gradient of SDF :white_check_mark:

### Quadrature

- need quadrature to integrate SDF along edges so that we move vertices in a way that is good for edged around vertex rather than just vertex
  - helps us move the entire mesh the way we want rather than only paying attention to vertices
- edge midpoints is a good quadrature strategy to start
- can also look at higher order quadrature
  - where?

### Gradient when vertices of $F$ intersect $C$

- if $v \in V_F$ is on an edge of $C$, can use unit edge normal (pointing outward) as $\nabla SDF$ (since it tells us exactly which way edge is pointing)
  - need to figure out normal orientation for this :bangbang:
  - \^have not done that yet
- if $v$ is also a vertex of $C$, then can average normals of two adjacent edges of $C$

## Test meshes :white_check_mark:

- currently creating from scratch
- can use Matlab gptoolbox `bwmesh` to turn alpha channel of img into mesh
  - https://github.com/alecjacobson/gptoolbox/blob/master/mesh/bwmesh.m

## How do we construct the constraint set?

- I'm thinking bounding volume hierarchy
- paper also mentions spatial hash
  - is there a spatial hash in libigl? i did not see it when looking through ``.h` files or tutorial

## Distance smoothing

- what is happening here? i do not see the intuition

![IPC_distance_smoothing_screenshot](IPC_distance_smoothing_screenshot.png)

## Friction

- just want to verify - is it alright to skip friction for nested cages? does not seem necessary here

## TinyAD

- how do i set up the cmake properly? do you have an example project?

## Gradient descent

- you suggested using gradient descent instead of Newton's method
- paper suggests using "barrier-aware projected Newton", are you saying replace this with gradient descent?
- is the idea to do gradient descent but using the filters for intersections and inversions that they suggest to use wihin the line search? (4.4)
