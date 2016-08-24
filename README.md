# pi3_ros
## 前言

我之前已經試過了許許多多的方法 , 但是在raspberry pi 3 上面都不行 , 這邊列出別人使用但我自己試都不行的方法 , 網路上很多人都很順利 , 歡迎大家來討論 http://wiki.ros.org/ROSberryPi/Setting%20up%20ROS%20on%20RaspberryPi http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Indigo%20on%20Raspberry%20Pi http://wiki.ros.org/indigo/Installation/UbuntuARM

https://drive.google.com/file/d/0B4lNqhhSk7SQLVNZYzBZbm9mUG8/view?usp=sharing

https://drive.google.com/file/d/0B4lNqhhSk7SQLVNZYzBZbm9mUG8/edit

## mavros on raspberry pi 3

首先要準備一張大於8Ｇ的sd 卡 OS有點大 去https://ubuntu-mate.org/raspberry-pi/ 下載ubuntu-mate按照上面的安裝指令

     sudo apt-get install gddrescue xz-utils
     unxz ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img.xz
     sudo ddrescue -D --force ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img /dev/sdx
安裝完後開機並設定網路我自己試用固定IP就看自己喜歡都行

## 安裝ROS

參考：http://wiki.ros.org/kinetic/Installation/Ubuntu Setup your sources.list Setup your computer to accept software from packages.ros.org. ROS Kinetic ONLY supports Wily (Ubuntu 15.10), Xenial (Ubuntu 16.04) and Jessie (Debian 8) for debian packages.

     sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
Set up your keys

     sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 0xB01FA116
Installation First, make sure your Debian package index is up-to-date:

     sudo apt-get update
There are many different libraries and tools in ROS. We provided four default configurations to get you started. You can also install ROS packages individually. In case of problems with the next step, you can use following repositories instead of the ones mentioned above ros-shadow-fixed

Desktop-Full Install: (Recommended) : ROS, rqt, rviz, robot-generic libraries, 2D/3D simulators, navigation and 2D/3D perception

     sudo apt-get install ros-kinetic-desktop-full //my ubuntu-mate16.04 can't find this

or click here
Desktop Install: ROS, rqt, rviz, and robot-generic libraries

     sudo apt-get install ros-kinetic-desktop //second install

or click here
ROS-Base: (Bare Bones) ROS package, build, and communication libraries. No GUI tools.

     sudo apt-get install ros-kinetic-ros-base //fist install

or click here
Individual Package: You can also install a specific ROS package (replace underscores with dashes of the package name): sudo apt-get install ros-kinetic-PACKAGE

e.g.

     sudo apt-get install ros-kinetic-slam-gmapping

To find available packages, use:

     apt-cache search ros-kinetic
Initialize rosdep Before you can use ROS, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core components in ROS.

     sudo rosdep init
     rosdep update
Environment setup It's convenient if the ROS environment variables are automatically added to your bash session every time a new shell is launched:

     echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
     source ~/.bashrc
If you have more than one ROS distribution installed, ~/.bashrc must only source the setup.bash for the version you are currently using. If you just want to change the environment of your current shell, instead of the above you can type:

     source /opt/ros/kinetic/setup.bash
Getting rosinstall rosinstall is a frequently used command-line tool in ROS that is distributed separately. It enables you to easily download many source trees for ROS packages with one command. To install this tool on Ubuntu, run:

sudo apt-get install python-rosinstall
Install mavros

https://github.com/mavlink/mavros/blob/master/mavros/README.md#installation Source installation Use wstool utility for retrieving sources and catkin tool for build. NOTE: The source installation instructions are for the ROS Kinetic release.

     sudo apt-get install python-catkin-tools python-rosinstall-generator -y

### 1. Create the workspace: unneded if you already has workspace
     mkdir -p ~/catkin_ws/src
     cd ~/catkin_ws
     catkin init
     wstool init src

### 2. Install MAVLink
     #    we use the Kinetic reference for all ROS distros as its not distro-specific and up to date
    rosinstall_generator --rosdistro kinetic --upstream mavlink | tee /tmp/mavros.rosinstall

### 3. Install MAVROS: get source (upstream - released)
    rosinstall_generator --upstream mavros | tee -a /tmp/mavros.rosinstall
    # alternative: latest source
    # rosinstall_generator --upstream-development mavros | tee -a /tmp/mavros.rosinstall

### 4. Create workspace & deps
    
    wstool merge -t src /tmp/mavros.rosinstall
    wstool update -t src -j4
    rosdep install --from-paths src --ignore-src -y

###if there are some problem like

Exception caught during install: Error processing 'mavlink' : [mavlink] Checkout of https://github.com/mavlink/mavlink-gbp-release.git version 2016.8.8 into /home/swimglass/catkin_ws/src/mavlink failed.

    cd ~/catkin_ws/src
    git clone https://github.com/mavlink/mavlink.git
    cd ~/catkin 
and do next

### 5. Build source

    catkin build -j2

#### 6. Make sure that you use setup.bash or setup.zsh from workspace.
Else rosrun can't find nodes from this workspace.
    
    $source devel/setup.bash

照著這個流程就可以把mavros裝好了

##補充

如果遇到了run out of memory可以參考下面 if you are running out of memory, you may want to increase your swap memory. Or you might have no swap enabled at all. In Ubuntu (it should work for other distributions as well) you can check your swap by:

    $sudo swapon -s
if it is empty it means you don't have any swap enabled. To add a 1GB swap:

    $sudo dd if=/dev/zero of=/swapfile bs=1024 count=1024k
    $sudo mkswap /swapfile
    $sudo swapon /swapfile
Add the following line to the fstab to make the swap permanent.

$sudo vim /etc/fstab

     /swapfile       none    swap    sw      0       0 
Source and more information can be found here.
