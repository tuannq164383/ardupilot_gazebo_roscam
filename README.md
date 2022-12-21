# SITL + Ardupilot + Gazebo + ROS Camera plugin Tutorial
SITL + ArduPilot + Gazebo + ROS Camera Plugin (Software In Loop Simulation Interfaces, Models)

## Description :

Finally an all in one tutorial for setting up your virtual drone using Ardupilot (Arducopter) + SITL in a complete 3D virtual environment provided by Gazebo.

At last but not least (Actually the most complicated to find examples of...) added the camera plugin from ROS to publish the images so you can take them outside Gazebo and do something with them...

Following are the requirements, setup steps and finally how to of each part.. 


## Requirements :
* Ubuntu (20.04.5 LTS) Full 3D graphics hight recommended.
* Gazebo version 8.x or greater (Recommended 11)
* ROS noetic (Required to work with Gazebo)
* MAVROS

## Gazebo 11 :

### Setup your computer to accept software from packages.osrfoundation.org.
````
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
````

### Setup keys
````
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
````

### Install Gazebo
````
sudo apt-get update
sudo apt-get install gazebo11
sudo apt-get install libgazebo11-dev
````

### Test Gazebo
````
gazebo --verbose
````

## ROS noetic installation :

Install ROS with sudo apt install ros-noetic-desktop-full (follow instructions here http://wiki.ros.org/noetic/Installation/Ubuntu).

### Configure your Ubuntu repositories
````
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

sudo apt install curl # if you haven't already installed curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

sudo apt update
````

### Install ROS noetic
````
sudo apt install ros-noetic-desktop-full
````

### Initialize ROS
````
sudo rosdep init
rosdep update
````

### Environment setup
````
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
````

### Dependencies for building packages
````
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
````

## MAVROS installation :

MAVLink extendable communication node for ROS with proxy for Ground Control Station (See original instructions here http://ardupilot.org/dev/docs/ros-install.html#installing-mavros).

### Configure your Ubuntu repositories
````
sudo apt-get install ros-noetic-mavros ros-noetic-mavros-extras

cd ~/

wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh

chmod a+x install_geographiclib_datasets.sh

./install_geographiclib_datasets.sh
````

### For ease of use on a desktop computer, please also install RQT
````
sudo apt-get install ros-noetic-rqt ros-noetic-rqt-common-plugins ros-noetic-rqt-robot-plugins
````

### Install catkin tools
````
sudo apt-get install python3-catkin-tools
````

Now that we have everything correctly installed we can begin our system configuration

## Environment setup and configuration :
* STEP 1 - SITL Ardupilot
* STEP 2 - Ardupilot gazebo plugin (Original khancyr version)
* STEP 3 - Gazebo ROS plugin (roscam)
* STEP 4 - Connect ArduPilot to ROS


## SETP 1 - SITL Ardupilot installation :

Instructions taken from ardupilot.org (See original instructions here http://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html).

### Clone ArduPilot repository

````
cd ~/
git clone https://github.com/ArduPilot/ardupilot.git
cd ardupilot
git submodule update --init --recursive
````

### Install some required packages

If you are on a debian based system (such as Ubuntu or Mint), we provide a script that will do it for you. From ardupilot directory :

````
Tools/scripts/install-prereqs-ubuntu.sh -y
````

Reload the path (log-out and log-in to make permanent):

````
. ~/.profile
````


### Finalize and test the installation

To start the simulator first change directory to the vehicle directory. For example, for the multicopter code change to ardupilot/ArduCopter:


````
cd ~/ardupilot/ArduCopter
````

Then start the simulator using sim_vehicle.py. The first time you run it you should use the -w option to wipe the virtual EEPROM and load the right default parameters for your vehicle.

````
sim_vehicle.py -w
````

````
sim_vehicle.py --console --map
````

### Updating MAVProxy and pymavlink

New versions of MAVProxy and pymavlink are released quite regularly. If you are a regular SITL user you should update every now and again using this command

````
sudo pip install --upgrade pymavlink MAVProxy --user
````


This concludes the first step SITL Ardupilot installation. 

## SETP 2 - Ardupilot gazebo plugin installation :

(See original instructions here https://github.com/khancyr/ardupilot_gazebo).

### Clone ArduPilot repository

````
cd ~/

git clone https://github.com/r0ch1n/ardupilot_gazebo

cd ardupilot_gazebo

mkdir build

cd build

sudo apt install

sudo apt-get install cmake

cmake ..

# use make without any parameter if running in a VM
make -j4

sudo make install
````

### Set environment variables

````
echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc

````

Set Path of Gazebo Models (Adapt the path to where to clone the repo)
````
echo 'export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models' >> ~/.bashrc
````

Set Path of Gazebo Worlds (Adapt the path to where to clone the repo)
````
echo 'export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:${GAZEBO_RESOURCE_PATH}' >> ~/.bashrc
````

Reload the path (log-out and log-in to make permanent):

````
source ~/.bashrc
````

### Test installation

Open one Terminal and launch SITL Ardupilot
````
sim_vehicle.py -v ArduCopter -f gazebo-iris --map --console
````

Open a second Terminal and launch Gazebo running ardupilot_gazebo plugin
````
gazebo --verbose worlds/iris_arducopter_runway.world
````

You should see a gazebo world with a small quadcopter right at the center


## SETP 3 - Gazebo ROS plugin (roscam) :

This contains the ROS integrated custom models and .world files for Gazebo

### Clone Gazebo roscam integrated mdodel repository

````
# Source ROS

source /opt/ros/noetic/setup.bash

# Clone custom Gazebo ROS package

cd ~/

git clone https://github.com/r0ch1n/ardupilot_gazebo_roscam

cd ardupilot_gazebo_roscam

catkin init

cd src

catkin_create_pkg ardupilot_gazebo

cd ..

catkin build

# Add Custom models and plugin to Gazebo
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo_roscam/src/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gazebo-11/plugins:$GAZEBO_PLUGIN_PATH 
export GAZEBO_PLUGIN_PATH=/opt/ros/noetic/lib:$GAZEBO_PLUGIN_PATH

# Test installation

source ~/ardupilot_gazebo_roscam/devel/setup.bash

roslaunch ardupilot_gazebo iris_with_roscam.launch

````

## SETP 4 - Connect ArduPilot to ROS using MAVROS :

Connect to Ardupilot from ROS (Ardupilot <–> MAVLink <–> ROS )  Original information taken from here http://ardupilot.org/dev/docs/ros-sitl.html
Note - Gazebo is not included in MAVROS so you cannot connect or access any of the Gazebo's Environments.

### Setup MAVROS

New versions of MAVProxy and pymavlink are released quite regularly. If you are a regular SITL user you should update every now and again using this command

````
cd ~/

mkdir -p ardupilot_ws/src

cd ardupilot_ws

catkin init

cd src

mkdir launch

cd launch

roscp mavros apm.launch apm.launch

sudo gedit apm.launch

To connect to SITL we just need to modify line 5 to <arg name="fcu_url" default="udp://127.0.0.1:14551@14555" />. save you file and launch it with
````

### Test

#Make sure the SITL software is running (sim_vehicle.py -v ArduCopter --console --map)
````
cd ~/ardupilot_ws/src/launch

roslaunch apm.launch
````

## Run it all


### Launch Gazebo
Close all teminal first

Open one Terminal and launch ROS integrated Gazebo

````
#Make sure you have all the right environment, if you are not sure run the following first

source /opt/ros/noetic/setup.bash

export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo_roscam/src/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gazebo-11/plugins:$GAZEBO_PLUGIN_PATH 
export GAZEBO_PLUGIN_PATH=/opt/ros/noetic/lib:$GAZEBO_PLUGIN_PATH

#Launch ROS integrated Gazebo

source ~/ardupilot_gazebo_roscam/devel/setup.bash

roslaunch ardupilot_gazebo iris_with_roscam.launch
````

### Launch SITL Ardupilot

Open a second Terminal and launch SITL Ardupilot

````
cd ~/ardupilot/ArduCopter

sim_vehicle.py -f gazebo-iris --console --map
````

### Subscribe to the virtual roscam feed

Open a third Terminal and RTL

````
rqt
````

Select Plugins -> Visualization -> Image View

Then choose /roscam/cam/image_raw

You should see the live feed from inside gazebo


## Final notes and comments

You can use any GCS Adrupilot software to control the UAV.

I hope you can use this basic template for your own projects

