# **MPC Control**

![image]("https://github.com/jcmaeng/CarND_Term2_P5_ModelPredictiveControl/blob/master/image/sim_result.png")
## Goal
- Move the car on the road with keeping the center of the track using MPC

## Description

### 1. The Model
In this project, I use the `Kinematic model` which is simplified version of dynamic models by ignoring  tire forces, gravity, and mass. This simplification reduces the accuracy of the models, but it also makes them more tractable. At low and moderate speeds, kinematic models often approximate the actual vehicle dynamics.

* State information (provied by the simulator)
 - x: x position of the vehicle
 - y: y position of the vehicle
 - psi: orientation of the vehicle
 - v: velocity of the vehicle

* Error state (calculated in the program)
 - cte: Cross Track Error of the vehicle
 - epsi: Orientation error of the vehicle

* Actuator constraints
 - delta: steering angle
 - a: throttle value

* State update equations
 - x(t+1) = x(t) + v(t) * cos(psi(t)) * dt
 - y(t+1) = y(t) + v(t) * sin(psi(t)) * dt
 - psi(t+1) = psi(t) + v(t) / Lf * delta * dt
 - v(t+1) = v(t) + a(t) * dt

* Error update equations
 - cte(t+1) = f(x(t)) - y(t) + (v(t) * sin(epsi(t)) * dt)
 - epsi(t+1) = psi(t) - psi_des(t) + (v(t) / Lf * delta(t) * dt)

### 2. Timestep Length and Elapsed Duration
The prediction horizon(T) is the duration over which future predictions are made. T is the product of two other Variables, N and dt. N is the number of timesteps in the horizon. dt is how much time elapsed between actuations. 
A good approach to setting N, dt, and T is to first determine a reasonable range for T and then tune dt and N appropriately, keeping the effect of each in mind.

For `dt`, I tried values within 0.05(50ms) <= dt <= 0.20(200ms). However, if dt = 100ms, it may cause some unstability. So it is better to select `dt` value slightly larger than 100ms (latency).

For `N`, I tested values within 10 <= N <= 20. During the test, `N * dt` > 2.3s are not good. Then, after choosing `dt`, I select `N` as small as possible.

This table shows my trial and error to tune N and dt values.

|N  | dt  | N * dt| Max speed|  Comment             |
|:--:|:---:|:---:|:---------:|----------------------|
|10 | 0.05| 0.5 |   Error   | bumpy, run out of track |
|10 | 0.10| 1 |   81.29   | good   |
|20 | 0.10| 2 |   89.08   | a little bit unstable during left turn |
|20 | 0.20| 4 |   80.12   | a little bit unstable with frequent breaking in curves|
|20 | 0.15| 3 |   84.45   | a little bit unstable with some wrong predictions |
|20 | 0.16| 3.2 |   82.74   | a little bit unstable with some wrong predictions |
|20 | 0.18| 3.6 |   80.04   | a little bit unstable with frequent breaking in curves|
|18 | 0.18| 3.24 |   78.68   | a little bit unstable during some curves (wrong predictions)|
|18 | 0.15| 2.70 |   83.76    | a little bit unstable during some curves (wrong predictions)|
| 15 | 0.15 | 2.25 |   81.36    | good |
| **13** | **0.13** | 1.69 |   83.26    | good|

Below lines are the weight values which I used in my code.
```
// weight for variables
const int w_cte = 5000;
const int w_epsi = 4000;
const int w_v = 1;
const int w_delta = 20;
const int w_a = 20;
const int w_delta_diff = 1000;
const int w_a_diff = 5;
```

### 3. Polynomial Fitting and MPC Preprocessing
To predict the next points of the vehicle trajectory, a third-degree polynomial is being used in this model. The predicted waypoints are being transformed from global coordinates to respective vehicle coordinates.

### 4. Model Predictive Control with Latency
To apply latency in this project, I use the equations as follows.
 - x_delay = x + v * latency
 - psi_delay = psi + v * steer_value / Lf * latency
 - v_delay = v + throttle_value * latency


---
# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).
