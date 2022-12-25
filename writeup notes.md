# Writeup notes

$$
\DeclareMathOperator{\Area}{Area}
\newcommand{\abs}[1]{\left\lvert #1 \right\lvert}
\newcommand{\brackets}[1]{\left[ #1 \right]}
\newcommand{\a}{\mathbf a}
\newcommand{\b}{\mathbf b}
\newcommand{\d}{\mathbf d}
\newcommand{\n}{\mathbf n}
\newcommand{\p}{\mathbf p}
\newcommand{\x}{\mathbf x}
\newcommand{\s}{\mathbf s}
\newcommand{\bv}{\mathbf v}
\newcommand{\F}{\mathbf F}
\newcommand{\BigO}{\mathcal O}
$$

## Abstract

## Intro

- "polygon" or "2d mesh" will refer to oriented boundary of 2d shape with holes
  - can provide diagram (or do that later)

## Related work

## Method

- flow (keep this brief since focus is on IPC)
  - how is grad SDF calculated
  - quadrature
    - need to do it to take edges into account, not just vertices
    - method
  - loop until fine mesh completely inside coarse mesh
    - all vertices inside
    - no intersections
- reinflation
  - reverse flow - for each flow step (in reverse), do gradient descent until fine mesh reaches target
  - for each descent step, do backtracking line search to choose best step to take based on energy
  - energy is displacement energy, cages energy, barrier potentials, and containment check
    - displacement energy guides fine mesh along reverse flow
    - cages energy maintains target goal of cage (e.g. minimal area)
    - barrier energy is from IPC, keeps coarse and fine meshes separated/prevents self-intersections
    - containment check tells us when we've crossed barrier
    - each energy has weight which may need to be changed based on mesh
      - area energy more prominent for meshes with greater area (larger, fewer/smaller holes)
  - cages energy is area implemented using green's theorem
    - integrate along oriented edges
    - formula in [implementation notes](implementation notes)
      - formula is simple for piecewise linear surface like polygon
    - other applications may use things like 3D volume, ARAP, symmetry, etc
  - barrier energy uses point-edge distances with asymptote near 0
    - high energy near 0 means they are always separated
    - calculated for point vs edge pairs of meshes
      - calculated for fine vs coarse and coarse vs fine to prevent collisions
      - calculated for coarse vs coarse to prevent self intersection (one of the guarantees of IPC)
      - not calculated for fine vs fine because that would interfere with flow reversal, which already makes fine mesh end up at intersection free state (since it goes back to original state)
    - calculate constraint set based on bounding volume hierarchy (!!), then calculate distances for pairs that constraint set returns
      - can create AABB trees in $\BigO(m + n)$ (?) time and query them in $\BigO(\log(m)) + \BigO(\log(n))$ (???) time vs checking all pairs takes $O(m \cdot n)$ time, so this is faster even though it needs to be recomputed at each step
    - differentiability
      - $-x^2 \log x$ if $x>0$ and $0$ if $x \leq 0$, so the asymptote function is twice differentiable everywhere
      - use autodiff to get gradient
  - containment energy
    - barrier energy does not take into account which side of edge we're on - prevents coincidence but not jumping
    - need to check if we've "clipped through" barrier
      - winding number doesn't cover all cases
      - if meshes intersect then vertex might be on opposide side of barrier (show diagram)
    - this is alternative to CCD, which limits step size based on barrier distance
      - cite Bijective Parameterization with Free Boundaries
  - weights - ? not sure if need to discuss this
    - disp energy vs cages energy weights are important since they are pretty much competing, but disp energy is extremely important
    - barrier energy weight not too important
      - affects "padding" between fine and coarse meshes
      - small weight is ok because barrier is asymptotic, goes to $\infty$ no matter what the coeffiecient is

## Results and applications

## Limitations and further work

- flow
  - no improvement on SDF flow which was main place where nested cages went wrong (still subject to impossible cases)
  - more sophisticated quadrature schemes may be faster
    - original paper used barycentre, which experimentally was inadequate
- need to compute global intersection tests
  - original IPC has options for primitive-primitive distances, developes edge-edge distance differentiability
  - edge-edge distances for barriers may be useful because if they are maintained as nonzero then we don't need to check intersections
- CCD may be faster
  - sets upper limit on line search, so need to compute fewer global intersection tests
  - still need to compute those intersection tests after we set upper bound though, since it relies on primitive-primitive distance
- cages energy relies on properly oriented mesh
