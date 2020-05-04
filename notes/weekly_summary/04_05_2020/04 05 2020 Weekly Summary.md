---
attachments: [exact_model_rollout_error.jpg, exact_model_success.jpg, simple_model_rollout_error.jpg, simple_model_success.jpg]
tags: [summary]
title: 04 05 2020 Weekly Summary
created: '2020-05-04T14:09:09.529Z'
modified: '2020-05-04T16:09:06.379Z'
---

# 04 05 2020 Weekly Summary

### Summary of Week
* Went through the [Deep RL Bootcamp course](https://sites.google.com/view/deep-rl-bootcamp/lectures) up to Frontiers Lectures
* Made some improvements to the callback function during training
* Geometrically exact model tube arbritrary
* Completed the transfer learning experiment for two tubes
* Some review in literature review (lots of work remain)
* Read the new Kuntz learning paper that learns CTR backbone shape
* IJCARS accepted!

### Implementations
* Geometrically exact n-tube artbritary
  * output $dy/ds$ variable needed to be changed with additional or less variables depending on tubes
  * $dy/ds=[\text{ntubes} \times 2, 3, 9]$
  * $dy/ds=[(u_z, \alpha), r, R]$
* Callback function
  * Refactored class with proper saving intervals for pcds, saving arrays and clearing arrays
  * Some work left to create new goal tolerance class. Implement when ready to experiment with curriculum learning

### Experimental Results
* Trained a constant curvature model 2 tubes successfully
* Weights set as constant curvature NN for geometrically exact experiment

Tube | $L_{curved}$ | $L_{total}$ | $x_{curvature}$ | $d_{inner}$ | $d_{outer}$ | Stiffness | Torsional Stiffness
--- | --- | --- | --- | --- | --- | --- | --- |
0 | 95.33 | 122.51 | 49.76 | 1 | 2.4 | 5e+10 | 2.3e+10 
1 | 58.34 | 107.48 | 12.66 | 3.0 | 3.8 | 5e+10 | 2.3e+10

* Constant curvature results
![simple model success](@attachment/02_04_2020/simple_model_success.jpg)
![simple model error](@attachment/02_04_2020/simple_model_rollout_error.jpg)
* Geometrically exact results
![simple model success](@attachment/02_04_2020/exact_model_success.jpg)
![simple model error](@attachment/02_04_2020/exact_model_rollout_error.jpg)
* Clear that transfer learning greatly improves error results for 2 tube case. Need to investigate 3 and 4 tube scenarios

### Ideas
* Save some trajectories during training and plot in matlab
* Videos of following a trajectory in matlab
* IPCAI will need a video
* Could include coefficents of output of NN in Kuntz paper as state for DRL and directly do control
* Similarly with coefficents can do model-based RL
* How does tube parameters effect exploration. Larger workspaces will need larger action steps

### Lookahead
* IPCAI presentation outline
* Come up with some video ideas for presentation
* 3 tube transfer learning experiment
* State noise to make policy more robust
* Lots of unneeded prints and output files in training, need to fix these
