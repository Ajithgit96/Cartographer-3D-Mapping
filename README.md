**Overview**

* This project deals with 3D Mapping of environment with Cartographer.

* Cartographer is a system that provides real-time simultaneous localization and mapping (SLAM) in 2D and 3D across multiple platforms and sensor configurations. It also provides Cartographer’s ROS integration. 

* Refer to wiki page of this project for detailed explanation on establishing connection with 3D LiDAR, visualzing point clouds and 3D Mapping with Cartographer.

* Note: All the contents of the cartographer could not be uploaded into this repository, as cartographer project has sub projects (cartographer, cartographer_ros) which will be empty here. But all the files created and in which the changes were made are present in this repository. Hence it is suggested to follow cartographer installation instructions from the below link and use this repository as reference.

* .lua Configuration files, launch files and urdf configuration files can be found in ‘install_isolated/share/cartographer_ros'

* Source: https://google-cartographer-ros.readthedocs.io/en/latest/index.html#

**Establishing connection with 3D LiDAR and visualizing point cloud**

- Performed in ROS melodic and eloquent.

- Below link has information about steps involved to achieve this task:
- [http://wiki.ros.org/velodyne/Tutorials/Getting%20Started%20with%20the%20Velodyne%20VLP16](http://wiki.ros.org/velodyne/Tutorials/Getting%20Started%20with%20the%20Velodyne%20VLP16)
- Github repository for velodyne ROS driver is: https://github.com/ros-drivers/velodyne/tree/master
- In this Github repository, master branch has driver for ROS melodic and ros2 branch has driver for ROS eloquent distribution
- In addition to content in above link, following description and hints are helpful:
- Setting up the hardware connections
- Identifying the port of your computer to which 3D LiDAR is connected.
- Useful terminal commands and techniques include:
   - $ ifconfig -a
   - Wireshark software can be particularly helpful in identifying to which port LiDAR is connected, the kind of communication protocol running and also to get the IP address of VLP 16.
   - Hint: Use below terminal command if your computer has permission problems in opening wireshark.
   - $ sudo wireshark - this command can be helpful to give permission to wireshark
- 
- Three communication protocols that might appear when you run wireshark with LiDAR connected are ARP, DHCP, UDP.
- ARP (Address Resolution Protocol) – Indicates successful communication and gives IP addresses of both computer port and LiDAR.
- ARP protocol in wireshark would look like:
![Screenshot_from_2020-09-10_01-30-21](uploads/2ddf1e8f8c9689e9024a127c76348d5f/Screenshot_from_2020-09-10_01-30-21.png)
- The statement here "Who has 192.168.42.31? Tell 192.168.42.35" means 192.168.42.35 is sender (3D LiDAR) IP address and 192.168.42.31 (computer) is Target IP address.
- Note: By default, the IP address of the VLP-16 is set to be 192.168.1.201 by the factory.
- But the IP address of VLP 16 used in this project was 192.168.42.35
- Configure your computer’s IP address through the Gnome interface in linux as per the tutorial in above mentioned link.
- Statically assign an IP to the port to which LiDAR is connected using: $ sudo ifconfig enp7s0 192.168.42.31

- Add a static route to the LIDAR's IP address using: $ sudo route add 192.168.42.35 enp7s0

- These commands will differ for different computers
- Now the connection must be established and it results in UDP communication protocol happening, which would appear in wireshark like below:
![Screenshot_from_2020-09-10_17-35-16](uploads/af427ca7706d281c2079a511dd536a2d/Screenshot_from_2020-09-10_17-35-16.png)

- By putting in LiDAR IP address in your web browser, the following webpage should appear:
![Screenshot_from_2020-09-10_17-49-46](uploads/b36cf2408dcdd26ce21122e9390cb59b/Screenshot_from_2020-09-10_17-49-46.png)

- Note: DHCP (Dynamic Host Configuration Protocol) - It is not suggested to turn DHCP on. Before doing so, ensure a DHCP server on the network is available to provide the sensor with an IP address. If you turn on DHCP and loose contact with the sensor, there is a section in troubleshooting in above manual to solve this problem.

- Useful links of VLP 16 user manuals: Can be useful for dos, don’ts and for troubleshooting. Troubleshooting can be helpful when faced with unfamiliar problems.
   - [http://isaza.co/VELODYNE/63-9266%20REV%20A%20WEBSERVER%20USER%20GUIDE,HDL-32E%20&%20VLP-16.pdf](http://isaza.co/VELODYNE/63-9266%20REV%20A%20WEBSERVER%20USER%20GUIDE,HDL-32E%20&%20VLP-16.pdf)
   - [https://greenvalleyintl.com/wp-content/uploads/2019/02/Velodyne-LiDAR-VLP-16-User-Manual.pdf](https://greenvalleyintl.com/wp-content/uploads/2019/02/Velodyne-LiDAR-VLP-16-User-Manual.pdf)

- Note: As the IP address of VLP 16 used in this project was 192.168.42.35, it requires an IP address change in the velodyne driver (ROS melodic) in the file velodyne/velodyne_driver/config/VLP16-velodyne_driver_node-params.yaml, ‘device_ip: 192.168.1.201’ must be changed to ‘device_ip: 192.168.42.35’
- Note: The 3D LiDAR data is currently published with reference to the frame "velodyne".
- Point Cloud data is published with the topic name "/velodyne_points" and when visualized in Rviz, it would appear as below.
![Screenshot_from_2020-12-15_11-41-31](uploads/77e316b59fb6f6d8b9188e115d2af98f/Screenshot_from_2020-12-15_11-41-31.png)

- Terminal commands for ROS melodic to run the launch file and visualizing in ROS melodic:
   - $ roslaunch velodyne_pointcloud VLP16_points.launch
   - $ rostopic echo /velodyne_points
   - $ rosrun rviz rviz -f velodyne
- Terminal commands for ROS eloquent:
   - $ ros2 launch velodyne velodyne-all-nodes-VLP16-launch.py
   - $ ros2 topic echo /velodyne_points
   - $ ros2 run rviz2 rviz2



**Cartographer 3D Mapping**

- Cartographer was used in this project to perform real-time simultaneous localization and mapping (SLAM) in 3D.

- 3D Mapping in this project was performed in ROS melodic distribution with sensor data from both 3D LiDAR and IMU sensors. The workflow looks like below and is explained in [https://google-cartographer-ros.readthedocs.io/en/latest/ros_api.html](https://google-cartographer-ros.readthedocs.io/en/latest/ros_api.html):
![Capture](uploads/e0a145a26f978b0bdcd25d4fd7d00241/Capture.PNG)

source: https://google-cartographer-ros.readthedocs.io/en/latest/

- IMU sensor is mandatory for building 3D Maps. The IMU sensor helps in measuring gravity (Z direction) and deriving Roll, pitch.

- The message type is ‘sensor_msgs/Imu’ which is published with the topic name ‘/imu/data’.

- The following link has a detailed explanation Cartographer’s ROS integration and about all features in it: [https://google-cartographer-ros.readthedocs.io/en/latest/](https://google-cartographer-ros.readthedocs.io/en/latest/)

- Accomplished Steps include:
   - Cartographer’s ROS integration
   - Running Cartographer ROS on a demo bag
   - Running Cartographer ROS on our own ROS bag or with real time sensor data (3D LiDAR and IMU). This step involves certain changes to be made in cartographer to be able to run our bag. 
   - Following changes or additions are made:
   - Unlike 2 LiDAR sensors (horizontal & vertical) used in cartographer code, only 1 3D LiDAR is used in this project. So, all necessary changes were done accordingly.
   - Changing robot configuration in .lua configuration in ‘install_isolated/share/cartographer_ros/configuration_files/my_robot.lua’.
   - Robot’s TF tree is calculated from robot's URDF defined in ‘install_isolated/share/cartographer_ros/urdf/my_robot.urdf’ file. The frames of sensors must be renamed to the frames to which sensor data is linked to. LiDAR data is linked to the frame ‘velodyne’ and IMU data is linked to the frame ‘imu’.
   - Remapping of topics is done in concerned launch files that can be found in ‘install_isolated/share/cartographer_ros/launch’ folder. New launch files created are with the name my_robot.

- Note: As Cartographer’s ROS integration uses tf2, this requires a minor change in launch/xsens_driver.launch (IMU sensor driver). 
   - The frame_id field in the header section must be changed from ‘/imu’ to ‘imu’.
   - Hence, IMU data is linked to the frame ‘imu’.
   - Errors that occur when above settings are not proper or when there is error in TF tree are:
   - [ WARN] [1606045752.492393806, 1599753182.586311340]: W1122 12:49:12.000000 14432 tf_bridge.cc:52] "velodyne" passed to lookupTransform argument source_frame does not exist.
   - [ WARN] [1606045752.520877411, 1599753182.606446007]: Could not compute submap fading: "map" passed to lookupTransform argument target_frame does not exist.

- TF tree that results from the above URDF configuration looks like below:
![Screenshot_from_2020-11-22_12-33-44](uploads/1c1c0266a0535da37571b79a9b05c89b/Screenshot_from_2020-11-22_12-33-44.png)

- TF tree with errors and lacking proper .lua configuration would look like:
![Screenshot_from_2020-11-22_12-51-01](uploads/dbc1ee577a5abc1cabd9a39c7824ea5f/Screenshot_from_2020-11-22_12-51-01.png)
- With these changes, cartographer must be able to generate 3D Maps of the environment in ROS.
- Hint: In case of not getting desired results with ROS bags, cartographer has the feature ‘Validate your bag’.
- Result of 'Validate your bag': The highlighted section here suggests that IMU linear acceleration value is different from expected value
![Screenshot_from_2021-01-05_16-21-49](uploads/cc7d4d172f5fb5a40e5a8ae7a051dc44/Screenshot_from_2021-01-05_16-21-49.png)
- Challenge: Generated 3D point cloud map of environment rotates as the LiDAR (velodyne frame) rotates. It is not a desired feature.
- Next steps: 
   - Generating a map of bigger environment with static objects (which do not move) and saving it.
   - Using this map to navigate autonomously using projects like 'navigation 2' in ROS. It works like we give an approximate initial pose of the robot and a navigation goal. 
   - Details present in: [https://navigation.ros.org/index.html](https://navigation.ros.org/index.html)
