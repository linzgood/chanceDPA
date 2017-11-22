# chanceDPA

## About this project
Dynamic Programming Algorithm (DPA) to solve a problem with joint probabilistic constraints. It can be used to solve a path planning problem where the chance of hitting an obstacle needs to be kept below a threshold. This project implements this paper [1] and can be directly used to efficiently solve problems with a large state-action-noise space.  <br />

## Path planning problem definition 
As in [1], the code solves a path planning problem with the following dynamics:<br />
![x_{k+1}=f_k(xk,uk,wk)=xk+uk+wk](https://latex.codecogs.com/svg.latex?x_%7Bk&plus;1%7D%20%3D%20f_k%28x_k%2Cu_k%2Cw_k%29%20%3D%20x_k%20&plus;%20u_k%20&plus;%20w_k)<br />
![norm(u_k), wk_distrib](https://latex.codecogs.com/svg.latex?%5Cleft%20%5C%7C%20u_k%20%5Cright%20%5C%7C_2%5Cleqslant%20d_k%2C%20%5C%20%5C%20w_k%5Csim%20N%280%2C%5Csigma%5E2I%29)<br />
As shown in the paper [1], this model is general enough to solve a Mars entry, descent and landing problem.
Note that for the path planning problem considered here, ![xk](https://latex.codecogs.com/gif.latex?x_k) consists of 100x100 states, ![uk](https://latex.codecogs.com/gif.latex?u_k) can take up to 81 different values using our discretized grid and the noise distribution increases the computations required to apply the DPA, requiring an efficient solution to the problem.<br /> <br />
Below is the map used for the simulation:<br />

![alt text](https://github.com/thomasjlew/chanceDPA/blob/master/imgs/map.png)  <br /> <br />
As in [1], we use ![sigma167](https://latex.codecogs.com/svg.latex?%5Csigma%3D1.67). Therefore, to minimize the complexity of the problem, we approximate ![wk](https://latex.codecogs.com/svg.latex?w_k) to discretized values, with ![abs(wk)<5](https://latex.codecogs.com/svg.latex?%7Cw_k%7C%3C5).  <br /> <br />
The cost-to-go is defined in this way:  <br />

![gN](https://latex.codecogs.com/svg.latex?g_N%28x_N%29%20%3D%20%5Cbegin%7BBmatrix%7D%20%5C%20%5C%200%20%5C%20%5C%20if%20%5C%20x_N%3Dx_G%5C%5C1%20%5C%20%5C%20otherwise%20%5Cend%7Bmatrix%7D%5Cleft) <br />
![gk](https://latex.codecogs.com/svg.latex?g_k%28x_k%2Cu_k%29%3D%5Calpha%5Cleft%20%5C%7C%20u_k%20%5Cright%20%5C%7C) <br /> <br />
We also define the Lagrangian which needs to be minimized and a variable indicating whether a state is an obstacle or not: <br />
![Lk](https://latex.codecogs.com/svg.latex?L%5E%5Clambda_k%28x_k%2Cu_k%29%3A%3Dg_k%28x_k%2Cu_k%29&plus;%5Clambda%20I_k%28x_k%29%2C%20%5C%20%5C%20k%3D1%2C%20...%2CN) <br />
![Ik](https://latex.codecogs.com/svg.latex?I_k%28x_k%29%3A%3D%5Cleft%5C%7B%5Cbegin%7Bmatrix%7D%20%5C%20%5C%20%5C%201%2C%20%5C%20%5C%20if%20%5C%20x_k%20%5Cin%20obstacle%20%5C%5C%200%2C%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%20%5C%20otherwise%20%5Cend%7Bmatrix%7D%5Cright.)<br />

To compute efficiently the expected cost value for each state and action, we convolve the costs for each action with a probability filter ![w_kern](https://latex.codecogs.com/svg.latex?w_%7Bkernel%7D), since<br />
![Jk_fixeduk](https://latex.codecogs.com/svg.latex?J%5E%5Clambda_k%28x_k%29_%7B%5Cmu_k%3Du_k%7D%3D%5Cunderset%7Bw_k%7D%7BE%7D%5Cleft%20%5C%7BL%5E%5Clambda_k%28x_k%2Cu_k%29&plus;%20J_%7Bk&plus;1%7D%28f%28x_k%2Cu_k%2Cw_k%29%29%20%5Cright%20%5C%7D)<br />
![Jk_Exp_sep](https://latex.codecogs.com/svg.latex?J%5E%5Clambda_k%28x_k%29_%7B%5Cmu_k%3Du_k%7D%3DL%5E%5Clambda_k%28x_k%2Cu_k%29&plus;%5Cunderset%7Bw_k%7D%7BE%7D%5Cleft%20%5C%7B%20J_%7Bk&plus;1%7D%28f%28x_k%2Cu_k%2Cw_k%29%29%20%5Cright%20%5C%7D) <br />
![Jk_sump(wk)](https://latex.codecogs.com/svg.latex?J%5E%5Clambda_k%28x_k%29_%7B%5Cmu_k%3Du_k%7D%3DL%5E%5Clambda_k%28x_k%2Cu_k%29&plus;%20%5Csum_%7Bw_k%7D%5E%7B%20%7Dp%28w_k%29%5Ccdot%20J_%7Bk&plus;1%7D%28f%28x_k%2Cu_k%2Cw_k%7Cw_k%29%29) <br />
![Jk_conv](https://latex.codecogs.com/svg.latex?J%5E%5Clambda_k%28x_k%29_%7B%5Cmu_k%3Du_k%7D%3DL%5E%5Clambda_k%28x_k%2Cu_k%29&plus;%20w_%7Bkernel%7D%5Cast%20J_%7Bk&plus;1%7D%28f%28x_k%2Cu_k%29%29) <br />

This avoids the need to use a stochastic approach by applying different ![wk](https://latex.codecogs.com/svg.latex?w_k) values, which would be averaged (Monte-Carlo) to obtain an estimate of the cost for each action. This approach (tested) is indeed computationally too expensive. <br />



## Results
Dynamic programming algorithm solving the problem for each timestep, with ![lambda](https://latex.codecogs.com/svg.latex?%5Clambda) = 1e-4.<br /> <br />
![alt text](https://github.com/thomasjlew/chanceDPA/blob/master/imgs/chanceDPA.gif) <br /> <br />

Solution using the Dynamic Programming Algorithm, with ![lambda](https://latex.codecogs.com/svg.latex?%5Clambda) = 1 (important penalization when hitting an obstacle).<br /> <br /> <br /> <br />
![alt text](https://github.com/thomasjlew/chanceDPA/blob/master/imgs/cost%26policy_lambda1.png) <br /> <br />
Typical costs results after N=50 steps, lambda = 0 (no chance constraint). <br /> <br />
![alt text](https://github.com/thomasjlew/chanceDPA/blob/master/old_files_201117/cost.png) <br /> <br />

### Execution time
The execution of the Dynamic programming Algorithm without evaluating the risk for the given policy currently takes approx. 3.8 sec (for N=50 steps on 100x100 states, with 81 inputs + noise, tested on a Intel Core i5-4210U CPU @ 1.70GHz, 4GB of RAM).

## Further work
- DONE The ongoing work aims at improving the performance of the dynamic programming algorithm to allow for more iterations.
- DONE Implementing the rest of the paper (to minimize the risk) is ongoing.
- TODO Implementing Brent's method for efficient interval computation for lambdas. Currently uses average returning non optimal but compliant result (for this path planning problem).
## References
- [1] M. Ono, M. Pavone, Y. Kuwata and J. Balaram, "Chance-constrained dynamic programming with application to risk-aware robotic space exploration", Autonomous Robots, 2015.
- [2] D.P. Bertsekas, "Dynamic Programming and Optimal Control", Vol. I, 3rd edition, 2005, 558 pages.
- [3] D.P. Bertsekas, "Nonlinear Programming", Second Edition. Athena Scientific (1999)
