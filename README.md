# Robot Localization Project

[Brooke Moss](https://github.com/EarlJr53) and [Isha Goyal](https://github.com/Isha-Goyal)
[Olin College](https://www.olin.edu) - A Computational Introduction to Robotics - Fall 2023

<!-- ! Add GIF here -->

## Introduction (Isha)

<!-- 
1. What was the goal of your project? 
-->

A challenge with tracking a robot’s position is the unreliability of odometry data. Thus, our challenge was to use data from the robot’s laser scanner in conjunction with its odometry data to determine where it was in a known map. Our approach was based on an algorithm called a particle filter, which probabilistically narrows down the robot’s position.

## Approach (Isha)

<!-- 
2. How did you solve the problem? (Note: this doesn’t have to be super-detailed, you should try to explain what you did at a high-level so that others in the class could reasonably understand what you did). 
-->

<!-- High level, then implementation? -->

## Process (Brooke)

<!-- 
3. Describe a design decision you had to make when working on your project and what you ultimately did (and why)? These design decisions could be particular choices for how you implemented some part of an algorithm or perhaps a decision regarding which of two external packages to use in your project.
4. What if any challenges did you face along the way? 
-->

<!-- Design decisions (resampling + adding noise? choosing the heaviest particle, not the avg?); Challenges -->

One of the big design decisions in our project was the mechanism by which we update each particle with the odometry movements of the robot. In the `update_particles_with_odom()` method, we were given computations for the delta, or change in position and orientation, of the robot in the `odom` frame. The purpose of this method is to apply these relative movements from the robot to each particle. For instance, if the robot moves forward 4cm and 3cm to the right, each particle needs to do the same.

However, the difficulty in this problem comes from the various reference frames. The position and orientation (and therefore delta) of the robot is in the odometry frame, so it is relative to the robot's starting point. However, the movements that we care about are relative to the *current* heading and location of the robot, not its starting position. Essentially, if the robot moves in the direction its facing, it is moving forward, no matter which direction that happens to be in the `odom` frame. Similarly, we needed to be able to translate that forward motion of the robot into forward motion of each particle (relative to the heading of each particle).

<!-- ! Insert diagram here? -->

One way we could have solved this problem was using matrices, but we decided to tackle it from a more trigonometric angle. We started by finding the hypotenuse of the *x* and *y* delta values we were given, effectively the overall displacement from the previous robot position to the current position.

Once we knew how far to move each particle, we needed to determine in which direction to move them. The arctangent of the *xy* delta we were given is useful to determine the direction of motion in the `odom` frame. The main issue here is that the output of `atan` is identical in quadrants diagonal from each other. Using some logic statements and the signs of the original *xy* deltas, we can normalize this angle to ensure the movement direction is properly in the *(-pi, pi)* range of values. Before adding these checks and normalization, we had issues with the particles moving essentially the absolute value of where they should have been moving (a.k.a. only in the positive direction of a given axis).

Once we know the movement angle in the `odom` frame, we calculate the angle difference between the orientation of the robot and the direction of motion. This puts the motion into the reference frame of the current robot pose. In turn, we can apply this resulting movement vector onto the pose of each particle, and then using the particle's position and orientation in the `map` frame, we can move the particle to mirror the robot's movements.

This was one of the biggest challenges we faced during the project, although we also encountered some challenges with *NaN* and *0* values in the `update_particles_with_laser()` method, where if a scan didn't detect an obstacle, or had bugged scan data, we would encounter divide by zero errors. We fixed these issues by further cleaning the data and adding some `try/except` statements.

## Looking Forward (Brooke)

<!-- 
5. What would you do to improve your project if you had more time?
6. Did you learn any interesting lessons for future robotic programming projects? These could relate to working on robotics projects in teams, working on more open-ended (and longer term) problems, or any other relevant topic. 
-->

<!-- Faster (used lots of loops); Robot kidnapping? -->

If we had more time to work on this project, we would want to focus on making our code faster and more efficient. Our particle filter implementation uses many loops, which can be slower than, for example, operating directly on lists of items. In our testing, we maintained the initial number of 300 particles. Although we haven't tested our current algorithm with a greater number of particles, we expect one major advantage of speeding up our algorithm would be an increased particle count allowance. This could allow our localization to be more accurate, and allow the robot to find its location even sooner upon startup.

<!-- Lessons (Isha & Brooke) -->

One major takeaway from this project is the power of debugging when pair-programming. There were a number of times when we were working together on implementating a function and encountered confusing behaviour. By drawing out the flow of the function on a whiteboard and then discussing how the code matches or doesn't match that flow, we could narrow down locations to add print statements and find the missing logic in our code.
