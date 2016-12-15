## Advanced Dynamic Simulations

Student Names: Anbang Hu, Yang Yu

_Note_: Proposal can be found [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Advanced%20Dynamic%20Simulations%20-%20Proposal.pdf). When writing the proposal, we did not present in great details what we intended to do, because we had few clue on what was actually coming in this project. As we proceed, the whole picture starts to become clear. Thus, we present detailed tasks in this writeup. 

### Overview
We have implemented forward Euler and simplectic Euler in [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28). In this project, we implement [improved Euler and backward Euler](https://math.la.asu.edu/~dajones/class/275/ch2.pdf) for both wave and heat equation within the framework of Scotty3D. When implementing the backward Euler, we try different  methods to solve the equation, e.g, Fixed-point iteration, Newton's Method, Solving Linear System. In addition, we support solving the heat equation with forward Euler. (Implementation can be found in ```mesh.h```, ```mesh.cpp```.)

### Enhanced Viewer
In order to visualize the effects from advanced solvers, we enhance the viewer from [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28) in the following way:
- We add visualization for temperature to the viewer to allow users  to simulate heat diffusion on objects.
- We also modify the viewer to allow users to use 
    1. forward Euler for heat equation
    2. improved Euler for wave equation
    2. improved Euler for heat equation
    2. backward Euler for wave equation
    3. backward Euler for heat equation

The modification is mostly done in ```application.h```, ```application.cpp```. The detail tasks on temperature visualization will be discribed at the beginning of __Heat Equation__ section.

### Wave Equation
#### Forward Euler
We saw from [A4](http://15462.courses.cs.cmu.edu/fall2016/article/28) that forward Euler is not stable. A slight initial offset at the left buttock of the bunny
![Initial offset](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/bunny_with_initial_offset.png?raw=true)

quickly blows up the mesh:
![Foward Euler (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/forward_euler_wave_bunny.png?raw=true)

#### Improved Euler
Fortunately, we can improve forward Euler by taking into account of both derivatives at current timestep and next timestep. The formulation for improved Euler for wave equation can be found [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Improved_Euler.pdf).

We code up improved Euler for wave equation in ```Mesh::improved_euler_wave()``` in ```mesh.cpp```:

1. calculate new velocity based on current offset and velocity for each vertex and store them in temporary variables,
2. calculate new offset based on current offset, current velocity and new velocity for each vertex and store them in temporary variables,
3. update velocity and offset for each vertex.

Although the mesh still blows up when running for a sufficiently long time, it has better result when running for the same duration as in __Forward Euler__ section:
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
- [Newton's method](https://en.wikipedia.org/wiki/Newton%27s_method)

##### Fixed Point Iteration
We describe fixed point iteration for wave equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Backward_Euler__Fixed_Point_Iteration_.pdf) and implement it in ```Mesh::backward_euler_fixedpoint_wave()``` in ```mesh.cpp```:

1. create temporary variables to store current velocity and offset for each vertex,
2. for vertex ```v```, calculate new velocity based on current velocity, ```v->offset``` and ```v->velocity``` and store it in ```v->velocity```,
3. for vertex ```v```, calculate new offset based on current offset, ```v->velocity```,
4. run step 2 and step 3 for several times (in our case, we choose to run 10 times).

Even with large initial offset, the result from fixed point iteration is rather smooth:
![Backward Euler Fixed Point (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_fixedpoint_bunny.png?raw=true)

##### Newton's method
We descibe Newton's method for wave equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Wave_Equation_With_Backward_Euler_Newton_Method_.pdf) and implement it in ```Mesh::backward_euler_newton_wave()``` in ```mesh.cpp```:
1. create temporary variables to store velocity  for each vertex in each iteration,
2. for vertex ```v```, calculate new velocity and new offset using Newton's Method and store it in ```v->velocity``` and ```v->offset``` (one iteration of Newton Method),
3. run step 2 for several times to get the final result for ```v->velocity``` (in our case, we choose to run 10 times),
4. for vertex ```v```, calculate new offset based on current offset, and add ```v->velocity * timestep```.

Newton's method can produce even smoother result:
![Backward Euler Newton (Wave)](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_newton_wave_bunny.png?raw=true)

_Note_: Though stable, both fixed point iteration and Newton's are iterative methods. They need to run several iteration for each vertex update, which slows down the computation, which in turn slows down the video playback.

### Heat Equation
To support heat equation, we 
1. add a field ```temperature``` at each vertex;
2. initialize ```temperature``` to 4200 Kelvin at each vertex creation (we follow the physics of [blackbody radiation](https://docs.kde.org/trunk5/en/kdeedu/kstars/ai-blackbody.html));
3. modify ```Vertex::laplacian()``` to allow computation of discrete laplacian for heat equation,
4. follow [this post](http://www.tannerhelland.com/4435/convert-temperature-rgb-algorithm-code/) to add ```temperature_to_rgb()``` in ```mesh.cpp``` to convert temperature into rgb color;
5. modify ```Mesh::draw_faces()``` to allow mesh coloring using ```temperature_to_rgb()``` based on temperature at each vertex;
6. modify ```Application::keyboard_event()``` in ```application.cpp``` to allow users to adjust the temperature of selected vertex;
7. modify ```Application::draw_action()``` in ```application.cpp``` to display temperature of selected vertex.

We use different method to implement the heat equation (forward Euler, improved Euler and backward Euler). After implementing these algorithms, we test them on ```dae/sky/bunny.dae```. At the beginning, all vertices have the same temperature (4200 Kelvin), which will show light red. First we select and increase the temperature at some vertices (to over 30000 Kelvin). The neighborhood of these vertices will become light blue or blue when the temperature keeps increasing. Then we use video playback to observe heat diffusion in the bunny mesh.

#### Forward Euler
First we use forward Euler to solve the heat equation. We described our algorithm [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Forward_Euler_.pdf).
We implement it in ```Mesh::forward_euler_heat()``` in ```mesh.cpp```:
1. for vertex ```v```, calculate the laplacian and store it in temporary variables,
2. use the formula in the algorihtm to compute the new temperature for each vertex, 
3. for vertex ```v```, update the temperature.

The result is shown below.
Initially, we select some vertices and increase their temperature (blue regions):
![Forward Euler (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat0.png?raw=true)
Let the heat diffusion process begin:
![Forward Euler (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat1.png?raw=true)
![Forward Euler (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat2.png?raw=true)
![Forward Euler (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat3.png?raw=true)
![Forward Euler (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/forward_euler_heat4.png?raw=true)

#### Improved Euler
We also use improved Euler to solve the heat equation. We described our algorithm [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Improved_Euler_.pdf) and  code up improved Euler for heat equation in ```Mesh::improved_euler_heat()``` in ```mesh.cpp```:
1. calculate new temperature based on current temperature and laplacian for each vertex and store them in temporary variables,
2. calculate new temperature based on current temperature and results in step 1 and store them in temporary variables,
3. update temperature for each vertex.

And the result is shown below.
![Improved Euler (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat0.png?raw=true)
![Improved Euler (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat1.png?raw=true)
![Improved Euler (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat2.png?raw=true)
![Improved Euler (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat3.png?raw=true)

![Improved Euler (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/improved_euler_heat4.png?raw=true)


#### Backward Euler
Results from wave equation suggests that backward Euler works better than other update rules. In this section we explore both iterative methods and matrix method:
- [fixed point iteration (iterative)](https://en.wikipedia.org/wiki/Fixed-point_iteration)
- [Newton's method (iterative)](https://en.wikipedia.org/wiki/Newton%27s_method)
- [matrix method](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf)


##### Fixed Point Iteration
We describe fixed point iteration for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler__Fixed_Point_Iteration_.pdf). We implement it in ```Mesh::backward_euler_fixedpoint_heat()``` in ```mesh.cpp```:

1. create temporary variables to store current temperature for each vertex,
2. calculate new temperature for each vertex ```v``` based on current temperature and ```v->temperature```, and store it in ```v->temperature```,
3. run step 2 for several times (10 times in our case).

The result is shown below.
![Backward Euler Fixed Point (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat0.png?raw=true)
![Backward Euler Fixed Point (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat1.png?raw=true)
![Backward Euler Fixed Point (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat2.png?raw=true)
![Backward Euler Fixed Point (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat3.png?raw=true)
![Backward Euler Fixed Point (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/cf3c043f729a600385f9750eeaaf23dc52202876/figures/backward_euler_fixedpoint_heat4.png?raw=true)


##### Newton's Method
We describe Newton's method for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler_Newton_Method_.pdf). We implement it in ```Mesh::backward_euler_newton_heat()``` in ```mesh.cpp```:
1. create temporary variables to store current temperature for each vertex,
2. calculate new temperature using Newton Method  for each vertex ```v``` based on current temperature and ```v->temperature```, and store it in ```v->temperature```,
3. run step 2 for several times to get the final temperature for each vertex (10 times in our case).

The result is shown below.
![Backward Euler Newton (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat0.png?raw=true)
![Backward Euler Newton (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat1.png?raw=true)
![Backward Euler Newton (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat2.png?raw=true)
![Backward Euler Newton (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat3.png?raw=true)
![Backward Euler Newton (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/1181b5f5a3b114c8bc7cb60fab30bc1eaf21fef6/figures/backward_euler_newton_heat4.png?raw=true)

##### Power of Matrix
As mentioned in __Wave Equation__ section, backward Euler implemented using an iterative method tends to be fairly slow. One way to speed it up is to assemble everything into a matrix and solve a big linear system.

We describe backward Euler with matrix method for heat equation [here](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/formula/Heat_Equation_With_Backward_Euler__Solving_Linear_Systems_.pdf). We follow the idea in the section titled "Meshes and Matrices" in these [course notes](http://www.cs.cmu.edu/~kmcrane/Projects/DGPDEC/paper.pdf) as well as linear algebra package [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) to implement it in ```Mesh::backward_euler_matrix_heat()``` in ```mesh.cpp```:

1. Assign unique index to the vertices in the mesh;
2. Build the Laplace and mass matrice (sparse matrix) according to the indices of the vertices;
3. Solve the linear system that describes the backward Euler update rule.

The result is shown below.
![Backward Euler Matrix (Heat)0](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_matrix_heat0.png?raw=true)
![Backward Euler Matrix (Heat)1](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_matrix_heat1.png?raw=true)
![Backward Euler Matrix (Heat)2](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_matrix_heat2.png?raw=true)
![Backward Euler Matrix (Heat)3](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_matrix_heat3.png?raw=true)
![Backward Euler Matrix (Heat)4](https://github.com/Haboric-Hu/advanced-dynamic-simulations/blob/master/figures/backward_euler_matrix_heat4.png?raw=true)

We expected that solving linear systems would be faster than iterative methods. However, it turns out that the opposite is true. A reason may be that iterative methods run for too few iterations (only 10 times). Another reason is that optimization in solving linear systems can be done to accelerate the computation.

### Conclusion
In this project, we implement advanced solvers for both wave equation and heat equation. We see that improved Euler gives a slightly more stable result than forward Euler, whereas backward Euler produces the most smooth result. In theory, iterative approximation for backward Euler is slow; so whenever possible, it is faster to implement backward Euler by first converting the problem into a linear system represented as sparse matrix and then solve it. However, our experiments show that solving linear systems may not be as fast as iterating for a few iterations.

### Summary of Wave & Heat Equation Solvers We Implement
```
forward_euler:
1. forward_euler_wave
2. forward_euler_heat

improved_euler:
1. improved_euler_wave
2. improved_euler_heat

backward_euler:
1. backward_euler_wave
    a. backward_euler_fixedpoint_wave
    b. backward_euler_newton_wave (default)
2. backward_euler_heat
    a. backward_euler_fixedpoint_heat
    b. backward_euler_newton_heat (default)
    c. backward_euler_matrix_heat
```
The default for ```backward_euler_wave``` is to use ```backward_euler_newton_wave```. In order to see the effect of ```backward_euler_fixedpoint_wave```, select the corresponding option in ```backward_euler_wave``` and recompile the code. In order to see the effect of ```backward_euler_fixedpoint_heat``` or ```backward_euler_matrix_heat```, select the corresponding option in ```backward_euler_heat``` and recompile the code.

### Key Commands
| Command   |     Key      |
|----------|:-------------:|
| Enter animate mode |  A |
| Enter wave action |    W   |
| Change simulation to use Forward Euler | F |
| Change simulation to use Backward Euler | B |
| Change simulation to use Improved Euler | Y |
| Change simulation to use Symplectic Euler | S |
|Increase thermal diffusivity for Physical Simulation	|SHIFT + .|
|Decrease thermal diffusivity for Physical Simulation	|SHIFT + ,|
|Increase temperature at selected vertex	|.|
|Decrease temperature at selected vertex	|,|
To play heat diffusion simulation, press SPACE in Wave action.
The other key commands are same in the [user guide](http://15462.courses.cs.cmu.edu/fall2016/article/14) in Scotty3D.

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
