## Advanced Dynamic Simulations

Student Names: Anbang Hu, Yang Yu

_Note_: Proposal can be found [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Advanced%20Dynamic%20Simulations%20-%20Proposal.pdf). When writing the proposal, we did not present in great details what we intended to do, because we had few clue on what was actually coming in this project. As we proceed, the whole picture starts to become clear. Thus, we present detailed tasks in this writeup. 

### Overview
We have implemented forward Euler and simplectic Euler in [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28). In this project, we implement [improved Euler and backward Euler](https://math.la.asu.edu/~dajones/class/275/ch2.pdf) for both wave and heat equation within the framework of Scotty3D. When implementing the backward Euler, we try different  methods to solve the equation, e.g, Fixed-point iteration, Newton's Method, Linear System.Also,we support solving the heat equation in forward Euler.(Implementation is in ```mesh.h```, ```mesh.cpp```.)

### Enhanced Viewer
In order to visualize the effects from advanced solvers, we enhance the viewer from [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28) in the following way:
- We add visualization for temperature to the viewer to allow users  to simulate heat diffusion on objects.
- We also modify the viewer to allow users to use 
    1. forward Euler for heat equation
    2. improved Euler for wave equation
    2. improved Euler for heat equation
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

#### Newton's method
We descibe Newton's method for wave equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Backward_Euler_Newton_Method_.pdf) and implement it in ```Mesh::backward_euler_newton_wave()``` in ```mesh.cpp```:
1. create temporary variables to store velocity  for each vertex in each iteration,
2. for vertex ```v```, calculate new velocity and new offset using Newton's Method and store it in ```v->velocity``` and ```v->offset```,(one iteration of Newton Method)
3. run step 2 for several times to get the final result for ```v->velocity```
4. for vertex ```v```, calculate new offset based on current offset, and add ```v->velocity * timestep```,

Newton's method can produces smoother result:
![Backward Euler Newton (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_newton_wave_bunny.png?raw=true)

_Note_: Though stable, both fixed point iteration and Newton's are iterative methods. They need to run several iteration for each vertex update, which slows down the computation, which in turn slows down the video playback.

### Heat Equation
#### Forward Euler
First we use forward Euler to solve the heat equation. We described our algorithm [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Forward_Euler_.pdf).And the result is shown below.
![Forward Euler (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat0.png?raw=true)
![Forward Euler (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat1.png?raw=true)
![Forward Euler (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat2.png?raw=true)
![Forward Euler (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat3.png?raw=true)
![Forward Euler (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat4.png?raw=true)


#### Improved Euler
We also use improved Euler to solve the heat equation. We described our algorithm [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Improved_Euler_.pdf).And the result is shown below
![Improved Euler (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat0.png?raw=true)
![Improved Euler (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat1.png?raw=true)
![Improved Euler (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat2.png?raw=true)
![Improved Euler (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat3.png?raw=true)

![Improved Euler (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat4.png?raw=true)


#### Backward Euler
Results from wave equation suggests that backward Euler works better than other update rules. We focus on ONLY __backward Euler__ for heat equation, and explore both iterative methods and matrix method:
- [fixed point iteration (iterative)](https://en.wikipedia.org/wiki/Fixed-point_iteration)
- [Newton–Raphson method (iterative)](https://en.wikipedia.org/wiki/Newton%27s_method)
- [matrix method](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf)

To support heat equation, we 
1. add a field ```temperature``` at each vertex;
2. initialize ```temperature``` to 1200 Kelvin at vertex creation (we follow the physics of [blackbody radiation](https://docs.kde.org/trunk5/en/kdeedu/kstars/ai-blackbody.html));
3. modify ```Vertex::laplacian``` to allow computing discrete laplacian for heat equation,
4. modify ```Mesh::draw_faces``` to allow mesh coloring using ```temperature_to_rgb``` based on temperature at each vertex;
5. modify ```Application::keyboard_event()``` to allow users to adjust the temperature of selected vertex;
6. follow [this post](http://www.tannerhelland.com/4435/convert-temperature-rgb-algorithm-code/) to add ```temperature_to_rgb()``` in ```mesh.cpp``` to convert temperature into rgb color;
7. modify ```Application::draw_action()``` in ```application.cpp``` to display temperature of selected vertex.

##### Fixed Point Iteration
We describe fixed point iteration for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler__Fixed_Point_Iteration_.pdf). We implement it in ```Mesh::backward_euler_fixedpoint_heat()``` in ```mesh.cpp```:

1. create temporary variables to store current temperature for each vertex,
2. calculate new temperature for each vertex ```v``` based on current temperature and ```v->temperature```, and store it in ```v->temperature```,
3. run step 2 for several times.
The result is shown below
![Backward Euler Fixed Point (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat0.png?raw=true)
![Backward Euler Fixed Point (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat1.png?raw=true)
![Backward Euler Fixed Point (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat2.png?raw=true)
![Backward Euler Fixed Point (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat3.png?raw=true)
![Backward Euler Fixed Point (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat4.png?raw=true)


##### Newton's Method
We describe Newton's for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler_Newton_Method_.pdf). We implement it in ```Mesh::backward_euler_newton_heat()``` in ```mesh.cpp```:
1. create temporary variables to store current temperature for each vertex,
2. calculate new temperature using Newton Method  for each vertex ```v``` based on current temperature and ```v->temperature```, and store it in ```v->temperature```,
3. run step 2 for several times to get the final temperature for each vertex.

The result is shown below
![Backward Euler Newton (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat0.png?raw=true)
![Backward Euler Newton (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat1.png?raw=true)
![Backward Euler Newton (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat2.png?raw=true)
![Backward Euler Newton (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat3.png?raw=true)
![Backward Euler Newton (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat4.png?raw=true)

##### Power of Matrix
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
- [How to Convert Temperature (K) to RGB: Algorithm and Sample Code](http://www.tannerhelland.com/4435/convert-temperature-rgb-algorithm-code/)
- [Blackbody Radiation](https://docs.kde.org/trunk5/en/kdeedu/kstars/ai-blackbody.html)
