# FAST-LIO 中文说明

> ROS 2 分支维护者：[Ericsiii](https://github.com/Ericsii)。
>
> 本文翻译本目录现有的 [英文 README](README.md)，保留其主要章节、参考链接和历史示例。标为“当前项目说明”的段落是依据本地源码核对后的补充，不属于上游原文，也不是 ROS 2 官方规范。
>
> 英文原文混有 ROS 1 命令和旧版保存方式，不能全部直接用于当前 Humble 项目。日常操作请看 [项目命令手册](../../../README.md)，输出接口与能力边界请看 [FAST_LIO 专题](../../../docs/FAST_LIO.md)。本文不修改算法、参数或启动流程。
>
> 英文 README 没有完整列出内部计算结果与发布情况。若想知道“算法算出了哪些量、是否写入话题”，可直接阅读本文的 [内部计算结果与发布映射](#project-calculated-results)；该节是本地源码补充，不是原文翻译。
>
> 当前依赖采用稀疏检出，`doc/` 下的图片和论文未出现在本地工作树；本文保留原文资源引用，未为翻译额外下载或展开这些文件。

## 相关工作与扩展应用

**SLAM 相关：**

1. [ikd-Tree](https://github.com/hku-mars/ikd-Tree)：用于三维 k 近邻搜索的先进动态 KD 树。
2. [R2LIVE](https://github.com/hku-mars/r2live)：以 FAST-LIO 为激光雷达与惯性前端的高精度激光雷达、惯性和视觉融合系统。
3. [LI_Init](https://github.com/hku-mars/LiDAR_IMU_Init)：稳健、实时的激光雷达与 IMU 外参初始化和时间同步工具。
4. [FAST-LIO-LOCALIZATION](https://github.com/HViktorTsoi/FAST_LIO_LOCALIZATION)：在 FAST-LIO 上集成重定位功能的扩展项目。

**控制与规划相关：**

1. [IKFOM](https://github.com/hku-mars/IKFoM)：快速、高精度的流形卡尔曼滤波工具箱。
2. [无人机动态障碍物避障](https://github.com/hku-mars/dyn_small_obs_avoidance)：将 FAST-LIO 用于机器人规划的应用之一。
3. [无人车演示](https://www.youtube.com/watch?v=wikgrQbE6Cs)：在可微流形上进行轨迹跟踪的模型预测控制。
4. [Bubble Planner](https://arxiv.org/abs/2202.12177)：利用滚动走廊进行高速、平滑的四旋翼轨迹规划。

当前项目说明：这些是相关项目，不表示当前 FAST-LIO 节点已经包含重定位、规划或控制功能。

## FAST-LIO 简介

**FAST-LIO** 的全称为 Fast LiDAR-Inertial Odometry，即快速激光雷达与惯性里程计。它通过紧耦合的迭代扩展卡尔曼滤波融合激光雷达特征点与 IMU 数据，以应对快速运动、噪声较大、环境杂乱和可能发生退化的场景。

原文列出的主要特点：

1. 使用快速迭代卡尔曼滤波优化里程计。
2. 在大多数稳定环境下自动初始化；这是原文的算法能力描述，不是对任意实物场景的保证。
3. 通过并行 KD 树搜索减少计算开销。

## FAST-LIO 2.0：2021 年 7 月 5 日更新

![FAST-LIO 实验演示](doc/real_experiment2.gif)
![FAST-LIO 运行演示](doc/ulhkwh_fastlio.gif)

**相关视频：**[FAST-LIO2](https://youtu.be/2OvjGnxszf8)、[FAST-LIO1](https://youtu.be/iYCY6T79oNU)。

**处理流程：**

![FAST-LIO2 处理流程](doc/overview_fastlio2.svg)

**新增特性：**

1. 使用 [ikd-Tree](https://github.com/hku-mars/ikd-Tree) 进行增量建图，提高处理速度；原文报告支持超过 100 Hz 的激光雷达处理频率。
2. 直接利用激光雷达点进行当前扫描与地图的配准，可以关闭特征提取，以提高精度。
3. 不要求特征提取，因此支持旋转式雷达，如 Velodyne、Ouster，以及固态雷达，如 Livox Avia、Horizon、MID-70，也可扩展到其他雷达。
4. 支持外部 IMU。
5. 支持 ARM 平台，包括 Khadas VIM3、NVIDIA TX2、Raspberry Pi 4B（8 GB 内存）。

当前项目说明：上述性能是原文描述，不是本台 Jetson 的实测承诺，也不会自动设置驱动的发布频率。本项目组合入口将 MID-360 点云发布频率配置为 10 Hz。“直接利用点”不表示省略时间同步、运动去畸变或降采样。

**相关论文：**

- [FAST-LIO2：快速直接式激光雷达与惯性里程计](doc/Fast_LIO_2.pdf)。
- [FAST-LIO：通过紧耦合迭代卡尔曼滤波实现快速、稳健的激光雷达与惯性里程计](https://arxiv.org/abs/2010.08196)。

**贡献者：**

[Wei Xu 徐威](https://github.com/XW-HKU)、[Yixi Cai 蔡逸熙](https://github.com/Ecstasy-EC)、[Dongjiao He 贺东娇](https://github.com/Joanna-HE)、[Fangcheng Zhu 朱方程](https://github.com/zfc-zfc)、[Jiarong Lin 林家荣](https://github.com/ziv-lin)、[Zheng Liu 刘政](https://github.com/Zale-Liu)、[Borong Yuan](https://github.com/borongyuan)。

## 1. 前置依赖

### 1.1 Ubuntu 与 ROS

原文要求：

- Ubuntu 20.04 或更新版本。
- ROS 2 Foxy 或更新版本，推荐 ROS 2 Humble；参见 [ROS 2 Humble 安装文档](https://docs.ros.org/en/humble/Installation.html)。
- 系统通过 apt 安装的默认 PCL 与 Eigen 通常足以运行 FAST-LIO。

当前项目说明：以上是原 README 的要求概述，不表示任意 Ubuntu 与 ROS 2 版本组合都兼容。当前项目使用 Humble 容器，不应只为阅读本文而在宿主机重新安装依赖。

### 1.2 PCL 与 Eigen

- PCL ≥ 1.8；参见 [PCL 安装说明](https://pointclouds.org/downloads/#linux)。
- Eigen ≥ 3.3.4；参见 [Eigen 官方页面](http://eigen.tuxfamily.org/index.php?title=Main_Page)。

### 1.3 livox_ros_driver2

按照 [livox_ros_driver2 安装说明](https://github.com/Livox-SDK/livox_ros_driver2) 准备驱动。原分支维护者还提供了一个 [使用标准单位的修改分支](https://github.com/Ericsii/livox_ros_driver2/tree/feature/use-standard-unit)。

原文提醒：在构建和运行 FAST-LIO 前，需要安装 Livox 驱动，并加载相应工作空间环境。

原文给出的环境加载示例使用了 `devel/setup.bash`，属于 ROS 1 工作空间形式；其中的 `$Licox_ros_driver_dir$` 是原文占位写法，不是可直接运行的 shell 变量。当前 ROS 2 工作空间应加载实际存在的安装环境，例如：

```bash
source <livox_workspace>/install/setup.bash
```

将 `<livox_workspace>` 替换为实际工作空间目录。当前项目在容器内统一构建和加载工作空间，无需另建驱动工作空间；固定驱动的 IMU 加速度仍为 raw g，不要未经核对切换到另一个单位约定的驱动。

## 2. 构建

原文的独立工作空间构建示例：

```bash
cd <ros2_ws>/src
git clone https://github.com/Ericsii/FAST_LIO.git --recursive
cd ..
rosdep install --from-paths src --ignore-src -y
colcon build --symlink-install
source ./install/setup.bash
```

将 `<ros2_ws>` 替换为实际 ROS 2 工作空间目录；使用 zsh 时加载 `setup.zsh`。

- 构建前应加载已构建的 Livox 驱动环境，参见第 1.3 节。
- 使用自定义构建的 PCL 时，原文建议在 `~/.bashrc` 中设置 `PCL_ROOT`，例如：

```bash
export PCL_ROOT=/path/to/custom/pcl  # 自定义 PCL 安装路径；仅确实需要该版本时设置
```

当前项目说明：本仓库已经固定 FAST-LIO 子模块和依赖版本，不要把上述克隆命令重复执行到现有源码目录。按 [项目命令手册](../../../README.md) 使用已有构建入口即可。

## 3. 直接运行

原文提示：

A. 确保 IMU 与激光雷达的时间同步，这一点非常重要。

B. `Failed to find match for field 'time'.` 警告意味着当前点云字段与预期的逐点时间字段不匹配，原文将其解释为录包中缺少逐点时间。逐点时间对状态传播和运动去畸变十分重要。

C. 已知外参时，原文建议将 `extrinsic_est_en` 设为 `false`。外参初始化可参考 [稳健实时激光雷达与惯性初始化](https://github.com/hku-mars/LiDAR_IMU_Init)。

当前项目说明：本项目当前 `mapping.extrinsic_est_en` 为 `true`。这里翻译上游建议，不修改项目参数；是否关闭在线外参估计应依据实际标定结果。MID-360 的 `CustomMsg` 使用 `offset_time`，不是名为 `time` 的 PointCloud2 字段。

### 3.1 使用 ROS 2 launch 运行

先按 [Livox ROS 驱动说明](https://github.com/Livox-SDK/livox_ros_driver2) 完成连接和实际网络配置，然后在独立工作空间中运行：

```bash
cd <ros2_ws>
source install/setup.bash
ros2 launch fast_lio mapping.launch.py config_file:=avia.yaml  # 示例选择 Avia 参数文件；按实际雷达更换
```

按需要将 `config_file` 改为 `config/` 目录下的其他 YAML 文件。驱动启动示例以 MID-360 为例：

```bash
ros2 launch livox_ros_driver2 msg_MID360_launch.py
```

- 对本版本的 Livox 专用预处理分支，FAST-LIO 使用 `livox_ros_driver2/msg/CustomMsg`，其中保留逐点时间，供运动去畸变使用。
- 原文提到的 `livox_lidar_msg.launch` 和 `livox_lidar.launch` 是历史名称；不要在 ROS 2 中原样照搬。当前项目组合入口固定 `xfer_format=1`，发布 `CustomMsg`。
- 原文建议通过驱动的 `publish_freq` 参数调整点云发布频率。当前项目由 `system_bringup/launch_helpers.py` 的组合驱动设置覆盖，因此修改独立驱动 launch 的同名参数不会改变组合入口的频率。

当前项目说明：独立驱动 launch 使用自己的示例 JSON，执行前必须确认实际 IP。日常使用项目组合入口，不要再重复启动一套驱动或 FAST-LIO。

### 3.2 Livox 雷达配合外部 IMU

原文说明 `mapping_avia.launch` 理论上可支持 MID-70、MID-40 等 Livox 雷达，但需要先在 `config/avia.yaml` 中设置：

1. `common.lid_topic`：激光雷达点云话题。
2. `common.imu_topic`：IMU 话题。
3. `mapping.extrinsic_T`：平移外参。
4. `mapping.extrinsic_R`：旋转外参，只接受旋转矩阵。

外参表示激光雷达在 IMU 坐标系中的位置和朝向，即以 IMU 为基准。对应坐标变换为 `p_IMU = R * p_LiDAR + T`；设备手册可能提供相应信息。

FAST-LIO 提供简单的 Livox 软件时间同步，可通过 `common.time_sync_en=true` 开启。原文强调：仅在确实无法实现外部时间同步时考虑使用，因为它不能保证同步精度。

当前项目说明：`mapping_avia.launch` 是历史文件名，当前本地 ROS 2 入口为 `mapping.launch.py`。当前 MID-360 组合配置使用内置 IMU，默认关闭上述软件自同步；更换 IMU 时需要重新核对时间戳、单位和外参。

### 3.4 保存 PCD 文件

**原文的旧版行为：**在 launch 中将 `pcd_save_enable` 设为 `1`，将世界坐标系中的所有扫描累积，并在 FAST-LIO 结束后保存为 `FAST_LIO/PCD/scans.pcd`。使用 `pcl_viewer scans.pcd` 查看点云。

**当前项目说明：上述自动保存说明不适用于当前本地实现。** 当前 `/Laser_map` 是有界近期点缓存，不是完整环境地图；`/map_save` 手动保存同一缓存的 PCD 快照。保存需要 `pcd_save.pcd_save_en=true`、有效 `map_file_path` 以及非空缓存。组合 launch 根据 `map_path` 是否为空覆盖保存开关，一键入口会传入会话保存路径。`Ctrl+C` 不保证保存。配置和命令见 [FAST_LIO 专题](../../../docs/FAST_LIO.md) 与 [项目命令手册](../../../README.md)。

原文的 `pcl_viewer` 提示：运行查看器时，用数字键切换显示方式：

| 按键 | 原文所述显示方式 |
|---|---|
| `1` | 随机颜色 |
| `2` | 按 X 坐标着色 |
| `3` | 按 Y 坐标着色 |
| `4` | 按 Z 坐标着色 |
| `5` | 按 intensity 着色 |

## 4. Rosbag 示例

### 4.1 Livox Avia 录包

![HKU 室内点云示例](doc/results/HKU_LG_Indoor.png)
![HKU 点云示例](doc/results/HKU_MB_002.png)

从 [Google Drive](https://drive.google.com/drive/folders/1CGYEJ9-wWjr8INyan6q1BZz_5VtGB-fP?usp=sharing) 下载原文示例数据。原文提醒：这些 ROS 1 录包必须先转换为 ROS 2 格式。

原文给出的运行示例：

```bash
ros2 launch fast_lio mapping.launch.py config_path:=<path_to_your_config_file>
ros2 bag play <your_bag_dir>
```

当前项目说明：上面的 `config_path` 用法有歧义。本地 `mapping.launch.py` 将它与 `config_file` 拼接，因此 `config_path` 应是目录，而不是 YAML 文件路径。用 ROS 2 仿真时钟回放时，可按本地 launch 接口明确写为：

```bash
ros2 launch fast_lio mapping.launch.py config_path:=/path/to/config_dir config_file:=avia.yaml use_sim_time:=true
ros2 bag play /path/to/ros2_bag --clock
```

两条命令分别在已加载环境的终端运行。路径和配置文件名为占位示例；时间同步、话题名与 QoS 必须匹配。当前项目已有 `replay` 组合入口，日常回放优先按 [项目命令手册](../../../README.md) 操作，不要同时启动独立与组合两套节点。

### 4.2 Velodyne HDL-32E 录包

**NCLT 数据集：**原始二进制文件见 [数据集页面](http://robots.engin.umich.edu/nclt/)。

原作者提供 [Rosbag 文件](https://drive.google.com/drive/folders/1VBK5idI1oyW0GC_I_Hxh63aqam3nocNK?usp=sharing) 和 [Python 转换脚本](https://drive.google.com/file/d/1leh7DxbHx29DyS1NJkvEfeNJoccxH7XM/view)，脚本调用形式为：

```bash
python3 sensordata_to_rosbag_fastlio.py bin_file_dir bag_name.bag
```

其中 `bin_file_dir` 和 `bag_name.bag` 是原文的输入目录与输出录包占位名称。这些 ROS 1 录包需要转换为 ROS 2；原文引用 [录包版本转换说明](https://ternaris.gitlab.io/rosbags/topics/convert.html)。

原文保留的以下命令属于 ROS 1，不应直接在当前 Humble 容器中执行：

```bash
roslaunch fast_lio mapping_velodyne.launch
rosbag play YOUR_DOWNLOADED.bag
```

使用 ROS 2 时需选择匹配的 launch、Velodyne 预处理配置、话题名、逐点时间字段，并使用转换后的 ROS 2 录包。本文不将历史命令自动替换为未经验证的本机运行方案。

## 5. 无人机上的实现

为验证 FAST-LIO 在真实移动机器人上的稳健性和计算效率，原作者搭建了一架小型四旋翼，搭载视场角为 70° 的 Livox Avia，以及 DJI Manifold 2-C 机载计算机。该计算机配备 1.8 GHz Intel i7-8550U CPU 和 8 GB 内存。

原文描述机体主要结构使用铝或 PLA 材料进行 3D 打印，并表示将来会开源 `.stl` 文件；这是原文当时的说明，本文没有核实其后续发布状态。

![无人机照片](doc/uav01.jpg)
![无人机系统](doc/uav_system.png)

## 6. 致谢

感谢 LOAM（J. Zhang 和 S. Singh 的实时激光里程计与建图工作）、[Livox_Mapping](https://github.com/Livox-SDK/livox_mapping)、[LINS](https://github.com/ChaoqinRobotics/LINS---LiDAR-inertial-SLAM) 和 [Loam_Livox](https://github.com/hku-mars/loam_livox)。

<a id="project-output-interfaces"></a>

## 附录：当前项目可用输出接口

本节不是上游 README 的翻译，而是依据本地 `src/laserMapping.cpp` 与项目 `fastlio_mid360.yaml` 核对的输出说明。完整能力边界仍见 [FAST_LIO 专题](../../../docs/FAST_LIO.md)。不要将本项目的缓存、保存行为推广为所有 FAST-LIO 版本的默认行为。

<a id="project-calculated-results"></a>

### 内部计算结果与发布映射

先从算法实际计算、估计或处理得到的量查起，再看它是否写入 ROS 消息。以下两张表固定使用四列：内部变量或读取方式、意义与单位、对应话题字段、是否写入。这里只列具有物理意义的状态与主要处理结果，不展开雅可比、计时等算法或调试中间量。

字段映射采用“话题 → 消息字段”写法，不是文件路径或新话题。PointCloud2 的每点属性保存在 `data` 中、由 `fields` 描述，因此写为“`data` 中的 `x/y/z/intensity`”，不写不存在的 ROS 字段 `points[].x`。

“是否写入”表示源码中的赋值与转换路径，不保证当前节点正在运行或已经产生消息。状态分为：**已写入、转换后写入、部分写入、未写入、条件写入**；条件写入必须满足相应开关和有效输入。把某个量用于点云转换，不等于把该量本身发布出来；没有字段映射也不表示算法没有使用它。

#### 核心估计结果

前八项由滤波器维护，可通过 `kf.get_x()` 取得；主循环将其复制到 `state_point`。`pos`、`vel` 等三维向量可按索引 0、1、2 读取分量，例如 `state_point.pos(0)`，不能把不同坐标系中的同名分量直接混用。

| 内部变量或读取方式 | 意义与单位 | 对应话题中的字段，或无对应字段 | 是否写入 |
|---|---|---|---|
| `state_point.pos` | body 原点在局部世界系 `camera_init` 中的位置，m | `/Odometry` → `pose.pose.position.x/y/z`；`/path` → `poses[].pose.position.x/y/z`；`/tf` → `transforms[].transform.translation.x/y/z`，对应 `camera_init`→`body` | 已写入 Odometry、TF；轨迹为条件写入，当前 `path_en=true`，约每 10 个处理帧采样一次 |
| `state_point.rot` | body 到局部世界系的旋转，SO(3)，无量纲 | 转为 `geoQuat` 后，`/Odometry` → `pose.pose.orientation.x/y/z/w`；`/path` → `poses[].pose.orientation.x/y/z/w`；`/tf` → `transforms[].transform.rotation.x/y/z/w` | 转换后写入 Odometry、TF；轨迹为条件写入，当前 `path_en=true` |
| `state_point.vel` | body 原点在局部世界系中的线速度，m/s | 无已实现的字段映射；`/Odometry` → `twist.twist.linear.x/y/z` 是预留字段，当前没有填入这个速度估计 | 未写入；预留字段保持默认零，不能用来判断静止 |
| `state_point.bg` | IMU 陀螺仪零偏，IMU 系，rad/s | 无对应话题字段；用于内部角速度修正、预测与去畸变 | 未写入 |
| `state_point.ba` | IMU 加速度计零偏，IMU 系，m/s²，对应归一化后的加速度信号 | 无对应话题字段；用于内部加速度修正、预测与去畸变 | 未写入 |
| `state_point.grav` | 局部世界系中的重力向量，m/s²，以 S² 状态表示方向 | 无对应话题字段；用于内部状态预测与去畸变 | 未写入 |
| `state_point.offset_R_L_I` | LiDAR 到 IMU 的旋转外参，SO(3)，无量纲 | 无对应话题字段；用于 LiDAR 点到 body/世界系的转换，不是 `/Odometry` 的 body 世界姿态 | 未写入外参状态本身；点坐标中体现其作用 |
| `state_point.offset_T_L_I` | LiDAR 原点在 IMU 系中的平移外参，m | 无对应话题字段；用于 LiDAR 点到 body/世界系的转换，不是 `/Odometry` 的 body 世界位置 | 未写入外参状态本身；点坐标中体现其作用 |
| `kf.get_P()` | 滤波器状态误差的协方差矩阵，单位随位置、转角、速度、偏置等矩阵项变化 | `/Odometry` → `pose.covariance`，仅提取并重排位姿相关部分；没有把完整矩阵写入消息 | 部分写入；`twist.covariance` 未填入有效速度协方差 |

计算与读取入口见 [laserMapping.cpp](src/laserMapping.cpp) 的滤波更新、`kf.get_x()` 和 `publish_odometry()`；状态定义见 [use-ikfom.hpp](include/use-ikfom.hpp)。`ba` 的单位依据 [IMU_Processing.hpp](src/IMU_Processing.hpp) 的输入归一化：驱动加速度为 raw g，算法先按初始化重力幅值转换到 m/s² 信号再使用。外参配置提供初值，在线估计开启后内部状态可能不同于 YAML 初值。

#### 主要派生与点云处理结果

这里的局部变量或成员名标明其源码归属，不表示外部节点可以通过公共 API 直接读取。PointCloud2 的映射只指转换函数实际写入的每点主属性，不包含内部逐点时间或有效法向量。

| 内部变量或读取方式 | 意义与单位 | 对应话题中的字段，或无对应字段 | 是否写入 |
|---|---|---|---|
| `geoQuat.x/y/z/w` | 从 `state_point.rot` 取得的 body 世界姿态四元数，无量纲 | `/Odometry` → `pose.pose.orientation.x/y/z/w`；`/path` → `poses[].pose.orientation.x/y/z/w`；`/tf` → `transforms[].transform.rotation.x/y/z/w` | 已写入 Odometry、TF；轨迹为条件写入，当前 `path_en=true` |
| `euler_cur = SO3ToEuler(state_point.rot)` | 从旋转派生的横滚、俯仰、偏航角，°；代码用 57.3 将弧度近似转换为角度 | 无对应话题字段；已发布四元数不等于发布了这个欧拉角向量 | 未写入话题；可能用于条件日志 |
| `pos_lid = state_point.pos + state_point.rot * state_point.offset_T_L_I` | LiDAR 原点在局部世界系中的位置，m，区别于 body 原点位置 | 无对应话题字段；`/Odometry` 的位置写入的是 `state_point.pos`，不是 `pos_lid` | 未写入 |
| `ImuProcess` 内部的 `angvel_last` | 相邻 IMU 样本平均角速度扣除陀螺仪零偏后的结果，IMU 系，rad/s；用于传播与去畸变 | 无已实现的字段映射；`/Odometry` → `twist.twist.angular.x/y/z` 未填入此结果，`/livox/imu` 仍是驱动原始输出 | 未写入；这是 IMU 处理阶段结果，不保证与帧末滤波更新状态同一时刻 |
| `ImuProcess` 内部的 `acc_s_last` | IMU 加速度归一化、扣除零偏、旋转到世界系并加上重力向量后的运动加速度，m/s² | 无对应话题字段；`/livox/imu` 没有被替换成该处理结果 | 未写入；这是 IMU 处理阶段结果，不是直接发布的原始加速度 |
| `feats_undistort` | 完成运动去畸变、表达在帧末 LiDAR 系的点集；坐标 m，强度无量纲 | 转到 body 后写入 `/cloud_registered_body` 的 `data` 中 `x/y/z/intensity`；`dense_publish_en=true` 时也作为 `/cloud_registered` 和近期缓存的源点集 | 条件写入；当前两个 body 输出开关均开启；当前 `dense_publish_en=false`，世界系输出/缓存不选这个密集点集 |
| `feats_down_body` | `feats_undistort` 经体素降采样后的 LiDAR 系点集；坐标 m，强度无量纲；变量名中的 body 不表示它已经是 IMU 系 | 当前转到世界系后，写入 `/cloud_registered`、`/Laser_map` 的 `data` 中 `x/y/z/intensity` | 条件写入；当前 `dense_publish_en=false` 且扫描/地图输出开启；不作为 `/cloud_registered_body` 的输出点集 |
| `feats_down_world` | 降采样点转换到世界系后的内部建图点集；坐标 m，强度无量纲 | 无直接字段映射；`/cloud_registered` 当前从 `feats_down_body` 另行转换生成，不是直接序列化此容器 | 未写入该容器本身；用于内部地图增量更新 |
| `publish_frame_world()` 内的 `laserCloudWorld` | 选定点集转换后的世界系当前帧点云；坐标 m，强度无量纲 | `/cloud_registered` → `data` 中的 `x/y/z/intensity`，由 `fields` 描述布局 | 条件写入；当前 `scan_publish_en=true`，`dense_publish_en=false`，源为降采样点集 |
| `publish_frame_body()` 内的 `laserCloudIMUBody` | 去畸变点集转换后的 IMU/body 系当前帧点云；坐标 m，强度无量纲 | `/cloud_registered_body` → `data` 中的 `x/y/z/intensity` | 条件写入；要求 `scan_publish_en` 和 `scan_bodyframe_pub_en` 均开启，当前均为 `true` |
| `laserCloudOri` | 参与有效配准的 LiDAR 系点集；坐标 m，强度无量纲 | 转到世界系后，写入 `/cloud_effected` → `data` 中的 `x/y/z/intensity` | 条件写入；当前 `effect_map_en=false`，不执行该输出的转换与消息发布 |
| `pcl_wait_pub` | 世界系配准点构成的有界近期缓存；坐标 m，强度无量纲，当前最多 200000 点 | `/Laser_map` → `data` 中的 `x/y/z/intensity`；`/map_save` 可将同一缓存写入 PCD 文件，但服务响应不携带点云 | 条件写入；当前 `map_en=true`，有新缓存且非空时发布；文件保存还需有效路径、保存开关及手动调用 |
| `path.poses` | body 历史采样位姿序列；位置 m、四元数无量纲、每个采样有时间戳 | `/path` → `poses[].header`、`poses[].pose.position.x/y/z`、`poses[].pose.orientation.x/y/z/w` | 条件写入；当前 `path_en=true`，约每 10 个处理帧采样一次，最多 2000 个位姿 |

派生姿态和 LiDAR 位置在 [laserMapping.cpp](src/laserMapping.cpp) 的滤波更新后计算；IMU 修正、去畸变见 [IMU_Processing.hpp](src/IMU_Processing.hpp)，降采样见主循环中的 `downSizeFilterSurf.filter()`。点云字段赋值见 `RGBpointBodyToWorld()`、`RGBpointBodyLidarToIMU()` 与对应发布函数；世界系内部建图点更新见 `map_incremental()`，近期缓存见 `append_recent_map()`，轨迹采样见 `publish_path()`。

当前组合入口的 `lio_consumer` 另外将 Odometry 中已写入的位姿转发到 `/demo_fastlio_cpp/pose` 的 `header`、`pose.position.x/y/z` 和 `pose.orientation.x/y/z/w`。这是项目消费者的转发接口，不是 FAST-LIO 又算出一套位姿，也不会补上未发布的速度。

### 可通过 ROS 订阅或调用的接口

下表的“当前配置”表示项目组合入口使用的参数；实际产生消息还需要节点正常运行、收到有效点云与 IMU，并完成相应处理。回放、参数覆盖和话题重映射可能改变运行时行为。

| 话题或服务 | 消息或服务类型 | 可以使用的数据 | 坐标系与当前配置 |
|---|---|---|---|
| `/Odometry` | `nav_msgs/msg/Odometry` | 当前位置、姿态四元数、位姿协方差；速度字段未填入有效估计 | 位姿为 `camera_init` 中的 `body`；无单独的发布开关 |
| `/cloud_registered` | `sensor_msgs/msg/PointCloud2` | 去畸变并配准后的当前帧环境点，主要使用 `x/y/z/intensity` | `camera_init`；当前开启，`dense_publish_en=false`，使用降采样点集 |
| `/cloud_registered_body` | `sensor_msgs/msg/PointCloud2` | 去畸变后的当前帧环境点，主要使用 `x/y/z/intensity` | `body`；当前开启，使用去畸变点集；同时依赖 `scan_publish_en` |
| `/path` | `nav_msgs/msg/Path` | 历史位姿序列，可查看运动轨迹 | `camera_init`；当前开启，约每 10 个处理帧加入一次，最多 2000 个位姿 |
| `/Laser_map` | `sensor_msgs/msg/PointCloud2` | 多帧配准点构成的有界近期缓存，不是完整全局地图 | `camera_init`；当前开启，最多 200000 个近期点 |
| `/cloud_effected` | `sensor_msgs/msg/PointCloud2` | 参与配准的有效点，用于算法调试 | `camera_init`；当前 `effect_map_en=false`，不发布点云消息 |
| `/tf` | `tf2_msgs/msg/TFMessage` | `camera_init` 到 `body` 的动态坐标变换 | 时间戳与本次 Odometry 一致；可通过 TF 查询使用 |
| `/map_save` | `std_srvs/srv/Trigger` | 请求保存近期点缓存的 PCD 文件；响应包含成功状态与说明文本 | 这是服务，不是点云话题；非空路径、保存开关和有效缓存是必要条件 |

### 数据字段的名称、含义、单位与有效性

这里区分三类名称，避免把“参数”混为一谈：**消息字段**是收到的消息中可读取的数据，例如 `pose.pose.position.x`；**内部状态**是算法估计的变量，例如 `state_point.vel`；**配置参数**是控制运行行为的设置，例如 `publish.scan_publish_en`。标准消息定义说明字段的结构与语义，当前源码决定是否实际赋值，本项目配置决定是否启用对应输出。

下表中的字段路径以一条收到的消息为起点，不是文件路径或新的话题。标为“未填入有效估计”的字段虽然随整条消息发布，但不能当成实测或有效估计值；标为“当前关闭”的话题则不会产生对应点云消息。

#### Odometry：位置、姿态与速度字段

以下字段均属于 `/Odometry`，类型为 `nav_msgs/msg/Odometry`。赋值依据是 [laserMapping.cpp](src/laserMapping.cpp) 的 `set_posestamp()` 与 `publish_odometry()`。

| 消息字段 | 含义 | 单位 / 形状 | 当前赋值与使用限制 |
|---|---|---|---|
| `header.stamp` | 本次状态输出的时间戳，来自当前点云帧结束时间 | `sec` 与 `nanosec` | 已赋值；不是消息到达订阅者的时间 |
| `header.frame_id` | 位姿的参考坐标系 | 字符串 | 已赋值为 `camera_init` |
| `child_frame_id` | 被描述的子坐标系，也是标准消息要求的速度表达坐标系 | 字符串 | 已赋值为 `body` |
| `pose.pose.position.x/y/z` | IMU/body 原点在局部世界系中的三轴位置 | m；3 个分量 | 已赋值，来自 `state_point.pos`；不是机器人几何中心或环境点的位置 |
| `pose.pose.orientation.x/y/z/w` | body 相对于局部世界系的旋转，使用四元数表达 | 无量纲；4 个分量 | 已赋值，来自状态旋转；不是四个欧拉角 |
| `pose.covariance` | 位置与姿态误差的协方差 | 36 个数，行优先 6×6；单位随位置、转角及交叉项变化 | 已由滤波器协方差提取并赋值；顺序为 x、y、z、绕 X/Y/Z 的旋转，不是可信度百分比 |
| `twist.twist.linear.x/y/z` | 标准消息预留的三轴线速度 | m/s；3 个分量 | 未填入有效估计，保持默认零；内部 `vel` 没有写入这里，不能用零值判断静止 |
| `twist.twist.angular.x/y/z` | 标准消息预留的三轴角速度 | rad/s；3 个分量 | 未填入有效估计，保持默认零；没有将内置 IMU 角速度写入这里 |
| `twist.covariance` | 线速度与角速度误差的协方差 | 36 个数，行优先 6×6 | 未填入有效估计，保持默认零；不能解释为速度完全确定 |

世界坐标是本次运行的局部参考系，不是 GPS 坐标；`body` 是 IMU 坐标系，不一定是机器人几何中心。如果未来发布内部世界系线速度，必须先按标准消息约定转换到 `child_frame_id` 指定的表达坐标系。

#### PointCloud2：每个环境点的数据

这里涉及 `/cloud_registered`、`/cloud_registered_body`、`/Laser_map` 和可选的 `/cloud_effected`。前文接口表区分了各话题的坐标系、点集来源和开启状态。点坐标描述环境，不是设备自身的位姿。

`PointCloud2` 并没有直接叫 `points[].x` 的 ROS 字段：各点的数值放在二进制 `data` 中，`fields` 描述每种点属性的名称、类型与偏移。下面的 `x/y/z/intensity` 等是解码后的每点属性名称。

| 字段或每点属性 | 含义 | 单位 / 形状 | 对应话题与当前有效性 |
|---|---|---|---|
| `header.stamp` | 点云时间戳 | `sec` 与 `nanosec` | 四个点云话题均赋值；当前帧话题使用本帧结束时间，`/Laser_map` 使用最近加入缓存帧的结束时间，不是缓存中每个点的采集时间 |
| `header.frame_id` | 所有点坐标的表达坐标系 | 字符串 | `/cloud_registered_body` 为 `body`；其余三个为 `camera_init` |
| 每点 `x/y/z` | 环境点在对应坐标系中的三轴坐标 | m；3 个分量 | 四个点云话题的转换函数均赋值；`/cloud_effected` 当前关闭，其他话题按已开启的配置输出 |
| 每点 `intensity` | 反射强度，来自 Livox 输入点的 `reflectivity` | 无量纲；当前 Livox 链路为 0～255 的值，以浮点数存储 | 四个点云话题均保留该属性；不是点的可信度，也不是 RGB 颜色 |
| 每点 `normal_x/normal_y/normal_z` | PCL `PointXYZINormal` 类型预留的表面法向量分量 | 通常为无量纲的 3 个分量 | 类型包含这些字段，但当前发布转换不计算或填入有效法向量；不能直接使用 |
| 每点 `curvature` | PCL 类型预留的曲率字段 | 当前输出不赋予有效物理含义 | 内部预处理中曾借用它存放逐点时间偏移，单位 ms；发布转换没有复制该时间，不能把输出字段当曲率或逐点时间 |
| `fields`、`point_step`、`row_step`、`is_bigendian` | 二进制点数据的布局说明 | 描述数组、字节数及布尔值 | 由 PCL 转换生成；解析 `data` 时使用，不能按固定内存布局随意读取 |
| `width`、`height`、`data` | 点云尺寸与实际点数据 | 点数、行数与字节数组 | 由 PCL 转换生成；当前输出是非组织点云，`height=1`，`width` 对应本条消息点数 |
| `is_dense` | 消息格式中的无效点标志 | 布尔值 | 由 PCL 转换生成；不是每个点的可信度，不能替代业务侧的有效性检查 |
| 原输入的 `offset_time/tag/line` | 逐点时间偏移、点标记、扫描线编号 | ns、标记或编号 | 在 `/livox/lidar` 的 `CustomMsg` 中存在并供去畸变、点筛选使用，不作为同名字段输出到上述 PointCloud2 话题 |
| 原输入的 `timebase` | Livox 首点基准时间 | ns | 在 `/livox/lidar` 中存在，但当前 FAST-LIO 回调读取 `header.stamp` 作为帧基准时间，不直接读取 `timebase`，也不输出同名 PointCloud2 字段 |

点属性赋值依据是 [laserMapping.cpp](src/laserMapping.cpp) 的 `RGBpointBodyToWorld()`、`RGBpointBodyLidarToIMU()` 与对应发布函数；内部逐点时间处理见 [preprocess.cpp](src/preprocess.cpp)。

#### 轨迹、TF、保存服务与项目位姿转发

| 话题或服务 | 字段路径 | 含义与单位 | 当前赋值与使用限制 |
|---|---|---|---|
| `/path` | `header.frame_id`、`header.stamp` | 轨迹参考系及最新采样位姿时间戳 | 已赋值；参考系为 `camera_init` |
| `/path` | `poses[].header.stamp`、`poses[].header.frame_id` | 每个轨迹采样位姿自身的时间与参考系 | 已赋值；不是每个处理帧都加入轨迹 |
| `/path` | `poses[].pose.position.x/y/z`、`poses[].pose.orientation.x/y/z/w` | 历史位置（m）及姿态四元数（无量纲） | 已赋值；轨迹数组不包含速度或位姿协方差 |
| `/tf` | `transforms[].header.stamp`、`transforms[].header.frame_id`、`transforms[].child_frame_id` | 动态变换的时间、父参考系与子坐标系 | FAST-LIO 填入与 Odometry 一致的时间，以及 `camera_init`、`body`；TF 话题也可能包含其他节点的变换，需按 frame 筛选 |
| `/tf` | `transforms[].transform.translation.x/y/z`、`transforms[].transform.rotation.x/y/z/w` | body 在父参考系中的平移（m）和旋转四元数（无量纲） | 已赋值，来自同一次 Odometry 位姿；不包含速度或协方差 |
| `/map_save` | 响应 `success`、`message` | 保存是否成功（布尔值）及说明文本 | 请求无字段；响应已赋值，点云写入配置路径的 PCD 文件，而不是放在服务响应里 |
| `/demo_fastlio_cpp/pose` | `header`、`pose.position.x/y/z`、`pose.orientation.x/y/z/w` | 时间、参考系、位置（m）和四元数（无量纲） | 项目 `lio_consumer` 从 Odometry 转发；不是 FAST-LIO 本体的发布接口，不包含速度或协方差 |
 
### 输出配置参数与受影响接口

这些才是可调整的运行参数，不是输出数据。当前值来自 [fastlio_mid360.yaml](../system_bringup/config/fastlio_mid360.yaml)，控制逻辑见 `laserMapping.cpp`；这里只解释，不改变现有参数。

| 配置参数 | 当前值 | 用途与约束 | 影响的话题或服务 |
|---|---|---|---|
| `publish.scan_publish_en` | `true` | 当前帧点云输出总开关；关闭不停止位姿估计 | `/cloud_registered`，同时约束 `/cloud_registered_body` |
| `publish.scan_bodyframe_pub_en` | `true` | body 系当前帧输出开关；还要求 `scan_publish_en=true` | `/cloud_registered_body` |
| `publish.dense_publish_en` | `false` | `true` 选去畸变点集，`false` 选体素降采样点集；前者通常点数与带宽更大 | `/cloud_registered`、`/Laser_map` 及其 PCD 快照；不控制 `/cloud_registered_body` 的点集密度 |
| `publish.effect_map_en` | `false` | 有效配准点的调试输出开关，不是地图开关 | `/cloud_effected`；当前关闭消息发布 |
| `publish.map_en` | `true` | 近期点缓存发布开关；保存开启时，即使关闭地图话题也仍维护缓存 | `/Laser_map`；不等于关闭定位算法内部地图 |
| `publish.map_max_points` | `200000` | 近期缓存最大点数，整数 ≥1；增大会增加缓存容量与相关输出/保存开销 | `/Laser_map` 和 `/map_save` 保存的快照；不限制整棵内部 ikd-tree |
| `publish.path_en` | `true` | 轨迹消息发布开关；关闭不停止位姿估计 | `/path` |
| `publish.path_max_poses` | `2000` | 轨迹最多保留的位姿数，整数 ≥1；增大保留更长轨迹但增加消息与存储开销 | `/path`；不限制滤波器状态历史 |
| `pcd_save.pcd_save_en` | YAML 为 `false`，组合 launch 会覆盖 | 允许手动保存；组合 launch 根据 `map_path` 是否非空覆盖此值 | `/map_save`；不代表退出时自动保存 |
| `pcd_save.interval` | `-1` | 保留的上游分段保存参数；当前分段保存逻辑已注释，修改此值无效 | 当前不控制话题或 `/map_save` 的手动保存行为 |
| `map_file_path` | YAML 为空，组合 launch 会覆盖 | PCD 输出文件路径；保存要求非空，由组合 launch 的 `map_path` 指定 | `/map_save` 写入的文件；响应只返回成功状态和文本 |

这些开关均不会把默认零的 `twist` 变成有效速度输出。若需要对外提供内部速度，必须另外实现符合坐标系约定的字段赋值或输出接口。

### 如何核对运行时真正能使用的数据

在启动相应节点、加载工作空间环境的 Humble 容器终端中查询：

```bash
ros2 node info /laser_mapping
ros2 topic list -t
ros2 topic info /Odometry --verbose
ros2 interface show nav_msgs/msg/Odometry
ros2 topic echo /Odometry --once
ros2 topic hz /cloud_registered
ros2 service list -t
```

判断分为三个层次：

1. **文档与源码：**接口设计上提供什么、哪些字段会被赋值、哪些开关控制输出。
2. **ROS 图：**当前存在什么节点、发布器、话题和服务；创建发布器不代表已经产生消息。
3. **实际消息：**当前是否收到数据、时间戳与坐标系是否正确、字段是否具有源码说明的实际意义。

不必为了使用位姿和点云通读滤波算法。先看上述接口，再按需阅读 `publish_odometry()`、`publish_frame_world()`、`publish_frame_body()` 和 `publish_map()`，确认具体字段的生成过程。

## 附录：当前项目的 Odometry 与速度字段

本节是本地源码说明，不属于上游译文。

- `/Odometry` 是 ROS 话题名称，不是文件路径。
- `nav_msgs/msg/Odometry` 是 ROS 提供的标准消息类型，不是 FAST-LIO 自定义的消息。
- `/Odometry.twist` 是“该话题消息里的 twist 字段”的说明写法，不是另一个话题，也不是一个 `.twist` 文件。

消息结构可以概念性地理解为：

```text
/Odometry                 话题，每次发布一条完整的 Odometry 消息
├─ header                 时间戳和位姿的参考坐标系
├─ child_frame_id         被描述的子坐标系，也是 twist 的表达坐标系
├─ pose
│  ├─ pose.position       位置
│  ├─ pose.orientation    姿态四元数
│  └─ covariance          位姿协方差
└─ twist
   ├─ twist.linear        三轴线速度
   ├─ twist.angular       三轴角速度
   └─ covariance          速度协方差
```

在当前 Humble 容器中，可阅读 `/opt/ros/humble/share/nav_msgs/msg/Odometry.msg`，或用原生命令查看定义：

```bash
ros2 interface show nav_msgs/msg/Odometry
ros2 interface show geometry_msgs/msg/TwistWithCovariance
ros2 interface show geometry_msgs/msg/Twist
```

当前 `src/laserMapping.cpp` 的 `publish_odometry()` 为消息填入时间戳、坐标系、位置、姿态和位姿协方差，然后发布完整消息，但没有给 `twist` 填入有效速度估计。因此：

1. 节点正常处理输入时会发布 `/Odometry`，不是缺少一个速度话题的发布动作。
2. 消息仍包含 `twist`；其数值保持默认零，不能据此判断设备真实静止。
3. 内部状态 `state_ikfom.vel` 已包含线速度估计，但当前发布函数没有将它复制到 `twist.twist.linear`。
4. 若以后增加速度输出，必须核对表达坐标系；标准消息要求 twist 在 `child_frame_id` 指定的坐标系中表达，不能直接把世界系速度当成 body 系速度写入。

例如完整字段访问为 `message.pose.pose.position.x` 和 `message.twist.twist.linear.x`。重复的 `pose` 或 `twist` 是因为外层包装还包含协方差，不是拼写错误。

现有消费者示例见 [pose_listener_node.cpp](../lio_consumer/src/pose_listener_node.cpp)，完整接口表与限制见 [FAST_LIO 专题](../../../docs/FAST_LIO.md)。
