# Linux ROS2
--------------------------------------------------

# 1. 安装相关
## 1.1 安装.deb文件
```
sudo dpkg -i package-file-name.deb
```
修复依赖关系:
```
sudo apt install -f
```

## 1.2 便捷启动程序
例如，Groot2软件的安装位置是`~/dev_ws/Groot2/bin/groot2`，每次都需要cd进这个目录，用`./groot2`命令来启动程序。**为了能打开终端后在任何位置都可以用`groot2`命令直接启动这个程序**：
1. 在 `.bashrc` 文件内添加一行 `export PATH="$PATH:~/dev_ws/Groot2/bin"`
2. 终端内运行命令`source ~/.bashrc`


# 2. 功能包
## 2.1 安装依赖

```
cd ~/dev_ws/src
sudo pip install rosdepc
sudo rosdepc init 
rosdepc update
```

```
cd ~/dev_ws
rosdepc install -i --from-path src --rosdistro humble -y
```

## 2.2 创建功能包
```
cd ~/dev_ws/src
ros2 pkg create --build-type ament_cmake package_name    # C++
ros2 pkg create --build-type ament_python package_name   # Python
```

## 2.3 编写程序
```python
rclpy.spin(node)
    # 阻塞型循环，不断查看队列，若队列里有数据则进入回调函数处理
rclpy.spin_once(node)
    # 循环执行一次节点
```

## 2.4 构建功能包

排除某些功能包
```
在功能包目录中放置名为COLCON_IGNORE的空文件
```

构建
```
sudo apt install python3-colcon-ros
source /opt/ros/humble/setup.bash
cd ~/dev_ws
colcon build
```

package.xml储存功能包需要的依赖
编写python功能包时需要在setup.py中

```
entry_points={
    'console_scripts': [
        '节点名 = 功能包名.文件名:函数名',
        ...
```

## 2.5 运行程序
```
ros2 run 功能包名 程序名
```

# 3. 话题
单向\
.msg文件定义数据结构:
```msg
# 例如：
# 通信数据
int32 x
int32 y
```
```
例如uint8[] xxx可表示数组
```

# 4. 服务
节点间的你问我答，双向。服务器唯一，客户端可不唯一\
.srv文件定义数据结构:
```srv
# 例如：
# request结构体 (客户端发送的请求)
int64 a
int64 b
---
# response结构体 (服务器端返回的应答)
int64 sum
```

## 4.1 查看接口数据结构在哪定义
```
ros2 service type /服务名
```

## 4.2 命令行请求服务器
```
ros2 service type /服务名
```


# 5. 通信接口
标准消息类型存放处
```
/opt/ros/humble/share/
```

列出系统内全部接口定义
```
ros2 interface list
```

查看某接口定义
```
# 查看标准定义例如
ros2 interface show sensor_msgs/msg/Image

# 查看功能包自定义例如
ros2 interface show 功能包名/action/xxx
```

查看功能包定义的接口
```
ros2 interface package 功能包名
```

自定义消息类型后需要放入CmakeLists，例如：
```
rosidl_generate_interfaces(${PROJECT_NAME}
    # 功能包目录下的
    "srv/GetObjectPosition.srv"
    ...
    )
```

引用自定义消息类型(python)
```python
from 功能包.其中某个文件夹 import 接口
# 例如：
# 目录结构：learning_interface/srv/GetObjectPosition.srv
# from learning_interface.srv import GetObjectPosition
```

# 6. 动作
完整行为的流程管理，一直有反馈，就像有进度条，可以把控进度

.action文件定义数据结构:
```
#目标
bool enable
---
# 结果
bool finish
---
# 反馈（当前执行到的位置）
int32 state
```

```
ros2 action list
ros2 action info /动作名
ros2 action send_goal /动作名 动作数据结构  "变量: 数值"
# 例如：
# ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 3.14}"
    # 需要周期反馈结尾再加 --feedback
```















--------------------------------------------------
--------------------------------------------------
# Robocup无人机仿真 (ROS1 Melodic)

## 强行关闭gazebo所有进程
```
killall -9 gzclient
killall -9 gzserver
```

## 用键盘控制无人机飞行

```
cd ~/PX4_Firmware
roslaunch px4 indoor1.launch
```

等Gazebo完全启动，新开终端

```
cd ~/XTDrone/communication/
python multirotor_communication.py iris 0
```

新开终端

```
cd ~/XTDrone/control/keyboard
python multirotor_keyboard_control.py iris 1 vel
```

按 I 把向上速度加到0.3以上，按 B 切offboard模式，按 T 解锁。

飞到一定高度后可以切换为‘hover’模式悬停，再利用键盘控制飞机，或运行自己的飞行脚本。

![无人机目录]





---
---

[无人机目录]:data:image/jpeg;base64,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snY8BTjPU9eKq2Xi7Rb65Ftb6gRKchVeJlz9CeK42W0+zeENJhud4h1K8El5n05GD7YArofG1jp9v4ajnghRJbdkMDL1zjjFJPS43GweL32a7oqbjhnO7Hc5GK3tV1bT9GUT6jciINwAFLE/gK5nxE7TSeGnlXLMFLbu5O2nSQpe/E2SG+VZIobdGt0b7obnJ/LFUtvmyW76+R0el63p2tRb9PullVeSpQq35HmlsNSstSSY2U3m+Q+yRcEEN1xzXM6nHFa/EbS30yJVkuflujH3XBOT+OKabiLwr4t1Kafatvdwm7U+rDC4/Sm9NStzdvvEFummapLYzq9zYRkspQkK2MjPrWbpY8b3cVtdz6rpS20yq5QWbbiCM4zuo8OaKlz4bmS+YxPqTM0pXgkZOP0NUtTgv/BX2a8tNTuLyyaVIGtrp92NxCgrj0os07MUrdDtQQR0we+OlFIhDKGGQGAODS0CCiiikMKKKKACiiigAooooAKKKKACiiigAooooAKKKKACloFFABSUtFACUUUUAFFFFABRRRQAUUUUAFRX9pb6pYyWV3EJIZRgj+tS0UAc1beDXs5kYeI9QkhjYbbZgm1QO3TNQ+MH8PLeW51LUG0m7X/V3aYBx6c/4V1dMkigkx51vHNjpvQNijrcDiPCdrFP4vuL+2upr+2jtwhvJsZkbPQY4xzWteeCbea6llstSvNMErbpYrcKVc/iDXRosaDEUSxj0VQKWgdynpul2mkWa2lnGY4wcnHc+pq5z6iiiiwg/GiiigAooooAKKKKACiiigAooooAR8BC7MFVBliewrhNfk03xtqNjZaZP9oeGXdcPHysceD1/Gu9zwR2PWo0ighJ8uBIyepSMDP5UWGirf6bYahp/wDZ89ujQuuEz0HuKxrXwPaLdxyXWrXd/FC2Y4Jgu2P8gK6Xrzj6U7ijqIy9d8P22vRxRSTPC0DBopo8blwc4GeOcVHq/hq31WG2D3M0V1br8l2gG8cY57Vr8dutL81JIDF0Pwxb6NM11JdyX16/DXM+N5HpxxS+IPDNj4mWFL15UEDZBQDLj0rY+tFMFoZ9/pCX1gtotzPalMbZYgM8dOtZdr4Kt476K61DU7vVDFkxpchQqn14A5FdJRTvqAAEdTnjFFFFIAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAFozSUUALmikooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKXNJRQAUUUUAFFFFABRRRQAUUUUAFFFFAH/9k=