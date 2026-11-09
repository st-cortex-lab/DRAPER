
# Robot Control Cloth
Authored by Halid Abdulrahim Kadi and Jose Alex Chandy.
Supervised by Luis Figueredo, Kasim Terzić, and Praminda Caleb-Solly.

This repository is for setting up the real-robot control part of the DRAPER project. It supports the operations of ROS1 Noetic for Franka Emika Panda and ROS2 Humble for UR3e robot arms.

## I. Installation

1. Install `ROS2 Humble`

```
sudo apt install ros-humble-moveit
```

2. Install relative packages for `Univeral Robots` 

```
sudo apt install ros-humble-ur-robot-driver ros-humble-ur-description ros-humble-ur-moveit-config
```

3. Install `moveit` python api for humble
```
source /opt/ros/humble/setup.bash
echo "deb [trusted=yes] https://raw.githubusercontent.com/moveit/moveit2_packages/jammy-humble/ ./" | sudo tee /etc/apt/sources.list.d/moveit_moveit2_packages.list
echo "yaml https://raw.githubusercontent.com/moveit/moveit2_packages/jammy-humble/local.yaml humble" | sudo tee /etc/ros/rosdep/sources.list.d/1-moveit_moveit2_packages.list

apt-get update 
Install ros-humble-moveit-py 
apt-get upgrade 
```

4. Install the Python packages, pinned, from [`actoris_harena`](https://github.com/st-cortex-lab/actoris_harena)
   (`main` branch, checked out beside this workspace). The interface imports `actoris_harena`
   itself; ROS 2 stays the apt install above.
```
pip install -r <path-to-actoris_harena>/requirements/draper.txt
pip install -e <path-to-actoris_harena> --no-deps
```

5. Build a workspace
```
mkdir <path-to-ws>/<ws_name>
cd <path-to-ws>/<ws_name>
mkdir src
git clone <this repo>
```

6. Right under `<ws_name>`
```
source /opt/ros/humble/setup.bash
mkdir rcc_build
cd rcc_build
colcon build --base-paths ../
```

## III. Run the Quasi-Static Pick-and-Place Robot Control Program

1. Lauch UR driver
```
source /opt/ros/humble/setup.sh

ros2 launch ur_robot_driver ur_control.launch.py ur_type:=ur3e robot_ip:=192.168.1.15 launch_rviz:=false
```

The above needs to be run first, otherwise the pedant will complain about the connection.

2. Run the `URCaps` program in pedant, and set to `remote control`.

3. Right under `<ws_name>`
```
source /opt/ros/humble/setup.bash
source ./rcc_build/install/setup.sh

ros2 launch rcc_qs_pnp \
    ur_robot_moveit_executable_launch.py \
    ur_type:=ur3e \
    executable:=quasi_static_pick_and_place.py \
    planner:=ompl
```

## IV. Test Individual Components

1. Test ur3e robot with moveit
Right under `<ws_name>`
```
source /opt/ros/humble/setup.bash
source ./rcc_build/install/setup.sh

ros2 launch rcc_qs_pnp ur_robot_moveit_executable_launch.py ur_type:=ur3e executable:=ur3e_robot_moveit.py
```

2. Test active gripper
```
source /opt/ros/humble/setup.bash
source ./rcc_build/install/setup.sh

ros2 run rcc_qs_pnp active_gripper_control.py
```

4. Test whole system
```
source /opt/ros/humble/setup.bash
source ./rcc_build/install/setup.sh

ros2 launch rcc_qs_pnp ur_robot_moveit_executable_launch.py ur_type:=ur3e executable:=test_whole_system.py
```

5. Hand-Eye calibration

```
source /opt/ros/humble/setup.bash
source ./rcc_build/install/setup.sh

ros2 launch rcc_qs_pnp ur_robot_moveit_executable_launch.py ur_type:=ur3e executable:=hand_eye_calibrator.py
```

## V. Build and Run integration with `actoris_harena`
1. Build
```
cd <path-to-actoris_harena>
ACTORIS_HARENA_CONDA_ENV=<the environment from step I.4> . ./env_setup.sh
```

```
cd <path-to-workspace>
mkdir agar_build
cd agar_build
colcon build --packages-select rcc_msgs --base-paths ../
source install/setup.sh
```

2. Run `actoris_harena` policies on the robot

```
cd <path-to-actoris_harena>
ACTORIS_HARENA_CONDA_ENV=<the environment from step I.4> . ./env_setup.sh
```

Go to the workspace
```
source ./agar_build/install/setup.sh
```

a. human_interace

```
cd <path-to-workspace>/src/robot_control_cloth/interface/
python human_interface.py
```

## VI. Video Recording

Video Recording Start
```
ffmpeg -f v4l2 -video_size 1920x1080 -framerate 30 -i /dev/video6 -c:v libx264 -crf 23 -preset medium -bufsize 5M output.mp4
```

Video Recording Stop
```
pkill -SIGINT ffmpeg
```


## V. Run foldsformer

```
python agent_arena_interface.py --agent foldsformer --domain ffmr-square-fabric --adjust_pick --adjust_orien --depth_sim2real v0 --mask_sim2real v2 --task all-corner-inward-folding --config default
```

## VI. Run JA-TN
```
python agent_arena_interface.py --agent transporter --domain sim2real-square-fabric --adjust_pick --adjust_orien --depth_sim2real v2 --mask_sim2real v2 --task all-corner-inward-folding --config MJ-TN-1000-rgb-maskout-rotation-90

```

## VI. Run Diffusion
```
python agent_arena_interface.py --agent diffusion_policy --domain real2sim-towels --adjust_pick --adjust_orien --depth_sim2real v2 --mask_sim2real v2 --task flattening --config masked-rgb

```
