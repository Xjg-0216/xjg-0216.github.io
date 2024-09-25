
# ROS环境接收和处理Livox激光雷达数据
* 雷达设备： **Livox MID-360**
* SDK链接： [https://github.com/Livox-SDK/Livox-SDK2](https://github.com/Livox-SDK/Livox-SDK2)
* 驱动链接： [https://github.com/Livox-SDK/livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)
* LivoxViewer2： 图形化软件工具（x86）， 主要用于可视化和管理livox激光雷达传感器的采集数据和设备信息， [下载链接](https://www.livoxtech.com/cn/downloads)

>Livox-SDK2:
Livox-SDK2提供了基础的开发接口，用于与Livox激光雷达设备进行通信，数据采集和点云数据处理

>livox_ros_driver2：
livox_ros_driver2是提供的一个ROS驱动程序，主要用于集成和操作livox激光雷达传感器，允许在ROS2环境中接收和处理Livox激光雷达采集点云数据。

livox_ros_driver2依赖于Livox-SDK2， 它利用Livox-SDK2提供的核心功能与Livox激光雷达进行通信，管理设备并接收点云数据，因此**在使用livox_ros_driver2时，需先编译SDK的库文件**

!!! note
    连接方式: 将Livox设备与3588或者PC通过交换机连接，Livox设备默认ip为 192.168.1.1xx， xx是广播码后两位
### 使用livox viewer连接设备

>通过livox viewer可以查看设备的ip, 实时显示激光探测测距仪点云数据

* 1. 将PC与Livox设备直连，同时将PC的IP设备设置为静态IP， IP地址为==192.168.1.2==, 子网掩码==255.255.255.0==，默认网关为==192.168.1.1==
* 2. 下载Livox Viewer并解压文件，启动终端并进行解压缩后文件夹的根目录，运行命令`./livox_viewer.sh`即可启动
* 3. 通过界面可实时 查看点云数据，验证是否正常工作；工具栏中可查看Livox设备的属性及各参数

### 安装Livox-SDK2
```bash
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake ..
make
sudo make install
```

编译安装后默认会在`/usr/local/lib`目录下生成共享库和静态库， 头文件会安装到`/usr/local/include`目录下。
如果要移除Livox SDK2:
```bash
sudo rm -rf /usr/local/lib/liblivox_lidar_sdk_*
sudo rm -rf /usr/local/include/livox_lidar_*
```

### 运行Livox SDK2示例

连接激光雷达，运行可执行文件`livox_lidar_quick_start`, 可以获取到点云数据
```bash
cd build/samples/livox_lidar_quick_start
./livox_lidar_quick_start ../../../samples/livox_lidar_quick_start/mid360_config.json
```

配置文件`mid360_config.json`示例：
```text
{
  "MID360": {
    "lidar_net_info" : {
      "cmd_data_port"  : 56100,
      "push_msg_port"  : 56200,
      "point_data_port": 56300,
      "imu_data_port"  : 56400,
      "log_data_port"  : 56500
    },
    "host_net_info" : [
      {
        "lidar_ip"        : "192.168.1.150",
        "host_ip"   : "192.168.1.56",
        "cmd_data_port"  : 56101,
        "push_msg_port"  : 56201,
        "point_data_port": 56301,
        "imu_data_port"  : 56401,
        "log_data_port"  : 56501
      }
    ]
  }
}
```

### 安装Livox ROS Driver2

ROS版本： `ROS2 Foxy`

#### 克隆Livox ROS Driver2源码
```bash
git clone https://github.com/Livox-SDK/livox_ros_driver2.git ws_livox/src/livox_ros_driver2
```
#### 构建Livox ROS Driver2

```bash
./build.sh ROS2
```
#### 运行Livox ROS Driver2
```bash
source install/setup.sh
cd install/livox_ros_driver2/share/livox_ros_driver2/launch_ROS2
ros2 launch msg_MID360_launch.py
```

py启动文件常用参数设置：
* publish_freq = 10.0  # 点云发布频率
* multi_topic = 0 # 所有激光雷达设备都使用同一个主题发布点云数据
* xfer_format = 0 # 点云格式， 0: PointXYZRTLT

#### 修改配置文件
配置文件路径： `install/livox_ros_driver2/share/livox_ros_driver2/config/MID360_config.json`

配置文件示例：
```
{
  "lidar_summary_info" : {
    "lidar_type": 8
  },
  "MID360": {
    "lidar_net_info" : {
      "cmd_data_port": 56100,
      "push_msg_port": 56200,
      "point_data_port": 56300,
      "imu_data_port": 56400,
      "log_data_port": 56500
    },
    "host_net_info" : {
      "cmd_data_ip" : "192.168.1.56",
      "cmd_data_port": 56101,
      "push_msg_ip": "192.168.1.56",
      "push_msg_port": 56201,
      "point_data_ip": "192.168.1.56",
      "point_data_port": 56301,
      "imu_data_ip" : "192.168.1.56",
      "imu_data_port": 56401,
      "log_data_ip" : "",
      "log_data_port": 56501
    }
  },
  "lidar_configs" : [
    {
      "ip" : "192.168.1.150",
      "pcl_data_type" : 1,
      "pattern_mode" : 0,
      "extrinsic_parameter" : {
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

此外， 还需要将sdk库文件的环境变量`/usr/local/lib`添加到启动中

* 如果添加到当前终端：
```bash
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib
```
* 如果要添加到当前用户：
```bash
vim ~/.bashrc
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib
source ~/.bashrc
```

