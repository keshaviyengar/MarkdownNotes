Learning to drive in a day
============
* First application of DRL in autonomous driving
* Learn a policy of lane following in handful of training episodes with single monocular image
* Easy reward, distance travelled by the vechicle without safety controller engaging
* Continouous, model-free DRL with exploration and optimization peformed onboard
* New framework to move away from reliance on logical rules, mapping and direct supervision

### Introduction
* Most apporahces for autonomous driving is logic driven and annotated 3D geometric maps, difficult to scale
* Authors advocate for comprehensive system of understanding
* Generality of RL makes it a useful framework to apply to autonomous driving
* This paper
    1. Poses autonomous driving as an MDP
    2. Show that a canonical RL algorithm can rapidly learn a simple driving task in simulation
    3. Learn to drive in real-life in a few episodes, using only on-board computation

### Related Work
* Closest work is imitation learning
* Most work to navigate safely in complex environments use independaant components of perception, mapping ,state estimation, planning and control. Since each component needs to be tuned, difficult to scale
* Focus has been on computer vision. Localization facilitates control of vechile within mapped area, segmentation can intrepret scenes
* Imitation learning:
    * Learns a policy by observing expert demonstrations
    * Can be use end-to-end deep learning, tuning all components at once
    * Hard to get every potential scenario so hard to scale
* RL:
    * Set of states, actions, transition probability function, reward function and future discount factor
    * Aim is to learn a policy $\pi$ that obtains a high cumulative reward
    * In model-based, explicit models for transition and reward functions are learned, in model-free, directly estimate Q-value of taking action $a$ in $s$
    * Model-based tend to be more data-efficent than model-based ones
* Closest work is training an RL agent to follow a GPS trajectory in an obstacle-free enviornment using a dense reward function

### System Architecture
#### State space
* This work shows a simple monocular image with observed speed and steering angle is succificent for learning simple driving task
* Could use a CNN or an VAE for treat the image. These are compared in section 5

#### Action space
* Continous actions provide a smoother controller, although ahrder to learn. 2-dimensional action space, steeering angle [-1, 1] and speed setpoint in km/h

#### Reward function
* Reward is based on forrewad speed and terminate if an episode infracts a traffic rule
* Value of a given states corresponds to average distance travelled before an infraction

### RL algorithm DDPG
* Use proritised experience replay
* Use OU for exploration:
    * Tuning of hyperparameters can be tricky, higher variance noise provides better state-action converge while strongly mean reverting noise with low variance is easy to anticipate

### Task-based Training Architecture
* Architecture has 4 tasks, train, test, undo and done
* Train and test allow interaction with vehicle in autonomus mode, executing policy, only difference is noise addition in training. In early episodes, skip training for exploration
* Episode termination is when automation is lost. In real-life, human driver needs to reset the vehicle to valid starting
* System can terminate for variety of reasons other than failing to drive correctly, should not be considered for training. Undo can undo last episode and restore model


### Experiment
* Main task is lane following on simlation and real-life
* With image input, without knowledge of lane position
* BOth sim and real-life, a CNN is used with 3x3 kernel, stide of 2 and 16 feature dims
* VAE, a decoder of same size encoder used

#### Simulation
* 3D driving simulation is developed in Unreal engine with generaltive model
* Could reliably learng in 10 training episodes and little advantage to using compressed VAE

#### Real-world driving
* Main difference is uncontrollable environmental factors and implementation of safety control systems
* 250 meter section of road. Episode terminated by safety driver and is returned to center of lane for next episode
* Single monocular forward facing camera

#### Discussion
* Able to learn lane follow within 30 minutes of training
* To tune hyperparameters, as simulated enviornment was used, and found to be transferable to real world
* VAE greatly improves the performance of DDPG in real world driving
* Semi-supervised learning and domain trasfer could improve data availablity for RL
* Improvements ahve been made to RL apporahces
* Model-based RL is an exiciting avenue as well