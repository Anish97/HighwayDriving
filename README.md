# Highway Driving

   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).  

To run the simulator on Mac/Linux, first make the binary file executable with the following command:
```shell
sudo chmod u+x {simulator_file_name}
```

### Goals
In this project the goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. The car's localization and sensor fusion data are provided and there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

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
# Approach

The main Path Planning algorithm corresponds to the file https://github.com/Anish97/HighwayDriving/blob/master/src/main.cpp

## Prediction

For avoiding collision, we need to predict the position of the traffic vehicles at the instant the ego vehicle completes the trajectory already planned.
So, we calculate the speed of each traffic vehicle using the data from Perception module and multiply with the time duration of the planned trajectory of the ego vehicle, and simply add it to the position of traffic vehicle to get its future position.

This method can be made more accurate by measuring the acceleration of each vehicle but we keep it simple in this project.

## Behaviour Planning

There are 5 states designed for the Path Planning problem.
1) Keep Lane
2) Left Intent
3) Left Lane Change
4) Right Intent
5) Right Lane Change

### Keep Lane
The ego vehicle would try to attain the maximum allowable speed if the vehicle infront of it in the same lane is sufficiently distant from it. Otherwise it decelerates, eventually travelling at the same speed as the vehicle in front and switching to the Left Intent state or Right Intent state. It tries to switch to Left Intent first, since it is assumed that in Left hand drive, overtaking from left is preferred. The target lane is the lane the ego vehicle is currently in.
The sufficient distance is calculated as:

![alt text](https://github.com/Anish97/HighwayDriving/blob/master/Screenshot%202021-09-16%20at%201.00.50.png)

assuming that maximum braking acceleration would be applied.

### Left Intent
The algorithm checks if there are no other vehicles within 30 metres in the lanes left to the ego vehicle. If yes, the state remains in the Left Intent state. If not, the state changes to Left Lane Chnage state. The target lane is the lane left to the ego vehicle.

### Left Lane Change
This is when there are no other vehicles within 30 metres in the lanes left to the ego vehicle. The ego vehicle moves to the lane immediately left to it in a smooth trajectory. The target lane is the lane left to the ego vehicle.

### Right Intent
The algorithm checks if there are no other vehicles within 30 metres in the lanes right to the ego vehicle. If yes, the state remains in the Right Intent state. If not, the state changes to Right Lane Chnage state. The target lane is the lane right to the ego vehicle.

### Right Lane Change
This is when there are no other vehicles within 30 metres in the lanes right to the ego vehicle. The ego vehicle moves to the lane immediately right to it in a smooth trajectory. The target lane is the lane right to the ego vehicle.

* Note that in Left Intent and Right Intent states, all lanes left/right to the ego vehicle are checked for nearby vehicles (instead of only the immediate left/right lanes) since there is a possibility of a vehicle two lanes away from the ego vehicle moving to the same lane as the ego vehicle as the same time.

## Trajectory Planning

After we obtain the target lane from the Behaviour Planning step, the near future trajectory of the ego vehicle is planned.
A First-In-First-Out vector of trajectory points is maintained and as the vehicle traverses over these points, new points are added from a spline curve.
The spline is calculate by intrapolating over the last two points (most recently added) of the trajectory points and one point far ahead (30metres) in the target lane. This ensures that the trajetory is smooth and eventually the ego vehicle ends up in the target lane. To simplify calculation, the spline curve is calculated in the ego vehicle's reference frame by transforming the required points by translation and rotation.
