### Introduction

These are some brief notes concerning my ROS ROV Rev 1 robot that I demo’d at Robot Garden on 11/14.  The major components of the bot are shown in the block diagram below.

![Block Diagram](https://lh3.googleusercontent.com/8O-8jIVW_iLk2wbIAhG3K-kSvgsT4Ziqcr24JOmETBA=s0 "Block Diagram")

The Raspberry Pi 2 with a minimal Ubuntu installation is running roscore and and a slightly hacked version of the arduino ROS node from ros_arduino_bridge.  The Teensy is running a hacked-up version of the ros_arduino_bridge firmware which uses a different diff_controller routine to match my encoders.  The laptop runs an Ubuntu VM where I can run rqt and other ros nodes or scripts.

I’d like to mention that the book "[ROS By Example (Vol 1)](http://www.lulu.com/shop/r-patrick-goebel/ros-by-example-indigo-volume-1/ebook/product-22015937.html)" by R. Patrick Goebel was very helpful in sorting out a number issues.  It was especially helpful to me with networking info, and in getting rviz and dynamic reconfigure set up properly.  It includes a number of open-source scripts that provide concrete examples.

### WiFi and Startup

I set up the Pi’s wifi adapter as an access point to which the laptop connects ([reference](http://askubuntu.com/questions/180733/how-to-setup-an-access-point-mode-wi-fi-hotspot?rq=1)).  For me this was the simplest way to get the networking going, although I understand that some wifi adapters work better than others in this mode.  The adapter I used came in the Pi starter [kit](http://www.amazon.com/CanaKit-Raspberry-Complete-Original-Preloaded/dp/B008XVAVAW/ref=sr_1_1?s=pc&ie=UTF8&qid=1448115831&sr=1-1&keywords=canakit) I bought and [uses](http://www.canakit.com/raspberry-pi-starter-kit.html) the Ralink RT5370 chipset.  The Amazon site claims:

![Amazon claim](https://lh3.googleusercontent.com/l_8R_4h5Zg4D6YmFZrx7kgjvis4AKXPvcq2hvTIz728=s0 "Copy of Amazon Pi2 Wifi Access Point.PNG")

To bring up the robot, I launch roscore and the arduino node on the Pi via ssh from the laptop:

```
ubuntu@ubuntu:~$ cd catkin_ws/
ubuntu@ubuntu:~/catkin_ws$ roslaunch ros_arduino_python arduino.launch &
```

One thing I learned about ssh operation is that appending "&" at the end of a command line runs the process in the background.  This is useful if you want to run other commands on the Pi after launching the ROS node.

Some of the initial output shows that roslaunch starts the necessary roscore components, including the parameter server, and launches the arduino node from arduino_node.py:

```
started roslaunch server http://ubuntu.local:48914/

SUMMARY
========

PARAMETERS
 * /arduino/FRAME_RATE: 1
 * /arduino/Kd: 0
 * /arduino/Ki: 30
 * /arduino/Ko: 6000
 * /arduino/Kp: 50
 * /arduino/accel_limit: 1.0
 * /arduino/base_controller_rate: 10
 * /arduino/base_frame: base_link
 * /arduino/baud: 57600
 * /arduino/encoder_resolution: 48
 * /arduino/gear_reduction: 1.0
 * /arduino/min_abs_speed: 80
 * /arduino/motors_reversed: True
 * /arduino/port: /dev/ttyACM0
 * /arduino/rate: 50
 * /arduino/sensors/arduino_led/direction: output
 * /arduino/sensors/arduino_led/pin: 11
 * /arduino/sensors/arduino_led/rate: 5
 * /arduino/sensors/arduino_led/type: Digital
 * /arduino/sensorstate_rate: 10
 * /arduino/timeout: 0.1
 * /arduino/use_base_controller: True
 * /arduino/wheel_diameter: 0.063
 * /arduino/wheel_track: 0.099
 * /rosdistro: indigo
 * /rosversion: 1.11.13

NODES
  /
    arduino (ros_arduino_python/arduino_node.py)

auto-starting new master
process[master]: started with pid [1574]
ROS_MASTER_URI=http://localhost:11311

setting /run_id to cd0b1b22-1dd2-11b2-8435-000f6004684f
process[rosout-1]: started with pid [1587]
started core service [/rosout]
process[arduino-2]: started with pid [1604]
[DEBUG] [WallTime: 316.733049] init_node, name[/arduino], pid[1604]
[DEBUG] [WallTime: 316.735455] binding to 0.0.0.0 0
[DEBUG] [WallTime: 316.737375] bound to 0.0.0.0 43504
[DEBUG] [WallTime: 316.740172] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.741912] [/arduino/get_loggers]: new Service instance
[DEBUG] [WallTime: 316.753033] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.754664] [/arduino/set_logger_level]: new Service instance
[DEBUG] [WallTime: 316.843519] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.845273] [/arduino/servo_write]: new Service instance
[DEBUG] [WallTime: 316.856840] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.858713] [/arduino/servo_read]: new Service instance
[DEBUG] [WallTime: 316.869857] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.871725] [/arduino/digital_set_direction]: new Service instance
[DEBUG] [WallTime: 316.882462] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.883965] [/arduino/digital_write]: new Service instance
[DEBUG] [WallTime: 316.894381] ... service URL is rosrpc://ubuntu.local:43504
[DEBUG] [WallTime: 316.895836] [/arduino/analog_write]: new Service instance
```

Next the arduino node connects to the Teensy on /dev/ttyACM0:
```
Connecting to Arduino on port /dev/ttyACM0 ...
Connected at 57600
Arduino is ready.
[INFO] [WallTime: 317.921358] Connected to Arduino on port /dev/ttyACM0 at 57600 baud
[INFO] [WallTime: 317.953352] arduino_led {'direction': 'output', 'type': 'Digital', 'rate': 5, 'pin': 11}
Updating PID parameters
[DEBUG] [WallTime: 318.150169] connecting to ubuntu.local 43504
[INFO] [WallTime: 318.171216] Started base controller for a base of 0.099m wide with 48 ticks per rev
[INFO] [WallTime: 318.175967] Publishing odometry data at: 10.0 Hz using base_link as base frame
```

Note that I set the ROS_HOSTNAME in the .bashrc file on the Pi:

```
export ROS_HOSTNAME=ubuntu.local</td>
```

On my laptop in the VM, I can launch rqt in a terminal after changing to my catkin workspace and pinging the Pi to verify connectivity:

```
ros@ros-VirtualBox:~$ cd catkin_rov1/
ros@ros-VirtualBox:~/catkin_rov1$ ping ubuntu.local
PING ubuntu.local (10.42.0.1) 56(84) bytes of data.
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=0.945 ms
64 bytes from 10.42.0.1: icmp_seq=2 ttl=64 time=3.17 ms
^C
--- ubuntu.local ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.945/2.059/3.174/1.115 ms
ros@ros-VirtualBox:~/catkin_rov1$ 
ros@ros-VirtualBox:~/catkin_rov1$ rqt
[ INFO] [1448133205.748886818]: Stereo is NOT SUPPORTED
[ INFO] [1448133205.749155943]: OpenGl version: 3 (GLSL 1.3).
```

In .bashrc on the VM I included:
```
# for my present Ubuntu VM
export ROS_HOSTNAME=ros-VirtualBox.local
# for rviz to work with VM on my Win 8.1 host
export LIBGL_ALWAYS_SOFTWARE=1
# for ROS_MASTER on RPi, Ros Rov Rev 1
export ROS_MASTER_URI=http://ubuntu.local:11311
```

Obviously the first environment variable above takes care of the ROS_HOSTNAME for the VM, while the third one tells the VM where the ROS master is running.  The LIBGL_ALWAYS_SOFTWARE=1 setting is needed to allow rviz to run on my machine.  I think this may be related to my host (Win 8.1) or its specific graphics hardware.  It may not be necessary on all platforms.

### RQT

Rqt has many plug-ins available that perform a wide array of useful functions.  One of the first ones to check out is rviz.  When rviz launches, I have to change some selections in the left column from the defaults to get the odometry data displayed (someday I’ll learn how to change the default values :smile: ):

* In "Fixed Frame" choose “odom”.
* Click "Add" and choose “Odometry” to add this info to the display.
* Expand the Odometry section and click in the blank next to "Topic".  
* Choose “/odom” from the drop-down.  This adds the red pose / location arrow to the grid display.

The figure below shows this setup.  From here the red arrows will track changes in the odometry data showing the trajectory the bot thinks it’s traversed.

![Rviz Display](https://lh3.googleusercontent.com/8iXYojNwma04k5-kJwtPEM9HjuvE7EUstbfEeHug0Co=s0 "Rviz Display")

The Robot Steering plug-in can be used for teleop: 

![Robot Steering](https://lh3.googleusercontent.com/9qy_VWjEGsc_DJ7pGuNFifwvUXLngyAu5_Mz726BpyY=s0 "image_2.png")

The Node Graph plug-in displays the current ROS graph:

![Node Graph](https://lh3.googleusercontent.com/zrP7bHkpBCzoqqVlpS-OZS0LRamBjaeC-LF2Ghh9Cqg=s0 "image_3.png")

Rqt also includes Message Publisher and Topic Monitor plug-ins which make it easier to work with topics.  For example, it’s easier to send test Twist messages using Message Publisher than typing on the command line.

![Message Pub and Topic Mon](https://lh3.googleusercontent.com/oY7EByS4DpMEPnoBRXty69c0yPt_5MsOw2Rab_HvaNw=s0 "image_4.png")

Dynamic Reconfigure is a useful plug-in that allows parameters to be changed on the fly.  For example, the image below shows sliders for adjusting a set of parameters that are defined in calibration program (calibrate_linear.py from "ROS By Example" by R. Patrick Goebel).  Running that python script in a terminal and then starting or refreshing the plug-in allows you to interactively change the parameters.  In this example, checking “start_test” runs a script to move the bot forward 1 meter judged by the odometry data.  Based on the actual distance moved, the correction factor can be adjusted and the test repeated to determine what value should be used to correct the bot’s odometry.

![Dynamic Recongifure](https://lh3.googleusercontent.com/X2BQ7MphhYQp9xVjXsEFuxnBo8f-6yYb-MJ0H_sznxI=s0 "image_5.png")

I’ve only illustrated a few of rqt’s plug-ins.  There are many more, including the useful capability to plot topic data vs time and a console to display filtered log messages.  Have fun exploring further!

### Remap and Teleop From Android

At the 11/14 meeting Fred was able to drive my bot by modifying his Android app so send Twist messages on the "cmd_vel" topic.  The original app uses the “turtle1/cmd_vel” topic.  Fred had sent me the original app, so after the meeting I tried it using ROS’s capability to remap names in launch files.  All I had to do to get things working was to change the launch file for the arduino node from:

```
<launch>
   <node name="arduino" pkg="ros_arduino_python" type="arduino_node.py" output="screen">
      <rosparam file="$(find ros_arduino_python)/config/my_arduino_params.yaml" command="load" />
   </node>
</launch>
```
to:

```
<launch>
   <node name="arduino" pkg="ros_arduino_python" type="arduino_node.py" output="screen">
      <rosparam file="$(find ros_arduino_python)/config/my_arduino_params.yaml" command="load" />
      <remap from="cmd_vel" to="turtle1/cmd_vel" />
   </node>
</launch>
```

