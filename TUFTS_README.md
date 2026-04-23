The sections go like this
1. General setup for a new Ava Gen2 robot, that I've noticed

2. SSH Setup and running

3. Cartography setup and running

# On a new Ava Gen2


## Hoyuko sensor

[See specifications](https://www.hokuyo-aut.jp/dl/UST-10LX_Specification.pdf)

The Hoyuko sensor should be on and broadcasting by default. Check first with
```
sudo tcpdump -i enp0s31f6 -n
```
Now join the subnet of the device
```
sudo ip addr add 172.18.0.1/24 dev enp0s31f6
sudo ip link set enp0s31f6 up
```
Should now be able to ping and connect to it
```
ping 172.18.0.10
nc -vz 172.18.0.10 10940
```

Also, this:

In settings, edit under the IPv4 tab. Address = 172.18.0.100, Netmask = 255.255.255.0 (???)

## XTion RGBD cameras
Follow Linux installation at [https://github.com/mgonzs13/ros2_asus_xtion](https://github.com/mgonzs13/ros2_asus_xtion)

In `/opt/ros/humble/share/openni2_camera/launch/camera_With_cloud_launch.py` I specify a device ID (Either #1 or #2).

I run `ros2 launch asus_xtion asus_xtion.launch.py` with `namespace:=<cameranamespace>`.
I run nav2 with the a params file (`/opt/ros/humble/share/nav2_bringup/params/nav2_params.yaml`) with an added `obstacle_layer`.

Note: when rviz-ualizing the pointcloud, turn the expected QoS to BEST_EFFORT.

### Test
`tcpdump` - look for stuff coming from the LIDAR
`ping 172.18.0.10`
`ros2 topic show /scan` (after starting `bringup`)

### Permission over serial port (what's this?)
```
sudo usermod -aG dialout $USER
newgrp dialout
```

## SSH
### Setup
More tweaks:
1. change `systemd` config in `/etc/systemd/logind.conf`
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

2.
```
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'
```
And reboot
```
reboot
```
Not sure which one was it, but the robot would shutdown after extended use. After this it does not.

```
sudo apt install openssh-server
```

### Running
```
ssh user@robot_ip
```
```
ssh hrilab@192.168.0.137 # password = HRIlabrules!
```

## Cartography

### Setup
```
sudo apt install ros-humble-cartographer-ros
sudo apt install ros-humble-navigation2
```

### Running
```
ros2 launch ava_description bringup.launch.py
ros2 launch ava description cartographer.launch.py
ros2 launch spm spm_keyboard_teleop.py.py # (--ros-args -p ...)
```
Optionally
```
ros2 launch ava_description display.launch.py
```
Saving:
```
ros2 run nav2_map_server map_saver_cli -f <name-of-map>
```

## Navigation
### Setup
```
sudo apt install ros-humble-nav2_bringup
```

### Running
```
ros2 launch ava_description bringup.launch.py
ros2 launch ava description cartographer.launch.py # needed?
ros2 launch nav2_bringup bringup_launch.py map:=<name-of-map>.yaml
ros2 launch nav2_bringup rviz_launch.py # then add initial estimate pose, then add a goal position
```

