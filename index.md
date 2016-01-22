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
* Motors - URDF only handles motors in the context of transmission interfaces.
* Controllers - URDF does not support motor controllers.
* Saved states - The URDF does not support the storing of named states for the robot(s), e.g. HOME or READY.
* Vacuum joints - The URDF has no support for vacuum joints - i.e. binary joints that are essentially On/Off.  

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

#### SMURF & SMURF scenes

The Supplementable, Mostly Universal Robot Format (SMURF) was designed as an extension of URDF and alternative to SDF and is described on the [github page of its parser](https://github.com/rock-simulation/smurf_parser). The basic idea behind its design was to keep URDF to stay compatible with current software tools and extend it in a human-readable and modular fashion, thus the use of YAML and distributed files.

SMURF scenes or SMURFS were designed to allow arranging multiple SMURF *entities* in a world, similar to SDF (see the [file format page](http://rock-simulation.github.io/mars//d7/dc2/smurfs.html) of the MARS simulation software, a simulator similar to Gazebo). The same modular principles apply as with SMURF itself: A robot is defined **once** in its SMURF definition, SMURF scenes merely describe how SMURF entities are placed, avoiding redundant definitions.



## URDF 2.0

It is clear that the URDF needs an update. Here's a minimal set of new features that have to get into the URDF. 

1. Multiple robots - The URDF currently supports a single _robot_ tag. We compose multiple robots into a single robot description by using Xacro and manually adding joints between robots, i.e. we treat a collection of robots as a single robot. MoveIt! uses groups to then allow users to address each part of the robot as a single entity. It is not clear that the URDF needs to allow multiple robot tags. However, there is a need for being able to parse some _world description_ and know all the robots that exist in the world. 
1. Closed loop chains - The URDF must be able to support the specification of closed loops, e.g. four-bar linkages or delta robots. 
1. Groups - URDF should natively support the specification of groups (user-defined sets of joints). This will significantly reduce configuration effort for upstream elements, e.g. MoveIt! or ROS-Control and also ensure consistency, e.g. there are too many places where lists of joints are currently specified in config files. 
1. Sensors - URDF must include support for the most commonly used sensors. 
1. Zero State/Default State/Saved state - The URDF should include support for specifying the _initial_ or _default_ state of a robot.  
1. Versioning - Support for versions.

In the following, the individual points will be discussed and solutions will be suggested respectively.

## Modular structure

The [SMURF](https://github.com/rock-simulation/smurf_parser) format uses a modular structure, allowing to hierarchically combine definitions of sensors, motors etc. for a model. This facilitates re-use of definitions. For instance, a specific laser scanner would only have to be specified once and could then be used in all robot models using that hardware; the model could even be provided by the manufacturer. Definitions of robot components can be stored in repositories (possibly also provided by manufacturers) and simply checked out when needed for a model, making handling and updating of models a lot more efficient. Having such centralized *model repositories* would also simplify resource sharing between ROS, Gazebo and other tools.

In case a specific model would have to change a component's properties from its default values found in the git, a project-specific fork or branch could be created, or properties could be overwritten locally without having to change the linked file, as it is currently already possible in SMURF.

### World Specification

URDF only describes one robot, but no environment or other robots and objects. There is a fundamental functional difference here: URDF is supposed to describe the "blueprint" of a robot and thus contains no states of joints, motors etc., whereas a world format will need to describe specific states of all entities in it, including those of robots.

To avoid both redundancy and behemoths of files, it makes sense to link to individual robot definitions from within the world definition. This is the case in the SMURF scene format developed for the [MARS simulation](https://github.com/rock-simulation/mars/). Similar to SMURF itself, a list of 'entities' is defined, each linking to a *file* specifying where the actual definition of the respective entity is found. This allows for instance to have separate robots or other environment objects in separate folders under (git) version control and combine them into a scene without having to update the scene when the robot definitions get updated.

However, the *file* tag in SMURFS is simply resolved by the parser, so that all contents of the specified file are added to the entity definition at parsing time, thus making it perfectly valid to not link to a file and simply directly define an entity in the scene when needed.

Every entity has a *type*, in the case of a robot defined for MARS, this type would be SMURF. In MARS, this makes it possible to create plugins for the simulation providing a factory for specified types and thus making the SMURF scene format easily extensible.

Additionally, assigning a *name* to each entity would allow to simply link the same file and thus the same robot definition multiple times in the same (world) context, providing  namespacing of all components defined for each robot, such as joints, motors or groups.

Moreover, it would easily be possible to attach one entity to another entity via a specified joint, enabling assembling a complex robot from simpler parts such as legs and arms or switching tools such as grippers on a robot. Currently, SMURFS includes only a simpler variety of this, allowing to *anchor* a robot to the world with a fixed joint or let it move freely. Since URDF joints already include a *fixed* and *floating* type, this could easily be replaced and extended.

For all the nice structure and flexibility SMURF scenes provides, they currently do not specify anything beyond entity lists. Yet worlds should contain other specifications, ranging from physical properties such as gravity vectors and friction parameters, over purely graphical elements such as background sky boxes or HUD elements. Such properties are already well-defined in the [SDF World](http://sdformat.org/spec?ver=1.5&elem=world) format and should not need to be reinvented. The same goes for the planning-relevant data defined in [MoveIt! Planning Scene](https://github.com/ros-planning/moveit_msgs/blob/indigo-devel/msg/PlanningScene.msg).

One issue that has to be decided is the use of XML vs. YAML: SMURF uses YAML, while SDF uses XML. While XML has been around for much longer and there are extensive sets of tools available for it, YAML is much more human-readable and slimmer. Due to similarities in syntax, YAML is also very easy to work with using Python dictionaries. Since Python is used extensively in ROS, this would simplify a lot of use cases.

#### Proposed Solution
* Add a new modular world format based on [SMURF scenes](http://rock-simulation.github.io/mars//d7/dc2/smurfs.html), [SDF World](http://sdformat.org/spec?ver=1.5&elem=world) and [MoveIt! Planning Scene](https://github.com/ros-planning/moveit_msgs/blob/indigo-devel/msg/PlanningScene.msg)
* By keeping the world format modular, linking files, and specifying entity types, URDF 2.0 could be introduced independent of the world format, keeping backwards-compatibility with URDF 1.0 as a separate entity type
* A new parser would be needed for the world format, passing the paths for URDF definitions to either the URDF 1.0 or 2.0 parsers (or other type parsers, for that matter). This will probably result in necessary changes on the URDF parser and the way URDF models are distributed in ROS.

### Closed loop chains
* Allow closed loop chains - this can be achieved by allowing two different joints to have the same child link but different parents
* SDF supports something similar (see the PR2 gripper closed loop [here](https://bitbucket.org/osrf/gazebo_models/src/68989fe22ddc430909763eace37d45a4a4936e25/pr2_gripper/model.sdf?at=default)).
* It does not seem like this will require a syntax change.

### Groups
* Add native ability to specify and designate groups in URDF 2.0
* Borrow directly from MoveIt! groups specification (SRDF) [SRDF](https://github.com/ros-planning/moveit_pr2/blob/indigo-devel/pr2_moveit_config/config/pr2.srdf)
* Group names in a robot definition would not have to be namespaced (robot1/right_arm) to allow multiple robots to exist together in a world if a modular solution with lists of entities as described above is realized. This is because every entity would not only specify a type and file, but also a name, thus allowing to load multiple instances of the same robot definition in a given context, differentiating entities by names.

### Sensors
* Add ability to specify most common types of sensors in URDF
* Borrow directly from [SDF](http://sdformat.org/spec?ver=1.5&elem=sensor)

### Robot State
* Allow robot state to be specified in URDF
* Allow combinations of overall body states and group states
* Borrow directly from [MoveIt! SRDF](https://github.com/ros-planning/moveit_pr2/blob/indigo-devel/pr2_moveit_config/config/pr2.srdf)
