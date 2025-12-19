# Quick Start

## 环境要求

[「RMCS」](https://github.com/Alliance-Algorithm/RMCS) 中提供了 Docker 镜像作为开发和运行环境，参考 [「RMCS/QuickStart」](https://github.com/Alliance-Algorithm/RMCS/wiki/Quick-Start) 配置好开发环境

手动下载依赖：

```sh
# 确保在一个 ROS2 环境中，最好是 Jazzy
sudo apt-get update
sudo apt-get install -y \
net-tools libeigen3-dev ros-$ROS_DISTRO-foxglove-bridge \
ros-$ROS_DISTRO-pcl-ros ros-$ROS_DISTRO-pcl-conversions ros-$ROS_DISTRO-pcl-msgs
```

## 构建项目

首先安装雷达的 SDK（手动安装的情况下，RMCS 镜像已经内置了该 SDK，可以跳过该步骤）

```sh
# 如你所见，Livox 的代码质量一般，仓库代码也不积极维护，需要自己做一些 Work Around
git clone https://github.com/Livox-SDK/Livox-SDK2.git --depth 1 && \
cd Livox-SDK2 && \
sed -i '6iset(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-pragmas -Wno-c++20-compat -include cstdint")' CMakeLists.txt && \
mkdir build && cd build && \
cmake -DCMAKE_BUILD_TYPE=Release .. && \
make -j && \
sudo make install && \
cd ../.. && rm -rf Livox-SDK2
```
随后编译 ROS2 Driver 仓库

1. 安装在标准的 RMCS 目录中
  ```sh
  cd ${RMCS_PATH}/rmcs_ws/src/
  git clone https://github.com/Alliance-Algorithm/livox_ros_driver2
  build-rmcs --packages-select livox_ros_driver2
  ```
2. 安装在自定义 ROS2 工作目录中
  ```sh
  # 进入你的 ROS2 工作目录
  git clone https://github.com/Alliance-Algorithm/livox_ros_driver2.git src/livox_ros_driver2
  # 构建之
  colcon build --packages-select livox_ros_driver2
  ```

## 配置雷达驱动

### 获取雷达 IP

首先从视觉物资或者车上寻找到我们要使用的雷达，在雷达身上贴了一张贴纸，上面有它的 SN 码

雷达 IP 默认规则是：`192.168.1.1{SN码最后两位}`，比如图片上的默认 IP 便是 `192.168.1.174`

<p align="center">
  <img src="https://r2-rmcs.creeper5820.com/doc/IMG20251219073639.jpg" alt="雷达标识贴纸" width="400">
</p>

为了避免和调试的网段冲突，我们往往会将雷达的 IP 调整到另一个子网段中，在 `2024/2025` 赛季，我们将该雷达调整到了 `192.168.100.174`，不出意外的话，以后也是按照这个规则来设定 IP

> 我们可以通过 `Livox Viewer2（Powered By Unity）` 这个软件修改雷达的 IP，这是 [「软件下载页」](https://www.livoxtech.com/cn/downloads)，Linux 上运行可能会有一些问题，尽量使用 Windows
> 
> 由于雷达是静态 IP，需要我们主动将网络设备设置为同一网段：
> ```sh
> # 在本机（非 Docker 容器）上运行
> # 根据实际情况修改 `192.168.100.1/24` 和 `eno1`
> sudo ip addr add 192.168.100.1/24 dev eno1
> ```
> 此时才能与雷达通讯

如果标签掉了，也不用担心，我们可以使用 `nmap` 扫描：

0. 下载工具

```sh
# Debain/Ubuntu
sudo apt-get install nmap

# ArchLinux
sudo pacman -S namp
```

1. 开始扫描

```sh
# 带权限运行可以显示更多的信息
sudo nmap -sn 192.168.100.0/24
```

会有类似输出：

```
⋊> ~ nmap -sn 192.168.100.0/24
Starting Nmap 7.98 ( https://nmap.org ) at 2025-12-19 08:39 +0800
Nmap scan report for 192.168.100.174
Host is up (0.00074s latency).
MAC Address: 58:B8:58:E2:4E:F9 (SZ DJI Technology)
Nmap scan report for 192.168.100.1
Host is up.
Nmap done: 256 IP addresses (2 hosts up) scanned in 6.87 seconds
```

可以看到 `192.168.100.174` 就是激光雷达的 IP

### 配置 `livox_ros_driver2`

首先找到 `livox_ros_driver2/config/MID360_config.json` 这个配置文件，按情况修改两处地方：

```json
{
  "lidar_summary_info": {
    "lidar_type": 8
  },
  "MID360": {
    "lidar_net_info": {
      "cmd_data_port": 56100,
      "push_msg_port": 56200,
      "point_data_port": 56300,
      "imu_data_port": 56400,
      "log_data_port": 56500
    },
    // 第一处：
    "host_net_info": {
      "cmd_data_ip": "192.168.100.2",
      "cmd_data_port": 56101,
      "push_msg_ip": "192.168.100.2",
      "push_msg_port": 56201,
      "point_data_ip": "192.168.100.2",
      "point_data_port": 56301,
      "imu_data_ip": "192.168.100.2",
      "imu_data_port": 56401,
      "log_data_ip": "",
      "log_data_port": 56501
    }
  },
  // 第二处：
  "lidar_configs": [
    // 激光雷达 1
    {
      "ip": "192.168.100.120",
      "pcl_data_type": 1,
      "pattern_mode": 0,
      "extrinsic_parameter": {
        "roll": 0.0,
        "pitch": 0.0,
        "yaw": 0.0,
        "x": 0,
        "y": 0,
        "z": 0
      }
    },
    // 激光雷达 2
    {
      "ip": "192.168.100.174",
      "pcl_data_type": 1,
      "pattern_mode": 0,
      "extrinsic_parameter": {
        "roll": 0.0,
        "pitch": 0.0,
        "yaw": 0.0,
        "x": 0,
        "y": 0,
        "z": 0
      }
    }
  ]
}
```
第一处需要填入运行平台的 IP，使用 `ifconfig`，`nmcli`，`ip ad` 中任意一个指令查看，一般物理网卡设备的名称是 `“eno1“` 这种 en 开头的单词

第二处就需要填入上一个步骤获取的雷达 IP，`lidar_configs` 下每一个雷达就对应一个 `Item`

如果使用多个雷达，还需要在 `livox_ros_driver2/launch/msg_MID360_launch.py` 文件中将`multi_topic` 设置为 1，让不同雷达有不同的 Topic 名字，另外，不要试图使用上面的 `extrinsic_parameter` 选项来将雷达点云信息和 IMU 信息变换到标准坐标系，它只变换点云，Imu 不动，而且 Imu 的内置偏移都没做纠正，这是实际的外参： 

```yaml
imu_extrinsic_translation: [ -0.011, -0.02329, 0.04412 ]
imu_extrinsic_orientation: [ 1., 0., 0., 0. ]
```

### 启动雷达驱动并可视化

首先使用下面的指令启动针对可视化的程序：

```sh
ros2 launch livox_ros_driver2 rviz_MID360_launch.py

# 如果仅仅是想发布消息，不打开 rviz，则使用 msg_MID360_launch
# 它们唯一的区别便是 xfer_format 不同，msg 使用的是 Livox 自定义的消息类型
# 包含更多信息，SLAM 程序就是使用这个消息类型，而 rviz 则是使用 Pointcloud2 类型
# 可以直接被 rviz，foxglove 等可视化
```

启动之后，如果雷达配置正确（能 ping 通一般就说明连上了），就会输出如下日志：

```
$ ros2 launch livox_ros_driver2 rviz_MID360_launch.py
[livox_ros_driver2_node-1] [INFO] [1766174829.964934467] [livox_lidar_publisher]: Livox Ros Driver2 Version: 1.2.4
[livox_ros_driver2_node-1] [INFO] [1766174829.965241363] [livox_lidar_publisher]: Data Source is raw lidar.
[livox_ros_driver2_node-1] [INFO] [1766174829.965255589] [livox_lidar_publisher]: Config file : /workspaces/RMCS/main/RMCS/rmcs_ws/src/livox_ros_driver2/launch/../config/MID360_config.json
[livox_ros_driver2_node-1] LdsLidar *GetInstance
[livox_ros_driver2_node-1] config lidar type: 8
[livox_ros_driver2_node-1] successfully parse base config, counts: 1
[livox_ros_driver2_node-1] [INFO] [1766174829.966597416] [livox_lidar_publisher]: Init lds lidar success!
[livox_ros_driver2_node-1] GetFreeIndex key:livox_lidar_2925832384.
[livox_ros_driver2_node-1] Init queue, real query size:16.
[livox_ros_driver2_node-1] Lidar[0] storage queue size: 10
[livox_ros_driver2_node-1] set pcl data type, handle: 2925832384, data type: 1
[livox_ros_driver2_node-1] set scan pattern, handle: 2925832384, scan pattern: 0
[livox_ros_driver2_node-1] begin to change work mode to 'Normal', handle: 2925832384
[livox_ros_driver2_node-1] successfully set data type, handle: 2925832384, set_bit: 2
[livox_ros_driver2_node-1] successfully set pattern mode, handle: 2925832384, set_bit: 0
[livox_ros_driver2_node-1] successfully set lidar attitude, ip: 192.168.100.174
[livox_ros_driver2_node-1] successfully change work mode, handle: 2925832384
[livox_ros_driver2_node-1] successfully enable Livox Lidar imu, ip: 192.168.100.174
[livox_ros_driver2_node-1] [INFO] [1766174832.966931439] [livox_lidar_publisher]: livox/imu publish use imu format
[livox_ros_driver2_node-1] [INFO] [1766174832.968994037] [livox_lidar_publisher]: livox/lidar publish use PointCloud2 format
```

此时可以使用 `ros2 topic list` 查看相关话题

如果使用 `Nvidia` 显卡在容器中启动的话，`rviz2` 大概率是不能打开的，我们需要使用下面这一句，将渲染方式设置为软件：

```sh
export LIBGL_ALWAYS_SOFTWARE=1
```

不过我们更推荐使用 `foxglove` 查看 ROS2 的 Topic

![](https://r2-rmcs.creeper5820.com/doc/2025-12-20-040824_hyprshot.png)

在运行驱动的环境中运行：

```sh
ros2 launch foxglove_bridge foxglove_bridge_launch.xml port:=8765
```

然后在本机打开即可，这里简要说明，详细的 Foxglove 使用指南请看：[foxglove 使用指南](TODO)

在后续的开发中，我们更常用到的启动脚本是：

```sh
ros2 launch livox_ros_driver2 msg_MID360_launch.py
```

> 小 Tip，建议使用 `Ctrl + \` 来强制关闭程序，正如上述所提到的，Livox 提供的代码质量不尽人意，多线程操作容易导致程序卡死，无法正常响应信号