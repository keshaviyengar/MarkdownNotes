---
tags: [paper]
title: Learning the Complete Shape of Concentric Tube Robots
created: '2020-05-03T17:31:05.924Z'
modified: '2020-05-04T09:30:42.064Z'
---

Learning the Complete Shape of Concentric Tube Robots
============
A. Kuntz, A. Sethi, R.J. Webster

### Abstract
* CTRs are nested, precurved  tubes for MIS at difficult to reach sites of human body
* Shape estimation is needed to safely peform surgeries
* State of art physics models cannot account for complex physical phenomena, not accurate for surgery
* Learned model to estimate shape
* Trained with mixture of physical and simulated data
* Evaluate multiple architectures and demonstrate shape accuracy

### Introduction
* Rotation and translation allows to curve around obstacles
* Application include skull base, lungs and heart
* Motion planning is simulated with robot motion with collision dectection of anatomy with robot geometry
* Hard to model full shape with joint configuration because of friction, material properties etc.
* NN takes input of robot configuration and outputs coefficents for orthonormal polynomial basis functions in $x, y, z$ parameterized by arc length along robot's tubular shaft
* Entire shape is predicted by feed forward pass of NN
* Arc length is independent of curvature and torsion (main sources of error)
* Transfer learning used to train simulation results and pass to real world data

### Related Work
* CTRs have hard, unintuitive control manually
* Most strategies involve desired tip movements, however collision free motion is difficult to compute
* Motion planning allows for global view in constrained anatomy. Simplified kinematics and sample based planning have been proposed
* An accurate shape model, mapping from control variables to geometry in the real world is required
* The shape can be sensed online using FBG sensors but we want to plan safe trajectories so require a method that computes shapes of the robot in advance
* Physics based models depend on Coserot rod theory and getting very complex with inclusion of torsion
* NN approaches ahve been applied but only consider robot pose

### Method
* We consider the problem of mapping concentric tube configuration, defined by rotation and translations, to geometry along entire length in the real world
* NN is trained on a combination of real-world and simulated data with configuration as input and coefficents for an arc length parameterized space-curve function to represent backbone
* Add figure 1

#### A. Ground Truth Data Generation
* Multi-view 3d computer vision shape from sillouettte where multiple images are collected from cameras with known position and orientation
* Two cameras, roughly orthogonal. Robot is mvoed in a sequence of random configurations and images are taken simulteneously
* For each image pair, segmented shaft with color theshold. Segmentation wise pixel ray tracing from camera is done with calibrated cameras
* Where rays intersect voxels from both cameras indicate robot shape
* Least squares fit in 3D space results in sensed backbone shape as ground truth
* Add Figure 2

#### B. NN Model
* Feed forward fully connected network because of simplicity
* Parametric rectified linear units as activation function between layers (slight improvement over standard retified linear unit)
* Inputs of the network: For a robot of k-tubes, parameterize ith tube state as
$$y_i = \left { \gamma_{1,i}, \gamma_{2,i}, \gamma_{3,i} \right} = \left { \cos \alpha_i, \sin \alpha_i, \beta_i \right}$$
* $\alpha_i \in \left(-\pi, \pi \right]$
* Robot configuration $q = (\gamma_1, \gamma_2, \dots, \gamma_k)$
* Outputs of the network are 15 coefficents $c_{1x} ,c_{2x}, \dots, c_{5x}, c_{1y}, c_{2y}, \dots, c_{5y}, c_{1z}, c_{2z}, \dots c_{5z}$
* Coefficents for a set of 5 orthonormal polynomial basis functions in $x, y, z$ parameterized by arc length
* Results in three function $x(q,s), y(q,s) and z(q,s)$, where
* $x(q,s) = len(q) \times (c_{1x}P_1(s) + c_{2x}P_2(s) + \dots + c_{5x}P_5(s))$
* $s$ is the normalized arc length parameter between 0 and 1 and len(q) is total length of robot backbone in configuration q
* $y(q,s) and z(q,s)$ are defined similarly
$$shape(q,s) = <x(q,s), y(q,s), z(q,s)>$$
* Resulting space-curve function can be evaluated at any arc length resulting in full geometry with robot diameter as a function of arc length
* Gram-Schmidt orthogonalization

#### C. Training Model
* 100,000 data points sampled uniformly from configuration space using physics model
* Real world data sampled uniformly and found with silloute method
* Split into three sets 7,000 training, 1,000 validation and tset set of 1,000
* Sum of least squared distances loss function and ADAM
* Early exit: If 10,000 epochs pass with no improvement as evaluted by validation, stopped and that model is used

### Results
* Table 2 shows tube parameters

#### A. Evaulation of Polynomial Basis Functions
* First, compute optimal set of of coefficents for 100,000 pre-training data points with least squares
* Evaluated resulting shape at 20 equally spaced point along backbone
* Calculated maximum $L^2$ distance over 20 points for each 100,000 configurations
* Mean $L^2: 0.044 +- 0.037$mm

#### B. Evaluation of Varying Network Architectures
* Trained multiple architectures varying number of hidden layers from 3 to 7 and number of hidden nodes 15 to 60
* Compared sim only,sim + real and real only data training
* Tested on set of 1,000 data points
* Performed a forward pass with configuration to determine coefficents and evaluted against ground truth of 20 evenly spaced points
* Error metrics:
* 1. Maximum deviation (L2) distance
* 2. Mean L2 error
* 3. Sum of devation of L2 error
* Both real and sim + real outperformed simultion only in all architectures for all error metrics
* Sim+real outperforms in all metrics except in arcitecture 5x15
* Best model is sim+real with 3 hidden layers and 30 nodes

#### C. Accuracy Comparison to Physics Based Model
* Histogram plot of the 1,000 test points (show figure)
* Indicates that the learned model error distribution is likely to produce loer error values, also lower minimum, maximum and mean errors
* Figure 7 plots error at point index with errors increasing as we get closer to the tip

#### Timing Comparison to Physics-Based Model
* Computation must be fast and NN can be batched
* Batching allows for many shape computations to happen parallely that physics based models cannot do
* Figure 9 shows number of hidden layers increases time but more importantly batch size drastically lowers computation time per configuration
* Without batching still faster than physics modelling

### Conclusion
* NN model that outputs arc length parameterized space curve
* Deviates from real ground truth less than physics model and is faster computation
* Investiage minimal force exertation on tissue (endoscope, delivery of ablation)
* Future work is to train models that account for tissue interaction at tip and along shaft
* Investigate why only a minor improvement when combining sim+real versus just real
* Integration of physics based with dda driven approach
* Investigate various loss functions and RNNs and hybrid networks
* Sampling distribution of configuration space affects results
* Ideal network size depend on number of tubes
* Augment learning to include other modelling error like hystersis
* Other basis functions other than orthonormal polynomials
* Integrate learned model to motion planner and evaluate avoidance during teleoperation
* Could also be used for other robots like tendon driven robots etc.





















