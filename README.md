Student Names: Anbang Hu, Yang Yu

## Proposal

### Advanced Solvers
We have implemented forward Euler and simplectic Euler in Assignment 4. We will explore another two stable integration methods - [backward Euler and improved Euler](https://math.la.asu.edu/~dajones/class/275/ch2.pdf).

#### Backward Euler
In order to take a backward Euler step, we need to solve a large linear system using linear algebra package [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). We will follow the section titled "Meshes and Matrices" in the [course notes](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf) to implement backward Euler.

1. Assign unique index to the vertices in the mesh;
2. Build the Laplace and mass matrice according to the indices of the vertices;
3. Solve the linear system that describes the backward Euler update rule.

Backward Euler update rule:

$y_{n+1} = y_n + \tau F(t_{n+1}, y_{n+1})$

__TODO: GIVE EQUATIONS OF BACKWARD EULER UPDATE RULE__

#### Improved Euler
We will modify the current forward Euler to an improved Euler by using the following update rule:

$y_{n+1} = y_n + \tau \frac{1}{2}(F(t_n, y_n) + F(t_{n+1}, y_n + \tau F(t_n, y_n)))$

This update rule is obtained by applying forward Euler to $y_{n+1}$ inside $F(t, y)$ in the following equation:

$\frac{y_{n+1} - y_n}{\tau} = \frac{1}{2}(F(t_n, y_n) + F(t_{n+1}, y_{n+1}))$

### Bubble Dynamics
We will following [this paper](http://www.cs.columbia.edu/cg/doublebubbles/doublebubbles.pdf) to simulate the way bubbles evolve over time.

#### One bubble with Dirichlet boundary
We will start with one bubble with Dirichlet boundary without any optimization.

__TODO: PDE WITH DIRICHLET BOUNDARY AND UPDATE RULE__

#### Multiple interacting bubbles and optimization
After we get one bubble dynamics to work, we will try implemeting multiple interacting bubbles and optimizing the program to make things faster. 

__TODO: NEED SOME MOTIVATING IMAGES__
