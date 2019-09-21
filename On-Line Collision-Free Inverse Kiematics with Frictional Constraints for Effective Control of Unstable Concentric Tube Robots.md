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
* The stability cost is $S(q, \Tau) = \begin{cases}1, & \text{if}\ \text{unstable} \\ 0, & \text{if}\ \text{stable}\end{cases}$