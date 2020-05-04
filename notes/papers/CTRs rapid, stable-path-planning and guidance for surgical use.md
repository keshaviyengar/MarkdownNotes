---
tags: [paper]
title: 'CTRs rapid, stable-path-planning and guidance for surgical use'
created: '2019-09-19T08:36:30.145Z'
modified: '2020-04-30T15:30:32.608Z'
---

Concentric Tube Robots: Rapid, Stable Path-Planning and Guidance for Surgical Use
============
* CTRs have non-intuitive kinematics, making teleoperation challenging
* Collaborative control schemes, with repulsive and attractive force feedback based on intraoperative path planning can simply 
* Computationally efficient algorithms are required to perform rapid path planning and solve IK 
* Previously long periods of pre-computation to establish kinematic lookup tables 
* Presents a high performance robot kinematics software architecture used along with a multi-node computational framework to rapidly calculate dense path plans for safe telemanipulation 
* Proposed architecture allows on the fly, incremental inverse kinematics estimation and interactive rates designed for multicore CPUs 
* Effectiveness quantified with metrics in simulation inspired from neurosurgery for hydrocephalus treatment 
* Active constraints can be generated on the fly and support operator in faster and more reliable operation

### CTRs for Surgical Applications ###

* Single / natural orifices for minimally invasive surgery well suited for continuum robots because of shape flexibility and manipulation to conform to transversed anatomy 
* Pre-curved tubes in CTRs interact to create a final robot shape 
* As number of tubes in robot increases, redundant configuration space can be exploited to simultaneously achieve shape control and tip manipulation 
* Computational design of CTR can deliver optimal robot architectures for different surgical tasks 
* Computational requirements for safe IK increase with number of robot tubes, supporting the need for quick and efficient solvers 
* Models based on tube mechanics have been developed for FK and IK of CTRs
    * Advanced models account for torsional wind-up along length of each tube as well as instability configurations arising from torsion 
    * Energy accumulated through wind-up can rapidly release in certain tube configurations and cause uncontrollable motion 
    * Unstable configurations need to be actively avoided 
    * Detection is additional computation cost and constraints on IK 
* Real-time, safe path-planning for custom designed CTRs have benefits 
* Empowers operator with reliable telemanipulation approach and ensuring avoidance of anatomical collisions and dangerous unstable configurations 
* If real-time is guaranteed, possible to make custom tubes for specific patients using rapid workspace optimization 
* Plan-to-deployment period can reduce intraoperative complication and morbidity, eg ruptured aneurysms 
* Coupled with ACs and virtual fixtures to form cooperative robotic manipulation controllers to gently steer operator to estimated safe paths 
* Shared control approach, operator has no idea of UK and allows seamless collision avoidance and unstable configuration 
* Fast solvers for robot IK are primary building block 
* Researchers have achieved real-time kinematics by creating look-up tables and employing root find approaches to estimate joint values
* Also Jacobian based approximations 
* Precomputation of dense path plans and sparse plans have been proposed 
* Solvers do not consider stability and prune the robot design space o avoid unstable robots 
    * Has been shown that this heavily limits combinations, and unstable CTRs, may well be the only robot that can perform certain interventions
* Precomputation assumes a static anatomy and complicates the intraoperative adaptation of said plans when anatomy is in motion 
* Precomputation for achieving interactive-rate kinematics solution adds hours of overhead computation 
* Software architecture takes advantages of distributed computing resources and multicore CPUs to provide globally dense workspaces within minutes 
* Memory optimal representations of CTRs and IK problems cam leverage dense workspace for on demand path planning with local inverse kinematics 
* Contributes to implicit ACs (IACs) which are algorithmically generated and tailored to respective task 
* IACs are more complex to define and estimate intraoperatively, further increasing benefit of real time safe effective robot guidance 

### Robot Kinematic Constraints ###
* Sections of CTRs can be variable curvature (VC) and fixed curvature (FC)
* A VC section consists of two tubes which rotate invidually and translate in tandem (3 DOF)
* FC section is a single tube and has 2 DOF
* The ith tubes translation
$_i\phi$ and base rotation is $_i\alpha^B$
* Joint space $q$ is described as a set $\{_i\phi `, {_i}\alpha^B\} \quad i \in [1,N_t ]$ where $N_t$ is number of tubes
* This work estimates shape of CTR based on unloaded torsionally compliant kinematics model
* Rely on computationally demanding iterative solving of a boundary value problem, beginning at torsion-free tip angle ${_i}\alpha^T$, differential equations are iteratively solved via Euler approximations (discretization step $\epsilon_{arc}$ to obtain base angle $_i\alpha^B$ for each tube
* Shape is calculated with matrix exponentials and curvature along robot's centre line

### Anatomical Constraints ###
* Important to avoid contact with anatomy (denoted as $\Gamma$)
* Rasterized polygons (maximum lattice of $\epsilon_{lat}$ provide representation of $\Gamma$ as set of 3D points
* Stored in a k-d tree for rapid distance queries between shape and $\Gamma$
* At each point along robot center line
    * maximum tube diameter $_{i_{cl}}r^t$ (external surface of robot)
    * k-d tree queried to find nearest neighbor $i_{i_{cd}}d^{nn}$ of every robot point to $\Gamma$
    * Given tube radii, $_{i_{cl}}r^t$, discretization step $\epsilon_{arc}$ and mesh lattice $\epsilon_{lat}$, distance to anatomy $d_{ana}$
 > $$d_{ana} = \min_{i_{cl}} \left(_{i_{cl}}d^{nn} - ||\frac{1}{2} [\epsilon_{arc}, \epsilon_{lat}]^T||_2 - {_{i_{cl}}r^t} \right)$$
> $$collision \iff d_{ana} \leq 0$$

### Stability Constraints ###
* Quantitative measure of stability is used, demonstrated that instability manifests as an S-shaped curve relating the tube rotation at base $\alpha^B$ to tube rotation at tip, $\alpha^T$
* Stable CTRs do not exhibit negative slopes on s-curve giving rise to measure of stability $d_{sta}$
> $$d_{sta} = \frac{\pi}{2} - atan2(1, \sigma^q)$$
* where $\sigma^q$ is the minimum inverse slope along the entire s-curve
* The configuration is unstable iff $d_{sta} < 0$
* Advantage of having a quantitative measure of stability is ease of generation of cost functions for inclusion in optimization-based inverse kinematics and thresholds for accepting or rejecting robot configuration under examination

### Rapid Forward and Inverse Kinematics ###
* describes the software architecture of high performacne to iteratively solve bvd with stability and anatomical constraints, C++ 14 standard
    1. memory allocation
    2. Memory alignment and random access memory
    3. m cache misses
    4. cache cherency
    5. branching and conditionals
    6. dynamic dispatch (virtual functions)
    7. vector instructions
* 1-4 relate to memory access bottlenecks. Dynamically allocating memory is reserved in heap, time consuming and avoided. Timings depend on where data is stored, magnititudes faster to store in cahce.
* 5-7 bottlenecks related to instructions. Branching means the sequence of instructions depend on conditions that are evaluated at runtime. If the processor mispredicts the branch, needs toflush and reload instruction pipeline. Dynamic dispatch (virtual functions) have overhead for each function call. Vector instructions operate on an array of data (single instruction lots od data). Using vectors with expensive operations (trig) can speed up exection
* Memory usage depends on number of sample points along robot centre line and architecture (type of tubes, number of tubes)
* Fixing number of sample points allows for preallocation of maximally needed memory
* Replace conditional and loops with equivelent constructs that are resolved at compile time

### Robot Architecture Using Template Class ###
* Using a template class provides a fully defined robot kinematic architecture at compile time
* Compilation time might be longer, but the transfer from runtime to compilation is important
* An abstract base class has limitd interface of elevator functions so computational overhead is marginal
* Variadic-template class derives from this base class and generates components of a CTR (FC or VC sections)
* Many instances of this robot class will be compiled , expecting different robot architectures
* Eg. If maximum number of sections is 2, six different robot class codes will be generated coresponding to different architectures for two sections {{FC}, {FC, FC}, {FC,VC},{VC,FC},{VC,VC}}. Number of class codes generated is $n_{class} = 2^{n_{max,sec}+1} - 2$. The maximum number of sections is 8, therefore in worst case $n_= 8$ 510 codes are genereated in less than a minute

### Unrolling Loops at Compilte Time ###
* Iterating over tuples of sections requires the indexes of elements, at compile time, to be accessed (such that the index is a template parameter)
* Every loop needs a iteration bound at compile time
* Standard approach is to use recursive template calls, acting on the next element in the tuple
* Developed a dedicated convienve struct:static for
* Resultss in a call to pointed function with current iteration index and a recursive call to static for with an incremented (inc >0) or decremented (inc <0) iter value
* This unrolls the for loop at compile time into instructions can be vectorized
* Last template parameter to ensure a stop criterion
* To use the function parameter as a parameter, the iteration index was packaged into an integral_constant class
* Using generic $\lambda$ loop index in the function can be retrieved from the declaretype of function parameter as a constexpr, because it is a parameter type instead of a parameter value

### Resolving Conditionals at Compile Time ###
* Runtime performance decreased by lots of machine-level jumps created by if/else
* Minimize number of jumps with static_if_else
* Two function pointers passed to static_if_else struct, first is if instructions and second is else
* Depending on the template parameter, correct branch is selected at compile time
* Using static_for and static_if_else to compile CTR classes codes, all combinations of sections, every single class can be individually optimized by compiler

### Guidance via IACs ###
* With the architecture described IACs can be generated
* Path plans are generated with precomupted road maps and navigation goal positions described by a clinician
* Precomputation can happen in a matter minutes, rather than hours
* In precomputation step, random configurations of robot generated in parallel (parallel computation)
* Path planner efficency relies on developed multinode framework
* A central computer controls computing clients and generates random robot configurations samples and solves the IK using different techniques

### Probabilitic Road Map ###
* Constraint generation framework is based on undirected graph $G$, with vertices $v_i \in G$
* Vertices represent random stable collision free CTR configurations, edges $e_{i,j}$ represent transitions possible transitions among configurations
* Graph queried using A* grap search to extract shortest paht between current configuration and configuration corresponding to tip pose
* Euclidean norm between two vertex postions is admissible A* heuristic
* Extracted series of configuration generates a guidance path along with which an operator is guided

### Precomputation of the Graph: Generation of the Road Map ###
* To find safe robot configurations, the server controls parameters defining configuration samping (density) $(\gamma_{\phi}, q)$ and configuration acceptance $(d^{thres}_{ana}, d^{thres}_{sta})$
* Since we need uniformly distributed configurations in task space, joint space sampling has to be non-linear
* Extended robot configurations are most likely rejected as they would collide with anatomy, leading to bias of shorter robot that needs to be accounted for
* Using a randomly uniformly distributed number $q_{r,u} \in [0,1]$ for a tube with maximum extension of $_{i}\phi^{max}$, translation/extension joints are scaled with $q_r = {_i}\phi^{max} q_{r,u}^{\gamma_{\phi}}, \gamma_{\phi} \in [0,1]$
* Extent of sampling elongated vs. rectracted is governed by $\gamma_{\phi}$
* Each client calculates the FK for the given joint value, determines $d_{sta}, d_{ana}$ and sends to server iff 
    > $d_{col}(q_r) \leq d^{thres}_{col} \land d_{sta}(q_r) \leq d^{thres}_{sta}$
* When the server has received a minimum number of configurations, a local planner calculates edges $e_{i,j}$ and cost for transitioning from one robot configuration ($v_i$) to another $v_j$
* Edge generation is two stages, fist the cost for a potential edge $e_{i,j}$ has to be below a threshold, and scond, only edges with $N^{max}_s$ smallest costs are introduced
