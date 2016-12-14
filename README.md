## Advanced Dynamic Simulations

Student Names: Anbang Hu, Yang Yu

_Note_: Proposal can be found [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Advanced%20Dynamic%20Simulations%20-%20Proposal.pdf). When writing the proposal, we did not present in great details what we intended to do, because we had few clue on what was actually coming in this project. As we proceed, the whole picture starts to become clear. Thus, we present detailed tasks in this writeup. 

### Overview
We have implemented forward Euler and simplectic Euler in [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28). In this project, we implement [improved Euler and backward Euler](https://math.la.asu.edu/~dajones/class/275/ch2.pdf) within the framework of Scotty3D.

### Enhanced Viewer
In order to visualize the effects from advanced solvers, we enhance the viewer from [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28) in the following way:
- We add ```Action::Heat``` to the viewer to allow users  to simulate heat diffusion on objects.
- We also modify the viewer to allow users to use 
    1. improved Euler for wave equation
    2. backward Euler for wave equation
    3. backward Euler for heat equation

The modification is done in ```application.h```, ```application.cpp```.

### Wave Equation
#### Forward Euler
We saw from [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28) that forward Euler is not stable. A slight initial offset at the left buttock of the bunny
![Initial offset](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/bunny_with_initial_offset.png?raw=true)

blows up the mesh:
![Foward Euler (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/forward_euler_wave_bunny.png?raw=true)

#### Improved Euler
Fortunately, we can improve forward Euler by taking into account of both derivatives at current timestep and next timestep. The formulation for improved Euler for wave equation can be found [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Improved_Euler.pdf).

We code up improved Euler for wave equation in ```Mesh::improved_euler_wave()``` in ```mesh.cpp```:

1. calculate new velocity based on current offset and velocity for each vertex and store them in temporary variables,
2. calculate new offset based on current offset, current velocity and new velocity for each vertex and store them in temporary variables,
3. update velocity and offset for each vertex.

Although running for a sufficiently long time still blows up the mesh, running for the same duration as in section Forward Euler gives better result:
![Improved Euler (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/improved_euler_wave_bunny.png?raw=true)

#### Backward Euler
Improved Euler does not seem to always give stable results. In this section, we look at a more powerful update rule called backward Euler, which is unconditionally stable.

Backward Euler update rule is
```
u(k+1) = u(k) + tau * u'(k+1)
```
Since ```u(k+1)``` appears on both side of the equation, we need to solve an algebraic equation for ```u(k+1)```.

There are numerous ways to solve backward Euler. We experiment on two of them:
- [fixed point iteration](https://en.wikipedia.org/wiki/Fixed-point_iteration)
- [Newton–Raphson method](https://en.wikipedia.org/wiki/Newton%27s_method)

#### Fixed Point Iteration
We describe fixed point iteration for wave equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Backward_Euler__Fixed_Point_Iteration_.pdf) and implement it in ```Mesh::backward_euler_fixedpoint_wave()``` in ```mesh.cpp```:

1. create temporary variables to store current velocity and offset for each vertex,
2. for vertex ```v```, calculate new velocity based on current velocity, ```v->offset``` and ```v->velocity``` and store it in ```v->velocity```,
3. for vertex ```v```, calculate new offset based on current offset, ```v->velocity```,
4. run step 2 and step 3 for several times.

Even with large initial offset, the result from fixed point iteration is rather smooth:
![Backward Euler Fixed Point (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_fixedpoint_bunny.png?raw=true)

#### Newton–Raphson method
We descibe Newton-Raphson method for wave equation here and implement it in ```Mesh::backward_euler_newton_wave()``` in ```mesh.cpp```:

__TODO__ implementation details and result

_Note_: Though stable, both fixed point iteration and Newton-Raphson are iterative methods. They need to run several iteration for each vertex update, which slows down the computation, which in turn slows down the video playback.

### Heat Equation
Results from wave equation suggests that backward Euler works better than other update rules. We focus on ONLY __backward Euler__ for heat equation, and explore both iterative methods and matrix method:
- [fixed point iteration (iterative)](https://en.wikipedia.org/wiki/Fixed-point_iteration)
- [Newton–Raphson method (iterative)](https://en.wikipedia.org/wiki/Newton%27s_method)
- [matrix method](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf)

To support heat equation, we 

1. add a field ```temperature``` at each vertex;
2. initialize ```temperature``` to zero at vertex creation;
3. modify ```Vertex::laplacian``` to allow computing discrete laplacian for heat equation,
4. __TODO__ add ```draw_color``` to color the mesh;
5. __TODO__ add functionality to allow users to select a vertex and adjust its temperature.

#### Fixed Point Iteration
We describe fixed point iteration for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler__Fixed_Point_Iteration_.pdf). We implement it in ```Mesh::backward_euler_fixedpoint_heat()``` in ```mesh.cpp```:

1. create temporary variables to store current temperature for each vertex,
2. calculate new temperature for each vertex ```v``` based on current temperature and ```v->temperature```, and store it in ```v->temperature```,
3. run step 2 for several times.

__TODO__ present result

#### Newton-Raphson Method
We describe Newton-Raphson for heat equation here. We implement it in ```Mesh::backward_euler_newton_heat()``` in ```mesh.cpp```:

__TODO__ implementation details and result

#### Power of Matrix
As mentioned above, backward Euler implemented using an iterative method tends to be fairly slow. One way to speed it up is to assemble everything into a matrix and solve a big linear system.

We describe backward Euler with matrix method for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler__Solving_Linear_Systems_.pdf). We follow the idea in the section titled "Meshes and Matrices" in these [course notes](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf) as well as linear algebra package [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) to implement it in ```Mesh::backward_euler_matrix_heat()``` in ```mesh.cpp```:

1. Assign unique index to the vertices in the mesh;
2. Build the Laplace and mass matrice (sparse matrix) according to the indices of the vertices;
3. Solve the linear system that describes the backward Euler update rule.

__TODO__ present result

### Conclusion
In this project, we implement advanced solvers for both wave equation and heat equation. We see that improved Euler gives a slightly more stable result than forward Euler, whereas backward Euler produces the most smooth result. However, iterative approximation for backward Euler is slow. Whenever possible, it is faster to implement backward Euler by first converting the problem into a linear system represented as sparse matrix and then solve it.

### References
We list all the references we have used in the following:
- [Computer Graphics Assignment 4](http://15462.courses.cs.cmu.edu/fall2016/article/28)
- [Numerical Approximations](https://math.la.asu.edu/~dajones/class/275/ch2.pdf)
- [Wiki: Fixed Point Iteration](https://en.wikipedia.org/wiki/Fixed-point_iteration)
- [Wiki: Newton's Method](https://en.wikipedia.org/wiki/Newton%27s_method)
- [Discrete Differential Geometry: An Applied Introduction](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf)
- [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
