<div align="center">

# CARLA-Autoware-Bridge - Enables the use of CARLA for Testing and Development of Autoware Core/Universe
[![Linux](https://img.shields.io/badge/os-ubuntu22.04-blue.svg)](https://www.linux.org/)
[![ROS2humble](https://img.shields.io/badge/ros2-humble-blue.svg)](https://docs.ros.org/en/humble/index.html)

</div>

## Installation
```bash
# Pull carla_autoware_bridge
git clone https://github.com/HoYongLee98/carla_autoware_bridge

# Build
cd carla_autoware_bridge
source /opt/ros/humble/setup.bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

#### Local Workflow
Comming Soon. Until then, take a look at our Dockerfile.

#### Maps
Autoware needs the maps in a special lanelet2 format, we will upload all converted maps in the future under the following link: [carla-autoware-bridge/maps](https://syncandshare.lrz.de/getlink/fiBgYSNkmsmRB28meoX3gZ/)

### 1) CARLA
We recommended to use the dockerized version of CARLA 0.9.15. To pull and start CARLA for usage with the CARLA-Autoware-Bridge follow the steps below.
```bash
# Start CARLA
./CarlaUE4.sh -prefernvidia -quality-level=Low -RenderOffScreen
```
Additional information:
- `-prefernvidia` - use NVIDIA GPU for hardware acceleration
- `-carla-rpc-port=3000` - use other than default port (2000) for RPC service's port
- `-quality-level=Low` - use low quality level mode for a minimal video memory consumption

### 2) CARLA-Autoware-Bridge
Run the carla-autoware-bridge
```bash
# Launch the bridge
ros2 launch carla_autoware_bridge carla_aw_bridge.launch.py
```
Additional information:
- `-port:=3000` to switch to a different CARLA port for it's RPC port
- `-timeout:=10` to increase waiting time of loading a CARLA town before raising error
- `-view:=true` to show third-person-view window
- `-town:=Town10HD` to set the town
- `-traffic_manager=False` to turn off traffic manager server (True by default)
- `-tm_port=8000` to switch the traffic manager server port to a different one (8000 by default)

If you want to spawn traffic run the following script inside the docker:
```
python3 src/carla_autoware_bridge/utils/generate_traffic.py -p 1403
```

### 3) Autoware
To use Autoware some minor [adjustments](/doc/autoware-changes.md) are required. Additionally you will need our sensorkit and vehicle model.
```
git clone https://github.com/TUMFTM/Carla_t2.git
```

Launch autoware
```bash
ros2 launch autoware_launch e2e_simulator.launch.xml vehicle_model:=carla_t2_vehicle sensor_model:=carla_t2_sensor_kit map_path:=<path to /wsp/map>
```

Autoware changes often, for a reproducible behaviour we recommend you to use a tagged autoware version:
https://github.com/autowarefoundation/autoware/tree/2024.01

```bash
docker pull ghcr.io/autowarefoundation/autoware:humble-2024.01-cuda-amd64

rocker --network=host -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp -e LIBGL_ALWAYS_SOFTWARE=1 --x11 --nvidia --volume /path/to/code -- ghcr.io/autowarefoundation/autoware-universe:humble-2024.01-cuda-amd64
```
