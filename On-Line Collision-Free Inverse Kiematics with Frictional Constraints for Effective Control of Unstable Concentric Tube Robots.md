On-Line Collision-Free Inverse Kinematics with Fricitional Active Constraints for Effective Control of Unstable Concentric Tube Robots
============
K. Leibrandt, C. Bergeles, and G.Z Yang

### Abstract
* CTRs are ideally suited for navigating along natural anotomical pathways
* Telemanipulation requires on-line computation of UK with simulanteous avoidance of anatomical obstacles
* Unstable configurations should be avoided
* Existing work as explored Jacobian approximation and configuration precomputation
* This paper leverages state-of-the-art multi-core computer architectures to deliver real-time local IK solutions using established models, with collision and instability avoidance
* Also considers frictional active constraints, viscoelastic force fields that move it away from obstacles and towards safe configurations
* Demonstrated in realistic clinical scenarios

### Introduction
* CTRs have been demonstrated to have unique benefits in senstive anatomical cavities
* CTRs have shape control through relative rotation and telescopic translation of pre-curved super-elastic tubes
* Dimensions similar to catheters, propsed for vasculature, nasal or urinary tract entrace
* Mechanics have been extensively investivated, with recent effort on computatinally designing optimal patient and surgery specific robots
* Most methods consider the tube torsion but few consider elastic instabilities from twistng elongated highly curved tubes
* Instability considerations are curcial, since it causes a sudden release of elastic energy in an instanteous change in robot configuration
* Real time solutions are equally important since it is a requirement for intraoperative use
* Previous works have proposed precomputation of the configuration space and IK through root finding
* Computes dense path plans for quick global navigation through anatomy and local IK solver 
* Some update rates are 26m s, pre-computation can be as long as 6h
* Pre-computation is a requirement for robot-tip manipulation, else tip errors of 25 mm may arise
* Rapid algorithm deployment is challenging, espeically for dynamic anatomy
* Efficent Jacobians have been proposed to avoid pre-computation but these papers do not consider instability or collision avoidance
* The current paper demonstrates the use of multi-core architectures to accurately solve local IK accounting for instability and anatomical constraints
* Enables interactive rate robot manipulation in complex dynamic environments without precomputation via implicit path planning
* Results based on cascaded global and local optimizers ranked according to IK error metric
* No single optimizer converges to optimal result, making the cascaded implementated efficent and accurate
* Ananotmical-specific frictional active constraints to CTR telemanipulation, speeding up manipulation by generating force feilds to guide user
    * Evaluted in clinical scenarios

### Parallel Optimized Local IK
* CTR manipulation involves translating and rotating sections of super-elastic pre-curved tubes
* Sections can be comprised of 1 or 2 tubes to form a fixed or variable curvature section denoted by $\phi$
* Rotation of each tube, relative to outer tube is denoted by $\alpha$
* Tubes of a variable curvature section are rotated, curvature is controlled
* Constant curvature segments have single curvature
* Variable curvature has 3 DOF $(_{1}phi = _{2}phi, _{1}\alpha^{base}, _{2}\alpha^{base})$
* Fixed curvature sections have 2 DOF $(_{1}phi, _{1}\alpha^{base})$
* The set $\{ _{i}\phi, _{i}\alpha^{base} \}$ for $i=1, \dots, t$ where $t$ is number of tubes, forms the fk variables
* Manipulation requires the mapping between joint space $q = [\phi \, \alpha^{base}]$ and task space $x$
> $x_{EE} = f_{fk}(q)$
> $q = f_{FK}^{-1}(x_{EE}) = f_{IK}(x_{EE})$
* CTRs modelled with torsional compliance, have no closed form solutions for FK or IK
* FKs are a bvp, calculated by solving differential equations from a torsional-free tip angle in absence of external loads $_{i}\alpha^{tip} = 0$ for $i = 1, \dots, t$ to the base angles $_{i}\alpha^{base}$ and subsequentely estimating the shape using jointvalues and matrix expoentials
* To solve IK, Jacobian must be determined via efficent approximations or optimal solutions via root finding or shooting methods

##### Parallel Jacobian Approach with Null-Space
* Efficent Jacobian calculations for IK rely on mehcanics approximations
* Formulation of Jacobian gives ability for parallelization
> $$J = \frac{\Delta x}{\Delta q} = \begin{bmatrix}\frac{x(q+\frac{\Delta q_1}{2} e_1) - x(q - \frac{\Delta q_1}{2}e_1)}{\Delta q_1} \\ \dots \\ \frac{x(q+\frac{\Delta q_n}{2} e_n) - x(q - \frac{\Delta q_n}{2}e_n)}{\Delta q_n}\end{bmatrix}^T$$
where $e_i$ is the ith unit vector of the canonical basis of the n-dimensional joint space
* Columns of $J$ can be computed in parallel
* Pseudo inverse used to calculate joint velocities
> $$\dot{q} = J^{\dagger} \dot{x} + \left (I_6 - J^{\dagger} J \right ) \dot{q}_0$$
where $J^{\dagger}$ is the pseudo-inverse, $I_6$ is identity matrix and $\dot{x}$ is the tip velocity
* Since concentric tube robots often redudant, Jacobians null-space canbe used to achieve secondary goals
> $\dot{q}_0 = k_{w0} \left ( \frac{\delta w (q)}{\delta q} \right )$
where $k_{w0}$ is a goal weighting factor and $w(q)$ is the objective function for secontary goal
* To avoid singular robot configurations damped least squares (DLS) inverse $J^*$ is used at the expense of slower convergence in comparision to pseudo-inverse
> $J^* = U diag \left ( \frac{\sigma_1}{\sigma_1^2 + \lambda^2}, \dots, \frac{\sigma_n}{\sigma_n^2 + \lambda^2} \right ) V^T$
where $\sigma_i$ are the singular values of J resulting from SVD and $\lambda$ is the damping factor
$x_{EE} = f_{FW} (q + \dot{q} \Delta t)$

##### Optimisation Approach
* Other IK solvers use optimisers to minimize a cost function related to joint variables, desired tip pose, and constraints
* A scalar cost function $c(q, \Tau, x^B, \Gamma)$ is used where $q$ is the joint angles $\Tau$ is robot architecure, $x^B$ is desired tip pose and $\Gamma$ is anatomical and stability constraints
* Collision modelled as a linear continuous function, increasing with distance between robot and anatomy $d(q, \Tau, \Gamma)$, in the interval $[d_0, d_1]$
> $c_{Collision} \left( d(q, \Tau, \Gamma) \right) =  \begin{cases}0, & \text{if}\ d \leq d_0 \\ \frac{d-d_1}{d_0-d_1}, & \text{if}\ d_0 < d < d_1 \\ 0, & \text{if}\ d_1 \leq d \end{cases}$
* The stability cost is 
> $S(q, \Tau) = \begin{cases}1, & \text{if}\ \text{unstable} \\ 0, & \text{if}\ \text{stable}\end{cases}$
* The stability cost is caclulated for every rotational joint variable $\alpha^{base}$ for each tube $t$. If every tube pair is stable, then the configuration is considered stable.
* Overall cost function is computed as
> $c(q, \Tau, x^B, \Gamma) = \gamma_1 ||x_{EE}(q) - x^B|| + \gamma_2 arcsin(||z(q) \times z^B||) + \gamma_3 c_{Collision}(q, \Tau, \Gamma) + \gamma_4 S(q, \Tau)$
* The fist ter mis tip position error, second is tip orientation error, third is collision cost, and final is stability cost
* Choose weights ($\gamma_3$ and $\gamma_4$) sufficiently high,
> $\gamma_3 = \gamma_4 \geq 2 \gamma_1 l_{max} + \gamma_2 \pi
* Cost function is pretty similar in literature but optimization techinques differ
* Example include
    1. Branch and Bound, for the guidance of i-Snake
    2. Broyden-Fletcher-Goldfarb-Shanno, for guidance of concentric tube robots through tubular anatomy.
    3. Nelder-Mead simplex method for inverse kinematics and optimal concentric tube robot design.
    4. Generalized Pattern Search for inverse kinematics and optimal concentric tube robot design.
    5. Gauss-Newton for inverse kinematics.
* Variety of solvers indicates no best algorithm yet, fine tuning of parameters making use in clincal setting error-prone

##### Parallel System Architecture
* Parallel system architecture that leverages multi-core clsters to overcome single optimiser limitations
* IK problem does not conform well to single instruction multiple data (SIMD), GPUs not viable
* Different optimisation algorithms with different parameters independently and asynchronously eployed to solve local IK before best solution used for control
* Master node gives user force feedback and recieves user input such as commanded pose $x_B$ and control parameters
* Each node recieves the control parameters continously and provides to optimisation threads, which minimize a global cost function.
* Communication thread for each optimiser nodes broadcasts its best result
* Master node compares te costs with current cost and updates joint variables to reduce error
* Parallel structure of the sovlers can deliver online local IK solutions that account for instability and collisions for general clinical scenarios

#### Active Constraints
* Active constraints are there to help operator perceive environment and direct towards safe solutions through anatomy and goal based force fields

##### Frictional Forbidden Region Active Constraints
* FFRAC that steer telemanipulators away from the anatomy
* When only elastic forces are applied, unwanted autonomous motions or oscillations from the telemanipulator may occur
* To dissipate kinetic energy delivered by active constraints and users motion, elasto-pastic friction models that superimpose viscous forces on the constraints are considered
* This results in minimized oscillations by dmping excessive tool and manipulator velocities and are termed frictional forbidden region active constraints
* Forces superimposed to act are:
> $f_{FFRAC} = \sigma_0 z + \sigma_1 \dot{z} + \sigma_2 \dot{x}^B$
where $z$ is the elastic displacement, $\dot{z}$ is the velocity of elastic displacement, $\dot{x}^B$  is commanded pose tip velocity and $\sigma_{0,1,2}$ are scaling coefficent.
* In literature there is no upper bound on fricitional forces
* Scale is emperically defined upper bound $||f_{FFRAC}|| \leq f_C$
* Direction nand magnitude of elastic force depends on commanded tip position
* Three main zones, $a_1, a_2, a_3$:
    1.  $x^B$ is exterior to anatomy, elastic force is maximized and is attractive to anatomy border.
    2. $x^B$ is within anatomy and a specified safety transition zone, which case is repulsive
    3. $x^B$ is in a safe zone, no force is applied
