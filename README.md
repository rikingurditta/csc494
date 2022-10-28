# csc494

This is my repo for notes I've taken for my CSC494 project on augmenting Nested Cages with IPC.

## Notes on relevant papers

- [Nested Cages](nested cages)
- [Incremental Potential Contact](incremental potential contact)

## Other notes

- [Implementation notes](implementation notes)
- [Questions for Alec](questions for alec)

## (Proposed) Timeline

| Date   | Code/Implementation goal                                     | Notes/Understanding goal                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Sep 27 | functions decided and implementation planned for nested cages (headers created) | notes on IPC contact model, contact mechanics, friction forces |
| Oct 4  | implementation for nested cages flow done (more or less)     | notes on IPC distance computation                            |
|        |                                                              |                                                              |
| Oct 18 | start planning IPC headers/structure                         |                                                              |
| Oct 24 | finish planning headers for IPC, implement energies (do barrier energy without constraint set) | partial notes on inversion-aware line search filter [Smith and Schaefer 2015] |
| Nov 1  | implement constraint set (spatial hash + bvh) and update barrier energy, incorporate TinyAD to get gradients and hessians for free | finish notes on line search filter                           |
| Nov 8  | implement time step with gradient descent, test/investigate  |                                                              |
| Nov 15 | test more, start implementing time step with line search filter |                                                              |
| Nov 29 | finish implementing time step with line search filter, more testing, start writeup |                                                              |
|        |                                                              |                                                              |
| Dec 13 | finish writeup                                               |                                                              |

- tinyad - autodiff library
- start w gradient descent instead of newton's method
