# Start Guide

Before starting the Local SP module follow the [Install guide](./opil_local_sp_install.md).


In the following, two start options will be described, depending if you installed the source from the docker containers or from scratch.





## <a name="fromdockerlocal">Starting from Docker - Local SP</a>

The Local SP container can be started in two ways: without RAN and with RAN. If started without RAN then the simulator Stage is used for testing and visualizing what the Local SP does. This option is described first. Prepare [docker-compose.yml](./opil_local_sp_install.md#dockercomposelocal).
It is important to put the environment variable `SIMULATION=true`.
This starts IML map, **amcl** localization inside that map shown on topic `/robot_0/pose_channel`, and calculation of map updates shown on topic `/robot_0/newObstacles`. The robot in the Stage simulator can be moved by mouse dragging, and changed pose can be seen in topic `/robot_0/pose_channel` which shows the current pose and the covariance of the pose estimation. There is also an obstacle in the Stage simulator that can be used for test of showing the map updates, seen in topic `/robot_0/newObstacles`.

<a name="simmap">To change the map for the simulation</a> prepare the following files: `testmap.yaml`, `testmap.png`, `amcl_testmap.launch`, and `testmap.world`, and remove the comment mark # in front of corresponding lines under **volumes**.
These example files can be found in the folder `test/docker_compose_files/Local_SP_docker`.
<a name="prepmap">The map</a>
can be changed by preparing `testmap.yaml` and `testmap.png`. The second file can be any bitmap picture (pgm), grayscale is desired, but if not, it will be automatically converted to a grayscaled image. 
Here is an example of the CHEMI factory floorplan saved as png file:

![IML topology](./img/map.png)

`testmap.yaml` is a parameter file with **resolution** and gray thresholds for defining what is free and what is occupied, as in this example yaml file:
```
#testmap.yaml
image: map.png
resolution: 0.06
origin: [0.0, 0.0, 0.0]
negate: 0
occupied_thresh: 0.65
free_thresh: 0.196
```
where the image name is `map.png` to which your `testmap.png` is copied to on the docker side, **resolution** defines the size of the pixel of the png file in meters. To calculate the resolution, you need to know the width of the image in meters. Then, simply divide the width of the image with the number of pixels. Adjust the parameters **occupied_thresh** and **free_thresh** to different values, depending on which shade of grey should be considered as occupied and free, respectively. 

Parameters of **amcl** localization can be set in `amcl_testmap.launch`, and here is a default file:
### <a name="amcllaunch">amcl_testmap.launch</a>
```
<launch>
<node pkg="amcl" type="amcl" name="amcl" respawn="true">
<remap from="scan" to="base_scan" />
  <!-- Publish scans from best pose at a max of 10 Hz -->
  <param name="odom_model_type" value="omni"/>
  <param name="odom_alpha5" value="0.1"/>
  <param name="transform_tolerance" value="0.2" />
  <param name="gui_publish_rate" value="10.0"/>
  <param name="laser_max_beams" value="24"/>
  <param name="min_particles" value="500"/>
  <param name="max_particles" value="5000"/>
  <param name="kld_err" value="0.05"/>
  <param name="kld_z" value="0.99"/>
  <param name="odom_alpha1" value="0.2"/>
  <param name="odom_alpha2" value="0.2"/>
  <!-- translation std dev, m -->
  <param name="odom_alpha3" value="0.8"/>
  <param name="odom_alpha4" value="0.2"/>
  <param name="laser_z_hit" value="0.5"/>
  <param name="laser_z_short" value="0.05"/>
  <param name="laser_z_max" value="0.05"/>
  <param name="laser_z_rand" value="0.5"/>
  <param name="laser_sigma_hit" value="0.2"/>
  <param name="laser_lambda_short" value="0.1"/>
  <param name="laser_lambda_short" value="0.1"/>
  <param name="laser_model_type" value="likelihood_field"/>
  <!-- <param name="laser_model_type" value="beam"/> -->
  <param name="laser_likelihood_max_dist" value="2.0"/>
  <param name="update_min_d" value="0.2"/>
  <param name="update_min_a" value="0.05"/>
  <param name="odom_frame_id" value="odom"/>
  <param name="resample_interval" value="1"/>
  <param name="transform_tolerance" value="0.1"/>
  <param name="recovery_alpha_slow" value="0.0"/>
  <param name="recovery_alpha_fast" value="0.0"/>
  <param name="initial_pose_x" value="7.717"/> 
  <param name="initial_pose_y" value="32.608"/>  
  <param name="initial_pose_a" value="0"/>
</node>
</launch>
```
The most important parameters are the range topic, which for simulator is **base_scan**, and initial values for the pose of the robot: **initial_pose_x**, **initial_pose_y**, **initial_pose_a** (angle in radians). For all other parameters visit <http://wiki.ros.org/amcl>.

`testmap.world` is the world file for the Stage simulator, where example is given as follows:
```
define kinect ranger
(
  sensor
  (
    range [0.0 10]
    fov 360.0
    samples 720
  )

  # generic model properties
  color "black"
  size [ 0.060 0.150 0.030 ]
)

define robot position
(
  origin [ 0.000 0.000 0.000 0.000 ]
  localizaton "odom"
  odom_error [0.03 0.03 0.01 0.01 0.02 0.02] 
  size [ 0.600 0.400 0.400 ]  
  drive "omni"
    
  kinect(pose [ -0.100 0.000 -0.110 0.000 ])
)
#define an obstacle
define worker model
(
  size [0.500 0.500 1.850]
  gui_nose 0  
)

define floorplan model
(
  color "gray30"  
  boundary 0
  gui_nose 0
  gui_grid 0
  gui_outline 0
  gripper_return 0
  fiducial_return 0
  laser_return 1
)

resolution 0.05

# simulation timestep in milliseconds
interval_sim 100  

#stage view window
window
(
  size [ 935 412 ]
  center [ 18.080 20.285 ]
  rotate [ 0.000 0.000 ]
  # pixel per meter
  scale 13.110
)

# stage map
floorplan
(
  name "testmap"
  bitmap "map.png"
  size [ 111.720 49.320 1.000 ]
  pose [ 55.860 24.660 0.000 0.000 ] 
)

# throw in a robot
robot
(
  pose [ 7.717 32.608 0.000 0.000 ] 
  name "robot_0"
  color "blue"  
)

worker
(
	pose [ -2.538 1.380 0.000 180.000 ]
	color "green"
)
```
The most important parameters are **size** and **pose** in the **floorplan** section which need to be calculated in meters from `testmap.png` and **resolution** in `testmap.yaml`. For example, our `testmap.png` has 1862 x 822 pixels, and in `testmap.yaml` we see that resolution is 0.06. The first column in **size** is the width of the map along **x** axis in meters obtained by multiplication of resolution and width in pixels, i.e., 1862 x 0.06 = 111.72, while the second column in **size** is the height along **y** axis obtained by multiplication of resolution and height in pixels, i.e., 822 x 0.06 = 49.32. The third column is the height along **z** axis of the floorplan, i.e. 1 meter. **pose** of the **floorplan** defines the origin of the loaded map in Stage in the form of **x**, **y**, **z**, **theta** (in degrees) values. If values for **x** and **y** coordinates are half of the **floorplan**'s **size**, then the origin will be in the lower left corner of the map. Adjust the **pose** of the **robot** to be aligned to the initial pose in [amcl_testmap.launch](#amcllaunch), and of the **worker** as desired (treated as moving obstacle). The rest of the parameters can be the same as in this example. For more explanation about the **world** files visit <http://wiki.ros.org/stage>. 

There are also some prepared maps inside this docker which can be set by changing the `local_robot_sim.launch` file, which has the following structure:
### <a name="localrobotsimlaunch">local_robot_sim.launch</a>
```
<launch>
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/map.yaml" respawn="false" >
<param name="frame_id" value="/map" />
</node>
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/yaml/map.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
<include file="$(find lam_simulator)/launch/amcl_map.launch" />
     <!--- Run pubPoseWithCovariance node from sensing_and_perception package-->
     <!-- Put args="1" if you are testing the robot with the id number 1 -->
     <node name="publishPoseWithCovariance" pkg="sensing_and_perception" type="pubPoseWithCovariance" output="screen" args="0"/>	

     <!--- Run mapup node from mapupdates package-->
     <!-- Put args="1" if you are testing the robot with the id number 1 -->
    <node name="mapup" pkg="mapupdates" type="mapup" output="screen" args="0" >
        <param name="cell_size" type="double" value="0.1" />
        <param name="scan_topic" value="/base_scan" />
        <param name="laser_inverted" type="bool" value="0" />
    </node>

    <!-- Run FIROS -->
    <node name="firos" pkg="firos" type="core.py" />

<node name="rviz" pkg="rviz" type="rviz" args="-d $(find lam_simulator)/rviz_cfg/singlerobot.rviz" /> 

</launch>
```
In this launch files you can set various parameters: **cell_size** defines the fine gridmap for calculating new obstacles (presented as tiny red squares); robot ID as an argument of packages _sending_and_perception_ and _mapupdates_ for having more robots (0,1,2,etc.), and **scan_topic** for defining the topic for the range data.
There is also a parameter **laser_inverted** to indicate if the laser is mounted upside down on the robot (use value 1), or normally (use value 0). The simplest check if laser is setup correctly is to see in rviz if laser readings are aligned with the png map.  
Right now it is implemented that Central SP can handle three robots with ID's 0, 1 and 2. If you change the ID of the robot to 1 the topics `/robot_1/newObstacles` and `/robot_1/pose_channel` will be created and visible in OCB.

To use the changed local_robot_sim.launch uncomment the line in [docker-compose.yml](./opil_local_sp_install.md#dockercomposelocal) under **volumes**:
```
            - ./local_robot_sim.launch:/root/catkin_ws/src/localization_and_mapping/sensing_and_perception/local_robot_sim.launch:ro
```
Replace the lines 2-8 from [local_robot_sim.launch](#localrobotsimlaunch) with the following:

* CHEMI map
```
 <node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/CHEMIchangedres.yaml" respawn="false"> 
     <param name="frame_id" value="/map" /> 
</node>	    	  
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_CHEMI.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisimCHEMI.launch" />
```
* ICENT map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/Andamapa.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/andaomnidriveamcltest.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisimdemo.launch" />
```
* IML map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/IMLlab.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_IML.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisimdemo.launch" />
```
* MURAPLAST map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/floorplan_muraplast.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_muraplast.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisim.launch" />
```
The following figure presents the output after moving the green box in the Stage simulator for the IML map:
![Local SP](./img/localIML.png)

In the rviz window you can see the AGV's pose (red arrow) and local updates (red tiny squares). All new obstacles are processed as they are detected, and there is no clearance of obstacles since they are considered to be static. That is the reason why you can see a trail of the obstacle. In future developments static and dynamic obstacles will be treated differently.

Here is the examle of using the ICENT map. 
After moving the green box in the simulator the result can be obtained as in this figure:
![Local SP in ICENT lab](./img/localicent.png)

## <a name="fromdockerlocalran">How to start the Local SP and RAN docker containers together</a>

The Local SP container needs to be started with the environment variable `SIMULATION=false`, as is the case in [docker-compose.yml](./opil_local_sp_install.md#dockercomposelocalran).

IML map will be started and you can command the robot through rviz by pressing 2D Nav Goal button and clicking the point on the map.
To change the map prepare `testmap.yaml`, `testmap.png`, `amcl_testmap.yaml`, `testmap.world` as explained [previously](#simmap).

Like in previous sections, there are some prepared maps inside both docker containers which can be set by changing the `local_robot.launch` file for Local SP, and `robot_launcher.launch` for RAN. First prepare `local_robot.launch` as follows:
### <a name="localrobotlaunch">local_robot.launch</a>
```
<launch>
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/map.yaml" respawn="false" >
<param name="frame_id" value="/map" />
</node>
<include file="$(find lam_simulator)/launch/amcl_map.launch" />
     <!--- Run pubPoseWithCovariance node from sensing_and_perception package-->
     <!-- Put args="1" if you are testing the robot with the id number 1 -->
     <node name="publishPoseWithCovariance" pkg="sensing_and_perception" type="pubPoseWithCovariance" output="screen" args="0"/>	

     <!--- Run mapup node from mapupdates package-->
     <!-- Put args="1" if you are testing the robot with the id number 1 -->
    <node name="mapup" pkg="mapupdates" type="mapup" output="screen" args="0" >
        <param name="cell_size" type="double" value="0.1" />
        <param name="scan_topic" value="/base_scan" />
        <param name="laser_inverted" type="bool" value="0" />
    </node>

    <!-- Run FIROS -->
    <node name="firos" pkg="firos" type="core.py" />

<node name="rviz" pkg="rviz" type="rviz" args="-d $(find lam_simulator)/rviz_cfg/singlerobot.rviz" /> 

</launch>
```
To use the changed local_robot.launch uncomment the line in [docker-compose.yml](./opil_local_sp_install.md#dockercomposelocalran) under **volumes**:
These example files can be found in the folder `test/docker_compose_files/Local_SP_docker_with_RAN`.
```
            - ./local_robot.launch:/root/catkin_ws/src/localization_and_mapping/sensing_and_perception/local_robot.launch:ro
```
Replace the lines from 2-5 in [local_robot.launch](#localrobotlaunch) with the following:

* CHEMI map
```
 <node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/CHEMIchangedres.yaml" respawn="false"> 
     <param name="frame_id" value="/map" /> 
</node>	    	  
<include file="$(find lam_simulator)/launch/amcl_omnisimCHEMI.launch" />
```
* ICENT map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/Andamapa.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisimdemo.launch" />
```
* IML map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/IMLlab.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisimdemo.launch" />
```
* MURAPLAST map
```
<node name="map_server" pkg="map_server" type="map_server" args="$(find lam_simulator)/yaml/floorplan_muraplast.yaml" respawn="false" >
    <param name="frame_id" value="/map" />
</node>
<include file="$(find lam_simulator)/launch/amcl_omnisim.launch" />
```

Then, prepare `robot_launcher.launch` as follows:
### <a name="robotlauncherlaunch">robot_launcher.launch</a>
```
<launch>
    <!-- Run FIROS -->  
    <node name="firos" pkg="firos" type="core.py" />
    <!-- Run the Stage simulation -->
    <node name="stageros" pkg="stage_ros" type="stageros" args="$(find opil_v2)/map/map.world" respawn="false"> 
        <param name="base_watchdog_timeout" value="0.2" />
    </node>
 
        <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
            <remap from="map" to="/map" />                      
            <param name="controller_frequency" value="10.0" />
            <rosparam file="$(find opil_v2)/config/costmap_common_params.yaml" command="load" ns="global_costmap" />
            <rosparam file="$(find opil_v2)/config/costmap_common_params.yaml" command="load" ns="local_costmap" />
            <rosparam file="$(find opil_v2)/config/global_costmap_params.yaml" command="load" />
            <rosparam file="$(find opil_v2)/config/local_costmap_params.yaml" command="load" />
            <rosparam file="$(find opil_v2)/config/base_local_planner_params.yaml" command="load" />
        </node>  
         <!--- Run the Robot Agent node -->
         <node name="robot_agent" pkg="opil_v2" type="opil_v2_robot_agent_with_SandP" output="screen" args="0"/>
    
</launch>
```
To use the changed robot_launcher.launch uncomment the line in [docker-compose.yml](./opil_local_sp_install.md#dockercomposelocalran) under **volumes**:
```
            - ./robot_launcher.launch:/root/catkin_ws/src/opil_v2/robot_launcher_with_SandP.launch:ro
```
Replace the lines from 5-7 in [robot_launcher.launch](#robotlauncherlaunch) with the following:


* CHEMI map
```
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_CHEMI.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
```
* ICENT map
```
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/andaomnidriveamcltest.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
```
* IML map
```
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_IML.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
```
* MURAPLAST map
```
<node pkg="stage_ros" type="stageros" name="stageros" args="$(find lam_simulator)/world/floormap_muraplast.world" respawn="false" >
    <param name="base_watchdog_timeout" value="0.2"/>
</node>
```

The following example presents testing the ICENT map. The robot can be controlled through rviz by pressing 2D Nav Goal button and clicking the point on the map. The following figure describes the possible outcome, where arrow trails correspond to the past positions of the robot:
![Local SP and RAN in ICENT lab](./img/movingwithran.png)


## <a name="fromscratch">Starting from Scratch</a>

In this quick guide, everything will be started on the same computer where you installed ROS and this source code. For advanced configuration of starting at different computers check [Interfaces](./../../develop/SP/opil_sp_interfaces.md).

* First, start the Stage simulator with the loaded example map and one AGV inside the map (red) and one box as the unknown obstacle (green):

```
terminal 1: roslaunch lam_simulator AndaOmnidriveamcltestZagrebdemo.launch
```
The result should be the started rviz and the Stage simulator as presented here:
![SP module architecture](./img/localizationAndamap.png)

The green dots are simulated laser readings, and the red arrow is the localized AGV.

* Then, start the module for creating the topic for Pose with Covariance of the AGV:
```
terminal 2: roslaunch sensing_and_perception send_posewithcovariance.launch 
```
You can type 'rostopic echo' to see the message of pose with covariance:
```
rostopic echo /robot_0/pose_channel
```
You can start some other simulation example following [this guide.](./../../develop/SP/opil_api_local_sp.md#poswithcov) 
You can also prepare your own simulation following [the guide for preparing the map.](./../../develop/SP/opil_api_local_sp.md#preparingmap)

* Start the calculation of local map updates that the AGV sees as new obstacles which are not mapped in the initial map.
```
terminal 3: roslaunch mapupdates startmapupdates.launch
```
You can check the topic of local map updates by typing: 
```
rostopic echo /robot_0/newObstacles
```
You can change the resolution of the local map updates by following the guide in Section [Map updates](./../../develop/SP/opil_api_local_sp.md#mapupdates).



To test if entities are sent to OCB, prepare the following docker-compose.yml on the machine where you want to have the OPIL server:
```
version: "3"
services:      
    #Context Broker
    orion:        
        image: fiware/orion
        ports:
            - 1026:1026
        command: 
            -dbhost mongo
            
    mongo:
        restart: always
        image: mongo:3.4
        command: --nojournal
```

To test sending local map updates and pose with covariance to OCB put in firos/config all json files from test/config_files/machine_2. Set the right IP addresses for your machine (server), OPIL server (contextbroker), and interface (check with ifconfig) in firos/config/config.json. Run firos in a new terminal and check entities in a web browser at the address <http://OPIL_SERVER_IP:1026/v2/entities.
```
terminal 5: rosrun firos core.py 
```

All previous steps can be replaced by calling a single launch file for the Local SP:
```
roslaunch sensing_and_perception local_robot_sim.launch 
```
This launch file starts the localization, local map updates and module for publishing Pose with Covariance, and firos. The launch file that does not start the simulator Stage because it will be started at RAN is **local_robot.launch**.

For more details how to set up firos config files check the Section [Interfaces](./../../develop/SP/opil_sp_interfaces.md). For more examples on sending and receiving these topics through OCB check the Section [Examples](./../../develop/SP/opil_api_sp.md#examplesOCB).