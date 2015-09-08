---
layout: default
title: URDF2
---
### Introduction
The URDF (Universal Robot Description Format) has not been updated in quite a while. Although it has served the ROS community admirably, it has several notable shortcomings. In this effort, we will try to modify the URDF specification to catch up with the needs of the ROS community

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
* Groups - URDF should natively support the specification of groups (user-defined sets of joints). This will significantly reduce configuration effort for upstream elements, e.g. MoveIt! or ROS-Control and also ensure consistency, e.g. there are too many places where lists of joints are currently specified in config files. 
* Sensors - URDF must include support for the most commonly used sensors. 
* Zero State/Default State/Saved state - The URDF should include support for specifying the _initial_ or _default_ state of a robot.  

### Suggested solution

The suggested solution is to borrow heavily from the following sources to create a URDF format coupled with a new __WORLD__ format to specify both the robot and the state of the world (including the robots in it). 

1. SDF
1. MoveIt! config
1. ROS-Control config

#### World Specification
__Solution:__ Add a new world format (separate from URDF)
__Sources:__ (a) SDF (b) MoveIt! Planning Scene
__Notes:__ Allow multiple robot tags

#### Closed loop chains
__Solution:__ Allow closed loop chains
__Sources:__ SDF

#### Groups
__Solution:__ Add native ability to specify and designate groups in URDF
__Sources:__ MoveIt! groups specification (SRDF)
__Notes:__ Make sure group names are namespaces inside robot name, e.g. PRX/right_arm to allow multiple robots to exist together in a world

#### Sensors
__Solution:__ Add ability to specify most common types of sensors in URDF
__Sources:__ SDF

#### Robot State
__Solution:__ Allow robot state to be specified in URDF
__ Sources:__ MoveIt! SRDF


