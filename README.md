# rmf-test
This is a demo project using open-RMF (Robot Middleware Framework).

This project has been tested on Ubuntu 22.04 with ROS2 Humble.

<b>Things implemented so far:</b>
* simple warehouse layout using ```traffic-editor```;
* generating Gazebo world using ```rmf_building_map_tools```;
* generating traffic navigation path graphs using ```rmf_building_map_tools```.

### Setup

Setup ROS2 (humble used here): https://docs.ros.org/en/humble/Installation.html

Setup open-rmf: https://github.com/open-rmf/rmf

#### Source ROS2
```
source /opt/ros/humble/setup.bash
```

#### Install all necessary dependencies:
```
rosdep install -i --from-path src --rosdistro humble -y
```

#### Build package
```
colcon build --packages-select rmf_test
```

#### Launch simulation
```
source install/setup.bash && ros2 launch rmf_test start_world.launch.xml
```

### Edit layout
To edit the layout, add more floors, alter traffic lanes and make different changes, use the ```traffic-editor```, which is a part of open-RMF. The ```warehouse_traf.building.yaml``` file must be loaded into ```traffic-editor``` with
```
traffic-editor src/rmf_test/rmf_config/warehouse_traf.building.yaml
```
or simply open ```traffic-editor``` and load file using GUI:
```
traffic-editor
```
<b>After performing and saving changes in ```traffic-editor```</b> rebuild the package, and a new Gazebo world will automatically be generated.


### Useful resources:
* https://github.com/open-rmf
* https://github.com/open-rmf/rmf_traffic_editor
* https://app.theconstructsim.com/open-classes/9cf8dc7a-a228-4127-b44a-e2ca0d34ec92
* https://osrf.github.io/ros2multirobotbook/intro.html
* https://github.com/open-rmf/rmf_demos