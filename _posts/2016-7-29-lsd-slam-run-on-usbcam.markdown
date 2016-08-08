---
layout: post
title:  "lsd-slam test"
date:   2016-07-29 16:44:59
categories: slam
---
some words before...

 **semantics are necessary to build bigger and better SLAM systems.**  
 **Robotics is the killer application of SLAM**  
 **Will end-to-end learning soon replace the mostly manual labor involved in building today’s SLAM systems?. **  
 ->  Integrating semantics into SLAM is often talk about, but it is easier said than done. Moreno's PhD thesis: Dense Semantic SLAM

 **SLAM (construction) <---help---> Deep Learning (perception)**
 large-scale "correspondence engines" -> large-scale datasets

调试过程：
1. 确认摄像头是或否工作，运用cheese
2. 安装ros下的库uvc_camera，确认摄像头所对应设备号/dev/video*
3. 在uvc_camera下新建launch文件夹，再新建uvc_camera_node.launch文件，复制以下代码
<launch><arg name="device" default="/dev/video0"/>
   <node pkg="uvc_camera" type="camera_node" name="uvc_camera" output="screen">
    <param name="width" type="int" value="640" />
    <param name="height" type="int" value="480" />
    <param name="fps" type="int" value="30" />
    <param name="frame" type="string" value="wide_stereo" />
    <param name="device" type="string" value="/dev/video0" />
  </node>
</launch>
4. 运行roscore
5. 运行rosrun lsd_slam_viewer viewer
6. 运行roslaunch uvc_camera uvc_camera_node.launch
7. 运行rosrun lsd_slam_core live_slam /image:=image_raw _calib:=~/ROS_DEV/rosbuild_ws/package_dir/lsd_slam/lsd_slam_core/calib/FOVxxx.cfg

 


