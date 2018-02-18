# **Path Planning Project**

### Using a path planner and state machine to maximize driving comfort and optimize path

---

**Path Planning Project**

The goals / steps of this project are the following:

* Complete starter code to successfully drive vehicle in simulator satisfying the following criteria
 * Drives farther than 4.32 miles without incident
 * Obeys that 50 mph speed limit
 * Does not exceed 10 m/s^2 of acceleration or 10 m/s^3 of jerk
 * Does not collide with any other cars
 * Does not spend more than 3 seconds outside of a lane
 * Can safely change lanes when lanes are clear

[//]: # (Image References)

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

My project includes the following files:
* [main.cpp](../src/main.cpp) main program that runs the IO server, makes lane change decisions, and creates trajectory
* [spline.h](../src/spline.h) header file for spline interpolation library
* [recording.mp4](./recording.mp4) video of 5.72 miles travelling around the course without incident

### Discussion

#### 1. Implementation

In this project there are actually two different algorithms implemented.  The first would be behavior control, what would typically be implemented in a Finite State Machine, and seoncd would be trajectory planning, which can be implemented any numberas as way such as with an MPC controller or spline interpolation.

The simplicity of this project did not require a full implementation of a FSM with explicit state definitions, transition rules, etc., however it is still there.  Instead of FSM, the state of the vehicle and all the decision making criteria is done with a few booleans.  The model that is implemented now makes the following decisions:
* Maintain 49.5 mph speed limit OR match speed to vehicle ahead
  * [Set new speed limit to leading vehicle speed](../src/main.cpp#L279)
  * [Reset vehicle speed to 49.5 mph if no obstructions](../src/main.cpp#L299)
* Request lane change if leading vehicle speed is slower than 45 mph
  * [Perform comparison of leading vehicle speed](../src/main.cpp#L281)
* If lane change is requested, evaluate which lanes are clear
  * [Perform comparison of leading vehicle speed](../src/main.cpp#L317)
* Initiate lane change when a viable lane is available
  * [Check if adjacent lane is open](../src/main.cpp#L344)
* Acelerate and decelerate at permissible values with a ramp function
  * [Ramp speed to maintain acceleration and jerk, with two different decleration profiles](../src/main.cpp#L301)

Now, regarding the path planning trajectory, the specific request was to minimize jerk.  Jerk can be characterized by discontinuities in accereration, which are inherent from discontinuities velocity.  So by creating a smooth path profile, we guarantee no sudden movements.  Also, during a lane change, if we ensure our departure path from one lane is tangent to the lane we're exiting, and our entrance path to the new lane is tangent to the lane we are changing to, we can guarantee jerk is minimized.

In the path planning, we must handle the following two issues:
* Create a smooth path that intersects all the waypoints without creating discontinuities
* Create a smooth lane change path that exits and enters the new lane tangentially

This can be accomplished with splines.  Splines are piecewise functions that are made to guarantee contiunity, and are a perfect fit for our application to prevent jerk.  In the project, we have used the tiny spline library that is in the self contained header [spline.h](../src/spline.h).  By feeding waypoints into a spline then generating smaller points around it for the actual json message, all vehicle movements are smooth and continuous.  However, implementation of the spline solution has one caveat.  Because we evaluate the spline repeatedly as a new time t = 0, we must keep track of our previous path points to guarantee continuity from the previous motion.  See several noteworth points in code below:
* [Using previous path points as start for spline fitting to ensure continuity from previous motion](../src/main.cpp#L406)
* [Spline creation](../src/main.cpp#L445) and [fitting to points](../src/main.cpp#L448)
* [Implementation of the reference velocity by calculation of distance at that speed](../src/main.cpp#L473)

#### 2. Results

The final results were very desirable.  The vehicle displays the ability to handle all possible scenarios created by the simulator without incident.  In particular, the following behaviors can be observed:
* Lane keeping below speed limit
* Distance keeping with leading vehicle by matching speed
* Lane changing decisions, including choosing left or right lange change over another based on lane availablility
* Decision making to change lane bases on how slow the leading vehicle may be travelling
* Two deceleration values for braking, depending on proximity of car ahead

Results:

[recording.mp4](./recording.mp4)

