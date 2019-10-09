On Merits of Using Joint Space and Orientation Representations in Learning the Forward Kinematics in SE(3)
============
Grassman and Burgner-Kahrs

### Abstract
* Investigates the influence of different joint space and orientation representations for approximating forward kinematics
* Consider all DOF in three dimensional space SE(3) and robot joint space $Q$
* Different shallow ReLU designed with bias and weights normalized
* Results show quaternion / vector pairs outperform other representations w.r.t approximation
* Tests done on a Stanford arm and CTR
* With quaternion and vector pairs, approximation is 7 and 3 times more accurate
* By using a 4-parameter orientation representation, position tip error is less than 0.8% w.r.t to robot length (where state of the art is at 1.5%)
* Other 3 parameter representations of SO(3) cannot achieve this
* Any set of euler angles is at best case 3.5% w.r.t robot length

### Introduction
* Rigid robots generally have easy kinematics
* CTRs are not rigid and have multiple telescopic super elastic tubes that interact as they rotate and extend with each other, resulting in highly non-linear behavuour
* Common model is to use Cosserat rods and solve numerically
* Torsion and friction result computationally expensive models (or hard to model), so FK remains challenging
* Machine learning techniques like NN can be used to approximate a function that represents the forward kinematics
* Robotics applications with regards to NNs have focused more on classification, such tasks are not sensitive to output since small distrurbances are not likely to change class
* In contrast to this application, where any small deviation results in large approximation errors
* Hypothesis that joint space and orientation representation has significant impact on input-output characteristics of FNN
* Lastly, most studies on FK approximation focus on position and not orientation, which could be a drawback in real-life scenarios
* This works investigates impact of different joint and orientation for learning FK, on a rigid link robot and kinematically complex robot (CTR). All six 6-DOF in SE(3) as well as 6-DOF for joint space are considered. Following contributiosn are made:
    * Different joint space and orientation representations are empirically compared indicating that even in low dimensional problems, represenation is advantageos
    * Pose in SE(3) both position and orientation is learned with no simplification
    * Simple and effective transformation leads to fast sampling methods considering inequalities and positively effects approximaion results
    * Minor contribution of a cylindrical joint representation

### Methods
* Utilization of ANNs briefly described. Different joint representations of SO(3) and provide approximation errors used a performance criteria. Formulate three different joint space representations

#### A. Artifical Neural Networks
* Feedforward NNs are used. Can approximate a smooth function in a compact set (can approximate FK)
* Radial Basis Function (RBF) could be used but the curse of dimensionality as RBF unit only covers a small local region of input space
* ReLU activation function used over tanh because of fast training and computational efficiency $\phi(x) = max(0,x)$
* Linear activation functions are used at the output layer
* Weights are initialized by HE-initialization but for linear activiation function weights and biases are initialized by uniform distribution
* Adam optimizer with mini-batch size $N_{bs} = 128$ used to optmize weights $w$ and biases $b$. Batches extracted randomly from training set $S_{tra}$ in each epoch $N_{ep}$  
* Adam optimizer had no under or over fitting and converged faster than the vanilla gradient descent

#### B. Parameterization of SO(3)
1. Euler angles:
    * Minimal number of parameters typically 3 angles. Set of 3 valid angles are Euler angles. 12 different sets of euler angles, depending on sequence. Notation used here is numbers, 123 represents a XYZ euler angle representation

2. Vector parameterization:
    * For a stictly increasing smooth scalar function $f(\theta)$ defined in semi-open interval $] -\theta, \theta ]$ with $f(0) = 0$ and $theta \geq 0$, orientation displacement can be represented as:
    > $$r = f(\theta)n$$,
    * where $n = (n_x, n_y, n_z)^T$ and $\theta$ are the unit vector and rotation of an equivelent angle/axis representation 
    * Two subclasses of representation, namely $f(\theta) = \mu tan(\frac{\theta}{\mu})$ and
    >  $$f(\theta) = \mu sin(\frac{\theta}{\mu})$$
    * The sine function is chosen because its bounded between +1 and -1. $\mu$ is set to 4 it allows to consider all angles $0 \leq \theta < 2 \pi$
    * Angle $\theta$ and axis $n$ can be calculated with algebra
    > $$\theta = 4 \arcsin(\frac{||r||_2}{4})$$
    > $$n = \frac{r}{||r||_2}$$
    * Cannot be recovered if $\theta = 0$ as that indicates $||r||_2=0$

3. Quaternion:

