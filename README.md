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

## 配置雷达

首先从视觉物资或者车上寻找到我们要使用的雷达，在雷达身上贴了一张贴纸，上面有它的 SN 码

雷达 IP 默认规则是：`192.168.1.1{SN码最后两位}`，比如图片上的默认 IP 便是 `192.168.1.174`

<p align="center">
  <img src="https://r2-rmcs.creeper5820.com/doc/IMG20251219073639.jpg" alt="雷达标识贴纸" width="400">
</p>

为了避免和调试的网段冲突，我们往往会将雷达的 IP 调整到另一个子网段中，在 2024 赛季，我们将该雷达调整到了 `192.168.100.174`，不出以外的话，以后也是按照这个规则来设定 IP

我们可以通过 `Livox Viewer2` 这个软件修改雷达的 IP，这是 [「软件下载页」](https://www.livoxtech.com/cn/downloads)