# Incremental Potential Contact: Intersection- and Inversion-free Large-Deformation Dynamics

$$
\newcommand{\A}{\mathcal A}
\newcommand{\C}{\mathcal C}
\DeclareMathOperator{\argmin}{argmin}
$$

- efficiently time-stepping accurate and consistent simulations of real-world contacting elastic objects remains difficult
- IPC is new model and algorithm for variationally solving implicitly time-stepped non-linear elastodynamics
- solves contact problems that are:
  - intersection-free
  - inversion-free
  - efficient
    - comparable to/faster than available methods (which lack both convergence and feasibility)
  - accurate

## Introduction

- contact is difficult due to intertwined physical and geometric factors, especially in the presence of friction and nonlinear elasticity
  - real-world contact and friction forces are discontinuous - makes for stiff time-stepping problems
  - small violations of exact contact constraints (which are nonconvex) can lead to very bad geometric configurations (impossible to untangle)
    - also often lead to extreme deformations resulting in element inversions
  - friction introduces challenges with asymmetric force coupling and rapid switching between sliding and sticking

Goals:

- achieve very high robustness
  - i.e. no catastrophic failures or stagnation
  - even for challenging elastodynamic contact problems with friction
  - should be independent of user-controllable accuracy in time-stepping, spacial discretization, contact resolution
  - still maintain efficiency
- ensure that all accuracies are efficiently attainable when required

Specifications:

- solver is constructed for mesh-based discretizations of nonlinear volumetric elastodynamic problems
- solver supports
  - large nonlinear deformations
  - implicit time-stepping with contact
  - friction
  - boundaries of arbitrary codimension (points, curves, surfaces, volumes)
- physics can be approximated arbitrarily coarsely but geometric constraints (no intersections, no inversions) are maintained exactly

Approach:

- key element is formulation of contact problem and customized numerical method to solve
- exact constraint formulation in terms of unsigned distance function, and rate-based maximal dissipation for friction
- for each timestep, solve discreete nonlinear contact problem with given tolerance
  - use smoothed barrier method
    - almost everywehre $C^2$ function of unsigned distances
      - $C^1$ for measure-zero set of configurations
      - makes it possible to use newton-type optimizatio methods
    - support is restricted to small part of configuration space close to configuration with contact
      - additional contact forces are applied highly locally
      - only a small set of terms of barrier function need to be computed explicitly during optimization
  - ensure that solution is intersection-free at all intermediate steps
    - line search has filtered continuous collision detection
  - use comparably smoothed arbitrarily close approximation to static friction (so no explicit Coulomb constant constraint needed)
    - cast friction forces at every timestep in a dissipative potential form
    - friction is resolved in same solver

## Method

### Contact model

- $x$ is the positions of all $n$ nodes (so $\dot x$ is their velocities)
- $M$ is the FEM mass matrix for the nodes
- $\Psi(x)$ is the deformation energy of the material
- $f_e$ is the external forces on the nodes
- $f_d$ is the dissipative forces, i.e. frictional forces
- $T$ is the total time
- $\A$ is the set of admissible trajectories

We consider the energy

$$
S(x) = \int_0^T \left( \frac{1}{2} \dot x^T M \dot x + \Psi(x) + x^T \left( f_e + f_d \right) \right) dt
$$

on $\A$ the set of admissible trajectories

#### Admissible trajectories

Let $\A_I$ be the set of trajectories that are intersection free, i.e. distances between objects are always positive ($d(p, q) > 0$). This set is defined by a strict inequality, so the $\A_I$ is an open set. We take $\mathcal A$ to be the closure of $\A_I$, i.e. it is $\A_I$ union its boundary.

Note that this isn't the set of trajectories where $d(p, q) \geq 0$, as that would be every trajectory since unsigned distance is nonnegative.

If $x(t)$ is a trajectory with $x(0) = x_0$ being intersection free, and we enforce $d_{i, j}(x(t)) > 0$ for all $t$ (i.e. no pair of mesh primitives $i, j$ intersects at any time $t$), then we know that $x(t)$ is intersection free.

#### Time discretization

Can construct energy so that we can solve time step by minimizing over $x$, given current $x^t$ and $v^t$ as well as time step size $h$

Can use Incremental Potential, e.g. for implicit Euler we have

$$
E(x, x^t, v^t) = \frac{1}{2} (x - \hat x)^T M (x - \hat x) - h^2 x^T f_d + h^2 \Psi(x) \\
(\text{where } \hat x = x^t + hv^t + h^2 M^{-1} f_e)
$$

We can make our time step update subject to our contact constraint by solving

$$
x^{t+1} = \underset{x}{\argmin} E(x, x^t, v^t),\ x^\tau \in \A \\
(\text{where $x^\tau$ is the linear path between $x^t$ and $x^{t+1}$})
$$

#### $\epsilon$-separated admissible trajectories

Two piecewise linear surfaces are $\epsilon$-separated if the distance between their boundary points is at least $\epsilon$ (unless they are on the same element of the boundary)

A trajectory $x$ is in $\A_\epsilon$ if the surfaces stay $\epsilon$-separated for the whole trajectory. So $\A_\epsilon \subset \A_I$

To implement a constraint so that we can solve for trajectories in $\A_\epsilon$, we use a barrier term, i.e. a term in the optimization energy that vanishes when the surfaces are $\epsilon$-separated and blows up as they get closer together. We also need to use continuous collision detection within minimization steps (this constraint alone doesn't ensure that there are no collisions between $x^t$ and $x^{t+1}$)

#### Trajectory accuracies

[TODO]

## Primal barrier contact mechanics

Will discuss friction later

#### Barrier-aware projected newton

```python
def BarrierAwareProjectedNewton(x_t, e):
    x = x_t
    Chat = ComputeConstraintSet(x, dhat)
    E_prev = Bt(x, dhat, Chat)
    x_prev = x
    do:
        H = SPDProject(Laplacian(B_t(x, dhat, Chat)))
        p = - inv(H) * grad(B_t(x, dhat, Chat))
        # CCD line search
        a = min(1, StepSizeUpperBound(x, p, Chat))
        do:
            x = x_prev + a * p
            Chat = ComputeConstraintSet(x, dhat)
            a = a / 2
        while B_t(x, dhat, Chat) > E_prev
        E_prev = B_t(x, dhat, Chat)
        x_prev = x
        Update K, BCs, and equality constraints -> supplemental
    while 1/h * infnorm(p) > e_d
    return x
```

### Barrier-augmented incremental potential

to enforce distance constraints $d_k(x) > 0$ for all $k \in \C$, we construct continuous barrier energy $b$

- only affects motion when primitives are close to collision
- $k$ are elements of $\C$
- global barrier energy is then $\displaystyle \kappa \sum_{k \in \C} b(d_k(x))$

can augment time step potential energy with global barrier energy:
$$
B_t(x) = E(x, x^t, v^t) + \kappa \sum_{k \in \C} b(d_k(x))
$$

- $\kappa > 0$ is adaptive conditioning pattern
  - controls barrier stiffness
- naively dealing with this would require evaluating barrier energy for every pair of elements aka $\mathcal O(|\mathcal T|^2)$ pairs
- speed it up by designing smooth barrier functions
  - allow for exact and efficient barrier energy computations
  - only need to evaluate distance for a small subset of pairs $k, \ell \in \C$
