持续更新中~~~  
有问题欢迎在Issues中反馈，欢迎大家一起加入学习。


若启动了conda环境先把环境关闭

```bash 
conda deactivate
```


### 安装环境（宿主机host中执行）

安装ros2_control :

https://github.com/DDTRobot/TITA_ROS2_Control_Sim.git

**或者直接使用docker，已配置好tensor\webots\ros2\ros2-control环境**

https://github.com/DDTRobot/webots2023b_ros2_docker

按照以下命令行拉取编译代码  

```bash 
#本仓库
git clone https://github.com/DDTRobot/sim2sim2real.git

cd sim2sim2real

#其他仓库
vcs import < sim2sim2real.repos
```

编译之前先修改这个文件：src/tita_locomotion/tita_controllers/tita_controller/src/fsm/FSMState_RL.cpp
![alt text](/pictures/image.png)
把位置修改为把推理出来的model_gn.engine路径。 **如果是在docker下运行，请将.engine的路径修改为docker中的路径，而不是宿主机的路径！！（一般为/mnt/dev/*.engine）**


### 运行 (docker和 host都可以)

```bash
#编译
source /opt/ros/humble/setup.bash && colcon build --packages-up-to locomotion_bringup webots_bridge template_ros2_controller tita_controller joy_controller keyboard_controller

source install/setup.bash 

#启动webots仿真
source /opt/ros/humble/setup.bash && source install/setup.bash && ros2 launch locomotion_bringup sim_bringup.launch.py

#启动键盘指令输入终端
source /opt/ros/humble/setup.bash && source install/setup.bash && ros2 run keyboard_controller keyboard_controller_node --ros-args -r __ns:=/tita

#如果webots遇到不显示机器人mesh的情况:
sudo mkdir -p /usr/share/robot_description
sudo cp -r src/tita_locomotion/tita_description/tita /usr/share/robot_description/

```

### 实机部分

```bash 
#拷贝文件进入机器
scp -r 你的路径/tita_ros2/src robot@192.168.42.1:~/tita_ros2/

#连接机器：
ssh robot@192.168.42.1
#密码：
apollo

#关掉自启的ros2
systemctl stop tita-bringup.service

cd tita_ros2/

source /opt/ros/humble/setup.bash

#编译
colcon build --packages-up-to locomotion_bringup template_ros2_controller tita_controller joy_controller keyboard_controller hw_broadcaster

source install/setup.bash 

#连接遥控器
crsf-app -bind

#原则上需要重新在机器人中重新推理生成.engine，当前版本镜像中不包含tensorrt的dev，因此需要重新安装
sudo apt install nvidia-cuda-dev
sudo apt install tensorrt-dev
sudo apt install tensorrt
#安装完毕后对你的onnx做重新推理：
/usr/src/tensorrt/bin/trtexec --onnx=test.onnx --saveEngine=model_gn.engine

#分别运行
nohup ros2 launch locomotion_bringup hw_bringup.launch.py ctrl_mode:=wbc &
nohup ros2 launch joy_controller joy_controller.launch.py &
```

如果您的tensorrt是10.x版本，参考[这里](https://github.com/DDTRobot/tita_rl_sim2sim2real/issues/1)
