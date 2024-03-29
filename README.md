rgbdslam-hydro
==============

Rgbdslam- freiburg that works with hydro.

To compile this you need add the header file OctomapROS.h to the folder: 
/opt/ros/hydro/include/octomap_ros/
file OctomapROS.h can be download from: 
http://docs.ros.org/fuerte/api/octomap_ros/html/OctomapROS_8h_source.html

#############################################################################
Quick Start:

##########  Check out codes  #############
$ cd ~/catkin_ws/src
$ git clone {this page link}

###########  Install dependencies  #########
$ apt-get install ros-hydro-libg2o
$ apt-get install ros-hydro-octomap ros-hydro-octomap-mapping ros-hydro-octomap-msgs ros-hydro-octomap-ros ros-hydro-octomap-rviz-plugins ros-hydro-octomap-server 

You also need add a deprecated header file OctomapROS.h to the folder:  /opt/ros/hydro/include/octomap_ros/
which can be download from: 
http://alufr-ros-pkg.googlecode.com/svn/branches/octomap_stacks-groovy-devel/octomap_ros/include/octomap_ros/OctomapROS.h

###########             Compile         ############
$ cd ~/catkin_ws
$ catkin_make
$ rosmake –pre-clean rgbdslam

#########    Install drivers for Kinect     #########
$ sudo apt-get install ros-hydro-libfreenect

###########      Run RGB-D SLAM     ###########
you might need to change the listening topic accroding to your drivers. if it is libfreenect:
$ roslaunch freenect_launch slam_freenect.launch
$ roslaunch rgbdslam optimized.launch

if you use openni, run with:
$ roslaunch rgbdslam kinect+rgbdslam.launch
Alternatively you can start the openni nodes and RGBDSLAM separately, e.g.:
$ roslaunch openni_launch openni.launch 
$ rosrun rgbdslam rgbdslam


#############################################################################
Additional information can be found here:
http://www.ros.org/wiki/rgbdslam

==============

LICENSE INFORMATION ##########################################################

This software is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE.  The authors allow the users to use and modify the
source code for their own research. Any commercial application, redistribution,
etc has to be arranged between users and authors individually.

RGBDSLAM is licenced under GPL v.3.

============== Original README of rgbdslam_freiburg ================

WARNING TO USERS OF THE SERVICE CALL INTERFACE: 
The service definitions have minor changes, which will break existing service
calls! Most importantly: The field "comand" is now correctly spelled "command" 
Another service "rgbdslam/ros_ui_s" has been created that expects a string 
value. Some of the "save" commands of the "rgbdslam/ros_ui" service calls have 
been moved to the new service: save_cloud,  save_g2o_graph,
save_trajectory,  save_features,  save_individual. "save_octomap" has been 
added. The string "value" is the respective filename to save to. See the 
documentation below, or the file src/ros_service_ui.cpp.

Additional information can be found here:
http://www.ros.org/wiki/rgbdslam

INSTALLATION ################################################################### 
Download rgbdslam with the following command to your ros workspace

$ svn co http://alufr-ros-pkg.googlecode.com/svn/trunk/rgbdslam_freiburg

If not yet installed, install rosdep first, then execute

$ rosdep update rosdep install rgbdslam_freiburg

This should take care of all system dependencies (e.g. libg2o). You can then
compile it with

$ rosmake rgbdslam_freiburg

which will take a while.

If you want to use RGB-D SLAM with a Kinect or Xtion Pro, you also need to
install openni_launch (apt: ros-fuerte-openni-launch). 
If you want to edit the saved point clouds you might want to install meshlab.

Configuration

There are several example launch-files that set the parameters of RGB-D SLAM
for certain use cases. For a definitive list of all settings and their default
settings have a look at their quite readable definition in
src/parameter_server.cpp or (with the current settings instead of the default)
in the GUI Menu Settings->View Current Settings.

The various use-case launch-files might not work correctly yet, as they are not
regularly tested. You should get them running if you fiddle with the topics
("rostopic list" and "rosnode info" will help you.  "rxgraph" is great too).


Usage ##############################################################

Most people seem to want the registered point cloud. It is by default sent out
on /rgbdslam/batch_clouds when you command RGB-D SLAM to do so (see below). The
clouds sent are actually the same as before, but the according transformation -
by default from /map to /openni_camera - is sent out on /tf.

This was also the way the octomap_server got the point clouds my making it
listen to the mentioned topic and set the fixed frame_id to /map (See the
launch file octomap_server.launch). Meanwhile, a variant of the
octomap server node is compiled into the rgbdslam node. This allows to
create the octomap directly without sending data. In the GUI this can be 
done by selecting "Save Octomap" from the "Data" Menu.

USAGE with GUI #################################################################

To start RGBDSLAM launch 
$ roslaunch rgbdslam kinect+rgbdslam.launch

Alternatively you can start the openni nodes and RGBDSLAM separately, e.g.:
roslaunch openni_camera openni_node.launch rosrun rgbdslam rgbdslam

To capture models either press space to start recording a continuous stream or
press enter to record a single frame. To reduce data redundancy, sequential
frames from (almost) the same position are not included in the final model.
The 3D visualization always shows the globally optimized model. Neighbouring
points are triangulated except at missing values and depth jumps.



USAGE without GUI ##############################################################

The RosUI is an alternative to the Grapical_UI to run the rgbdslam headless,
for example on the PR2.  rgbdslam can then be used via service-calls.
The possible calls are:
 * /rgbdslam/ros_ui   {reset,  quick_save,  send_all,  delete_frame,  optimize,  reload_config, save_trajectory}
 * /rgbdslam/ros_ui_b {pause, record} {true, false}
 * /rgbdslam/ros_ui_f {set_max} {float}
 * /rgbdslam/ros_ui_s {save_octomap,  save_cloud,  save_g2o_graph,  save_trajectory,  save_features,  save_individual} {filename}

To start the rgbdslam headless use the headless.launch: 
$ roslaunch rgbdslam headless.launch

Capture single frames via: 
$ rosservice call /rgbdslam/ros_ui frame

Capture a stream of data: 
$ rosservice call /rgbdslam/ros_ui_b pause false

Send point clouds with computed transformations (e.g., to rviz or octomap_server): 
$ rosservice call /rgbdslam/ros_ui send_all

Save data using one of the following:

All pointclouds in one file quicksave.pcd in rgbdslam/bin-directory: 
$ rosservice call /rgbdslam/ros_ui_s save_cloud

Every pointcloud in its own file in rgbdslam/bin-directory: 
$ rosservice call /rgbdslam/ros_ui save_individual

/rgbdslam/ros_ui: 

 * reset	''resets the graph, delets all nodes (refreshes only when capturing new images)''
 * frame	''capture one frame from the sensor''
 * optimize	''trigger graph optimizer''
 * reload_config	''reloads the paramters from the ROS paramter server''
 * quick_save ''saves all pointclouds in one file quicksave.pcd in rgbdslam/bin-directory''
 * send_all	''sends all pointclouds to /rgbdslam/transformed_cloud (can be visualized with rviz)''
 * delete_frame	''delete the last frame from the graph (refreshes only when capturing new images)''
	
/rgbdslam/ros_ui_b: 

 * pause <flag>	''pauses or resumes the capturing of images'' 
 * record <flag>  ''pauses or stops the recording of bag-files, can be found in the rgbdslam/bin-directory''
	
/rgbdslam/ros_ui_f: 

 * set_max <float>	''filters out all datapoints further away than this value (in cm, only for saving to files)''

/rgbdslam/ros_ui_s: 

 * save_features <filname> ''saves the feature locations and descriptors in a yaml file with the given filename''
 * save_cloud	<filename> ''saves the cloud to the given filename (should end with .ply or .pcd)''
 * save_individual <fileprefix>	''saves every scan in its own file (appending a suffix to the given prefix)''
 * save_octomap	<filename> ''saves the cloud to the given filename''
 * save_trajectory	<fileprefix> ''saves the sensor trajectory to the file <fileprefix>_estimate.txt''



FURTHER HELP ##################################################################

1. If you are located in Germany and get errors loading the saved ply files
into meshlab, try switching to U.S. locale or replace the decimal point with a
comma in your .ply file 
2. To speed up compile times consider to use "export ROS_PARALLEL_JOBS=-j<#cpus>"
before rosmake, but you should have lots of memory as gcc may take up to 2GB 
for four parallel jobs.

If you have questions regarding installation or usage of RGBDSLAM please refer
to http://answers.ros.org/questions/?tags=RGBDSLAM For further questions,
suggestions, corrections of this README or to submit patches, please contact
Felix Endres (endres@informatik.uni-freiburg.de)

Apart from this manual, detailed code documentation can be created using rosdoc
("rosrun rosdoc rosdoc rgbdslam"), which will create a "doc" folder in your
current directory.



GICP AND SIFTGPU ###############################################################

If there are problems related to the compilation or linking of GICP or SIFTGPU,
you can deactivate these features at the top of CMakeLists.txt. You might get
even faster GPU features setting SIFT_GPU_MODE to 1 (CUDA) but you will need to
install proprietary drivers: SiftGPU uses (in our case) CUDA, which needs a new
NVidia GPU (see http://www.nvidia.com/object/cuda_gpus.html).  For installing
the development drivers and the CUDA SDK you can use the following tutorial:
http://sublimated.wordpress.com/2011/03/25/installing-cuda-4-0-rc-on-ubuntu-10-10-64-bit/
or for ubuntu 10.04: http://ubuntuforums.org/showthread.php?t=1625433 (tested
on Ubuntu 10.04 x64) To use SiftGPU you should install "libdevil-dev".
  
  Additional compiling information can be changed in
  external/siftgpu/linux/makefile.

  GICP Generalized ICP can be (de)activated for refining the registration. For
  more information see http://stanford.edu/~avsegal/generalized_icp.html



LICENSE INFORMATION ############################################################

This software is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE.  The authors allow the users to use and modify the
source code for their own research. Any commercial application, redistribution,
etc has to be arranged between users and authors individually.

RGBDSLAM is licenced under GPL v.3.
