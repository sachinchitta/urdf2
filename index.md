---
layout: default
title: URDF2
---
### Introduction
The URDF (Universal Robot Description Format) has not been updated in quite a while. Although it has served the ROS community admirably, it has several notable shortcomings. In this effort, we will try to modify the URDF specification to catch up with the needs of the ROS community. There are two parts to the URDF: (a) The XML format itself, (b) The URDF parser.

This document currently addresses only the URDF format itself. 

### Shortcomings
* Multiple robots - The URDF currently supports a single _robot_ tag. Putting multiple robots together into a URDF can only be done by combining them (manually or through the use of Xacro) into a single robot element. 
* Closed loop chains - The URDF is not as _Universal_ as the name implies, e.g. it does not support closed loop chains. 
* Groups - Software like MoveIt! uses groups of joints as a fundamental element. Groups are used to specify configuration for planning and could also be used for elements like ROS-Control.
* Sensors - URDF has no support for sensors. 
* Zero State/Default State - The URDF defaults the initial state of a robot to zero joint values. This might be an issue for certain robots since the zero joint value might not be reachable. 
* Saved states - The URDF does not support the storing of named states for the robot(s), e.g. HOME or READY.

### Alternatives

#### SDF Format 
The SDF or Scene Definition Format was defined for the Gazebo simulator and has become fairly well standardized for Gazebo. It is a viable alternative to the URDF although it comes with several pros and cons:

##### Pros 
1. It exists and has been used and developed for a while. 
1. It supports multiple robots (in a world).
1. It includes support for several types of sensors.
1. It includes support for storing state of the robots and the world.
1. It supports closed loop chains.
1. It includes a C++ API.
1. It is now fairly independent of Gazebo. 
1. It uses XML.

##### Cons
1. The SDF has several simulation specific parameters.
1. It does not support groups. 
1. The SDF is designed to support simulation _worlds_. The URDF was designed to support a single robot. 
1. The SDF has no support for specifying hardware parameters. 
1. The SDF cannot be exported from any CAD software today (AFAIK). 

#### Collada
Collada is a new format that has started to become more accepted. It is widely used in ROS for certain robots as well and is well-supported in OpenRAVE too. It has its own pros and cons. 

##### Pros
1. It exists and has been used and developed for a while. 
1. Several CAD software packages can export COLLADA files. 

##### Cons
1. It is not very _human readable_. 
1. There is no support for actuators/transmission specifications?

## URDF 2.0

It is clear that the URDF needs an update. Here's a minimal set of new features that have to get into the URDF. 

1. Multiple robots - The URDF currently supports a single _robot_ tag. We compose multiple robots into a single robot description by using Xacro and manually adding joints between robots, i.e. we treat a collection of robots as a single robot. MoveIt! uses groups to then allow users to address each part of the robot as a single entity. It is not clear that the URDF needs to allow multiple robot tags. However, there is a need for being able to parse some _world description_ and know all the robots that exist in the world. 
1. Closed loop chains - The URDF must be able to support the specification of closed loops, e.g. four-bar linkages or delta robots. 
1. Groups - URDF should natively support the specification of groups (user-defined sets of joints). This will significantly reduce configuration effort for upstream elements, e.g. MoveIt! or ROS-Control and also ensure consistency, e.g. there are too many places where lists of joints are currently specified in config files. 
1. Sensors - URDF must include support for the most commonly used sensors. 
1. Zero State/Default State/Saved state - The URDF should include support for specifying the _initial_ or _default_ state of a robot.  

## Suggested solution

The suggested solution is to borrow heavily from the following sources to create a URDF format coupled with a new __WORLD__ format to specify both the robot and the state of the world (including the robots in it). 

#### World Specification
* Add a new world format (separate from URDF)
* Borrow from: (a) [SDF World](http://sdformat.org/spec?ver=1.5&elem=world) (b) [MoveIt! Planning Scene](https://github.com/ros-planning/moveit_msgs/blob/indigo-devel/msg/PlanningScene.msg)
* Allow multiple robot tags in a world
* Note that the core URDF robot tag itself does not need to change
* What will potentially change is the way robot description is specified and read by nodes through the URDF parser.

#### Closed loop chains
* Allow closed loop chains - this can be achieved by allowing two different joints to have the same child link but different parents
* SDF supports something similar (see the PR2 gripper closed loop [here](https://bitbucket.org/osrf/gazebo_models/src/68989fe22ddc430909763eace37d45a4a4936e25/pr2_gripper/model.sdf?at=default).
* It does not seem like this will require a syntax change. 

#### Groups
* Add native ability to specify and designate groups in URDF
* Borrow directly from MoveIt! groups specification [SRDF](https://github.com/ros-planning/moveit_pr2/blob/indigo-devel/pr2_moveit_config/config/pr2.srdf)
* Make sure group names are namespaced inside robot name, e.g. robot1/right_arm to allow multiple robots to exist together in a world.

#### Sensors
* Add ability to specify most common types of sensors in URDF
* Borrow directly from [SDF](http://sdformat.org/spec?ver=1.5&elem=sensor)

#### Robot State
* Allow robot state to be specified in URDF
* Borrow directly from [MoveIt! SRDF](https://github.com/ros-planning/moveit_pr2/blob/indigo-devel/pr2_moveit_config/config/pr2.srdf)
