# CarND-Controls-MPC
##Self-Driving Car Engineer Nanodegree Program

[Video Result](https://youtu.be/Q78VaBzg0xs)
---
## Discussion:
I found this project to be one of the more difficult projects to date. I found myself relying heavily on the quiz from the lectures, as well as scouring the slack channels and forums to hunt down solutions to my various problems. I used a lot of the code I worked on from the quiz in this project, and it was really quite helpful. Other students were also very helpful and shared some insightful answers. The model follows the same pattern as used in the quiz, and makes use of the update equations described in the lectures:

x_[t] = x[t-1] + v[t-1] * cos(psi[t-1]) * dt
y_[t] = y[t-1] + v[t-1] * sin(psi[t-1]) * dt
psi_[t] = psi[t-1] + v[t-1] / Lf * delta[t-1] * dt
v_[t] = v[t-1] + a[t-1] * dt
cte[t] = f(x[t-1]) - y[t-1] + v[t-1] * sin(epsi[t-1]) * dt
epsi[t] = psi[t] - psides[t-1] + v[t-1] * delta[t-1] / Lf * dt

In addition to the state values that are being updated above, there were two actuator values being optimized by the code: delta (steering angle) and throttle (acceleration).
I started out using the same N and dt values used in the quiz, and was actually able to get things going when I turned off latency, but I felt like I didn't want to predict that far out, and also that my timestep should not be smaller than the latency, so I reduced N to 20, and increased dt to 0.1, which gave me a prediction horizon of 2. (Within the suggested range)

The waypoints were coming in with global map coordinates, and these needed to be translated into coordinates relative to the car. The operation is similar to the one used in the particle filter project when converting from local to global coordinates, but in reverse. With the car being the reference frame, I could evaluate my polynomial using this neutral point.

At first, I turned off the latency as I hadn't implemented any solution to handle it, and viewed how the MPC performed. It started out promising enough, but quickly started swerving uncontrollably until it was off the track and unable to correct itself. I knew right away that I would need to drastically increase the penalty for adjusting steering angles too quickly. I was surprised at how high I needed to penalize quick changes in steering angle, but eventually the car started making it around the track. There were occasionaly oddities where the car would still jerk out of its smooth path, so I tried adding a penalty to frequent actuator adjustments and that seemed to help - but the issue was transient in nature, so I'm not sure if it's actually a good solution. I used the cost penalty for speed described in the lecture to prevent the car from stopping: add a reference speed. I felt this was a better approach than a euclidean distance one as I could foresee potential issues with euclidean distance if your endpoint was close by, but you had to travel a large distance in order to get there. (U-turn type scenarios).

Then I turned the latency back on and the car was able to make it around the track so long as I kept the speed low - but I knew this was wholly unsatisfactory and started to think about ways to handle it. This was an issue that took me to the forums/slack and eventually I found a solution that made perfect sense: predict where the car would be after the latency and optimize from there.  

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
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
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.


