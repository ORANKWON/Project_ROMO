# Welcome to the wiki!  
기억력의 한계로.. 나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TX2 flash 방법  
![Jetson TX2](https://news.developer.nvidia.com/wp-content/uploads/2017/03/NVIDIA-Jetson-TX2-Developer-Kit.png)  
준비사항: Ubuntu 14.04 버젼이 설치된 host PC(인터넷 연결 필요, 16.04도 가능하나 Nvidia에서 추천하지 않음) 리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 가능하다.  
  
먼저 NVIDIA 개발자 사이트에서 JetPack 3.1 설치 파일을 [다운로드](https://developer.nvidia.com/embedded/jetpack)한다. 그리고 아래와 같이 실행권한을 설정한 후 실행하면 아래와 같은 화면이 나오는데 여기서 TX2 보드를 선택한다.  
```
$ chmod +x ./JetPack-L4T-3.1-linux-x64.run
$ ./JetPack-L4T-3.1-linux-x64.run
```  
  
![](https://cloud.githubusercontent.com/assets/23667624/26388538/dddc23be-408f-11e7-92f7-813072f17998.png)
  
그리고 다음 그림과 같이 필요한 항목을 선택하여 다운로드한다. OS flash는 JetPack을 통하지 않고 수동으로 진행할 것이므로 생략하도록 한다. 다운로드를 완료하면 'jetpack_download'폴더에 라이브러리 패키지 파일(*.deb)들이 생긴 것을 확인할 수 있다.  
* Root File System
* Drivers
* CUDA Toolkit
* OpenCV 4 Tegra
* cuDNN  
* TensorRT  
  
![](https://cloud.githubusercontent.com/assets/23667624/26388554/f0e458fa-408f-11e7-89b3-d1c5420b2695.png)

Jetson TX2의 flash방법은 [NVIDIA 공식문서](http://developer2.download.nvidia.com/embedded/L4T/r27_Release_v1.0/BSP/l4t_quick_start_guide.txt?2mqXqZYk2lRkqV54f6GeNyhy4RgV9594dHWPQAUAyCjGnRpw6TlhzRpg7OY7eI-bp4AZf-n3gc1x5-SRn0f1DbnSsgdymb93JSA_78ja9w6DJ1Np5VYzeh49E12qJO9W2p7x0GFUfJ0xCDq9FSv1GioO5-RF58lG64c)에 정리된 순서에 따라 진행하면 OK. 마지막 flash 단계에서 eMMC 32GB 용량을 모두 사용하고 싶으면 '-S 28GiB' 옵션을 추가할 것.
```
$ sudo ./flash.sh -S 28GiB jetson-tx2 mmcblk0p1
```  
  
Flash작업이 종료되면 보드는 자동으로 재부팅을 시작하고 우분투 GUI화면이 표시될 것이다. 아래 명령으로 저장소 추가한다.  
```
$ sudo apt-add-repository universe
$ sudo apt-add-repository multiverse
$ sudo apt-add-repository restricted
$ sudo apt-get update
$ sudo apt-get upgrade
``` 
  
[Note] 한글 입력을 사용하고 싶은 경우 <http://hochulshin.com/ubuntu-1604-hangul/>를 참고한다.
```
1. sudo apt-get install fcitx-hangul
2. System Settings > Language Support: install 'Korean'
3. Keyboard input method system:ibus -> fcitx
4. reboot
5. All Settings > Text Entry : Add 'Hangul(fcitx)'
```
  
[Note] Uninstall libreoffice
```
sudo apt purge libreoffice*
```
  
[Note] If you lost desktop menu bar, top bar, launcher and windows borders after Ubuntu upgrade, try this: 
```
$ cd .cache
$ mv compizconfig-1 compizconfig-1_renamed
$ sudo reboot 
```
  
## Build Kernel and ttyACM Module(Jetson TX2)
To support USB ACM devices such as Arduino, Kernel should be rebuild. Follows [Jetnsonhacks](http://www.jetsonhacks.com/2017/07/31/build-kernel-ttyacm-module-nvidia-jetson-tx2/) instruction(Tested with L4T R28.1, JetPack 3.1)

## Install CUDA, OpenCV4Tegra, cuDNN, TensorRT  
Jetson TX2 JetPack 3.1에서 지원하는 라이브러리 버젼은 다음과 같다.  
```
CUDA 8.0 (8.0.84)
OpenCV4Tegra 2.4 (2.4.13)
cuDNN v6.0
TensorRT v2.1
```
    
앞서 다운로드 받은 CUDA, cuDNN, TensorRT 패키지 설치 파일(*.deb)을 USB 등의 저장매체를 이용해 TX2 보드에 복사한다. 다음 명령으로 CUDA ToolKit을 설치하고 라이브러리 및 포함 경로를 설정한다.  
```
$ sudo dpkg -i cuda-repo-l4t-8-0-local_8.0.84-1_arm64.deb 
$ sudo apt-get update
$ sudo apt-get install cuda-toolkit-8-0 -y
$ sudo usermod -a -G video $USER
  
$ echo "# Add CUDA bin & library paths:" >> ~/.bashrc
$ echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc
$ echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH" >> ~/.bashrc
$ source ~/.bashrc
```  
  
CUDA의 설치여부는 아래 명령으로 확인할 수 있다.
```
nvcc -V
```
  
아래와 같이 출력되면 정상적으로 설치된 것이다.
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2016 NVIDIA Corporation
Built on Mon_Mar_20_17:07:33_CDT_2017
Cuda compilation tools, release 8.0, V8.0.72
```
  
cuDNN은 아래와 같이 설정하면 완료.
```
$ sudo dpkg -i libcudnn6_6.0.21-1+cuda8.0_arm64.deb
$ sudo dpkg -i libcudnn6-dev_6.0.21-1+cuda8.0_arm64.deb 
$ sudo dpkg -i libcudnn6-doc_6.0.21-1+cuda8.0_arm64.deb 
$ sudo apt-get update  
```
  
TensorRT를 설치하는 과정은 다음과 같다.([참고](http://docs.nvidia.com/deeplearning/sdk/tensorrt-install-guide/index.html))
```
$ sudo dpkg -i nv-gie-repo-ubuntu1604-ga-cuda8.0-trt2.1-20170614_1-1_arm64.deb
$ sudo apt-get update
$ sudo apt-get install libgie-dev
```
  
OpenCV4Tegra(2.4.13) 설치를 위해서 Nvidia에서 제공하는 스크립트를 활용한다. 스크립트를 [다운로드](https://devtalk.nvidia.com/cmd/default/download-comment-attachment/73046/OpenCV4Tegra)하고 압축을 해제하면 'build_opencv2.4.13.sh' 파일이 나오며 다음 명령으로 설치를 진행할 수 있다.
```
$ ./build_opencv2.4.13.sh <path/you/want/to/install>
```  
  
## Cloning Jetson TX2  
Using the L4T tools, it is possible to create a carbon copy of Jetson TX1 by backing-up and restoring the eMMC partitions. The Jetson must be connected to a remote PC over USB and entered into recovery mode.  
  
### Cloning the image  
cd into the directory containing the L4T installation package on the host PC. The command below will save the TX2's eMMC image to the specified file on the host. 
```
$ sudo ./flash.sh -r -k APP -G system.img jetson-tx2 mmcblk0p1
```
  
In this case, we call the file system.img, so the same flash.sh script can be re-used to format and flash other Jetson's with the image.  
  
### Restoring the image  
The recommended way to restore multiple units with different serial numbers is to save the image above as "system.img" and use the head L4T flashing script, flash.sh, with the -r option (to reuse your backed-up system.img without rebuilding the vanilla image from scratch):  
```
$ sudo ./flash.sh -r jetson-tx2 mmcblk0p1
```
  
## Install ROS - Kinetic  
ARM 버전의 ROS Kinetic [설치가이드](http://wiki.ros.org/kinetic/Installation/Ubuntu)에 따라 순서대로 설치를 진행한다. 패키지
 설치 단계에서 데스크탑용 패키지를 설치한다. (추가로 필요한 패키지가 생길 경우에는 'apt-cache search ros-kinetic' 명령으로 필요한 패키지를 찾아서 설치하면 된다)
```
$ sudo apt-get install ros-kinetic-desktop 
```  
  
* 만일 rosdep init 과정에서 오류가 발생하면 아래 명령을 실행하고 다시 시도해본다.
```
$ sudo c_rehash /etc/ssl/certs
```

Catkin workspace 생성을 위해서 다음을 실행한다.
```
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws
$ catkin_make
```
    
## Install ZED SDK & ROS integration
![ZED - Stereolabs](https://www.stereolabs.com/img/product/ZED_product_main.jpg)  
ZED 스테레오 카메라의 Jetson TX2 SDK-v2.0.1를 [다운로드](https://www.stereolabs.com/developers/release/2.0/#sdkdownloads_anchor) 한다. 
```
$ chmod +x ZED_SDK_JTX2_v*.run
$ ./ZED_SDK_JTX2_v*.run
```  
  
ZED 스테레오 카메라는 USB 3.0으로 통신하므로 TX2 보드가 USB 3.0포트를 사용할 수 있도록 수정한다. 
```
$ sudo sed -i 's/usb_port_owner_info=0/usb_port_owner_info=2/' /boot/extlinux/extlinux.conf
```
  
L4T는 기본적으로 에너지 절약모드로 동작하도록 되어 있는데 특히 USB 포트의 경우에는 사용을 하지 않으면 대기모드로 빠지게 된다. 이렇게 되면 연결되어 있는 센서들과의 통신이 제대로 이루어지지 않으므로 autosuspend를 막는 스크립트를 추가하도록 한다. 터미널을 열고 아래 명령으로 'disableUSBautosuspend.sh' 파일을 생성한다.
```
$ sudo gedit /usr/local/bin/disableUSBAutosuspend.sh
```
  
파일이 열리면 아래의 내용을 추가후 저장하고 닫는다.
```
#!/bin/sh
sudo sh -c 'for dev in /sys/bus/usb/devices/*/power/autosuspend; do echo -1 >$dev; done'
```

ZED 스테레오 카메라의 성능을 최대로 끌어내기 위해서 ZED SDK에서 제공하는 스크립트를 부팅때 자동으로 실행시킨다. 추가로 위에서 작성한 쉘스크립트를 실행하도록 하는 내용도 추가하자. 편집기로 /etc/rc.local 파일을 열고 다음과 같이 입력을 하면 된다. (파일을 열어서 제일 하단의 'exit 0' 바로 위에 입력한다)
```
# Turn up the CPU and GPU for max performance
/usr/local/zed/scripts/jetson_max_l4t_updated.sh

# disable USB autosuspend
/usr/local/bin/disableUSBAutosuspend.sh
```
  
수정한 파일에 실행권한을 부여하자.
```
$ sudo chmod +x /etc/rc.local
```
  
ZED를 ROS와 연동하기 위한 방법은 Stereolabs [공식매뉴얼](https://www.stereolabs.com/documentation/guides/using-zed-with-ros/introduction.html)에서 확인할 수 있다.  SDK v2.0.1와 호환되는 ros-wrapper는 [GitHub](https://github.com/stereolabs/zed-ros-wrapper)에서 다운로드할 수 있다.
```
$ cd ~/catkin_ws/src
$ git clone  https://github.com/stereolabs/zed-ros-wrapper.git
$ cd ..
$ catkin_make
$ $ source ./devel/setup.bash
``` 

다음 명령으로 ZED 카메라의 데이터를 ROS 토픽으로 브로드캐스트 할 수 있다.  
```
roslaunch zed_wrapper zed.launch
```  
  
[Tips] 만일 아래와 같은 메시지가 출력된다면 zed firmware가 최신버젼이 아니라는 의미이다.
```
## ZED Firmware version is not up-to-date with current ZED SDK.
## VGA mode will be not consistent
## You have to update the firmware with the latest version (v1142) using the ZED Explorer
```  
  
이 경우에는 메시지에 나와 있듯이 ZED Explorer를 우선 실행시키고 우측 상단의 톱니바퀴 형태의 아이콘을 클릭 > Firmware 탭에서 업데이트 시켜준다. (확인결과 펌웨어 업데이트는 Jetson 보드상에서는 할 수 없고 Windows나 Ubuntu같은 데스크탑 환경에서만 가능하다) 업데이트에 필요한 바이너리 파일(zed_fw_vXXXX_spi.bin)은 ZED SDK가 설치된 아래 경로에서 찾을 수 있다.  
```
/usr/local/zed/firmware
```  
  
## Install Sweep LiDAR SDK & Obstacle detection package & ROS Integration 
![SWEEP LiDAR](http://scanse.io/sites/default/files/media/image/Sweep%20Packaging.jpg)
  
Sweep LiDAR는 [Scanse](http://scanse.io/)사에서 개발한 초소형 360도 레이저스캐너이다. C/C++ SDK 뿐만 아니라 아두이노 및 ROS용 드라이버도 제공하고 있다. ROS와 연동하기 전에 먼저 아래 방법으로 SDK 및 드라이버를 설치한다.
```
$ git clone https://github.com/scanse/sweep-sdk
$ cd sweep-sdk/libsweep
$ mkdir -p build
$ cd build

$ cmake .. -DCMAKE_BUILD_TYPE=Release
$ cmake --build .
$ sudo cmake --build . --target install
$ sudo ldconfig
```
  
ROS와 연동하기 위해 필요한 의존성 패키지(pointcloud_to_laserscan 패키지는 소스빌드) 및 wrapper 노드를 설치한다.
```
$ sudo apt-get install ros-kinetic-tf2-sensor-msgs

$ cd ~/catkin_ws/src
$ git clone https://github.com/ros-perception/pointcloud_to_laserscan.git
$ git clone https://github.com/scanse/sweep-ros.git
$ cd ..
$ catkin_make
$ source ./devel/setup.bash
```

다음 launch파일을 실행하면 rviz상에서 sweep lidar 데이터를 확인할 수 있다. 퍼블리시되는 토픽은 laserscan 타입과 pointcloud2 타입중에서 선택 가능하다(아래는 pointcloud2 타입).
```
$ roslaunch sweep_ros view_sweep_pc2.launch
```
  
2D LiDAR based [obstacle_detector](https://github.com/tysik/obstacle_detector) package is used to detect and track object with laser scanner. [Armadillo C++](http://arma.sourceforge.net/download.html) library is required. 
```
$ cd ~
$ tar -xvf armadillo-7.960.1.tar.xz
$ cd armadillo-7.960.1
$ mkdir build && cd build
$ cmake ..
$ make && sudo make install
```
  
Cloning 'obstacle_detector' package and compile.
```
$ cd catkin_ws/src
$ git clone https://github.com/tysik/obstacle_detector.git
$ cd ..
$ catkin_make --pkg obstacle_detector
$ source devel/setup.bash
```
  
## Install MoveIt with ROS-kinetic
로봇팔 제어와 시뮬레이션을 위해 'MoveIt' 소프트웨어를 사용한다. 먼저 아래 명령으로 kinetic 버젼용 moveit 패키지를 설치한다.
```
$ sudo apt-get install ros-kinetic-moveit*
$ sudo apt-get dist-upgrade
```
  
Install dependencies(clang is optional)
```
$ sudo apt-get install python-wstool python-catkin-tools clang-format-3.8
```
  
다이나믹셀 서보모터를 활용한 로봇팔 제어를 위해 아래 패키지를 설치한다.
```
$ sudo apt-get install ros-kinetic-industrial-core
$ sudo apt-get install ros-kinetic-dynamixel-sdk
$ sudo apt-get install ros-kinetic-robotis-math
$ sudo apt-get install ros-kinetic-qt-build
$ sudo apt-get install ros-kinetic-gazebo-ros-control
$ sudo apt-get install ros-kinetic-gazebo-ros-pkgs
$ sudo apt-get install ros-kinetic-ros-control
$ sudo apt-get install ros-kinetic-ros-controllers
```
  
[xacro](http://wiki.ros.org/xacro)를 이용한 urdf 모델 생성
```
$ rosrun xacro xacro robot_model.xacro > robot_model.urdf
```
  
[Note] MoveIt 0.7.3+ 버젼에서 “Invalid Trajectory: start point deviates from current robot state more than …“ 에러가 발생하는 경우 trajectory_execution.launch.xml파일을 찾아서 allowed_start_tolerance를 0으로 수정하도록 한다.
```
<!-- Allowed joint-value tolerance for validation that trajectory's first point matches current robot state -->
<param name="trajectory_execution/allowed_start_tolerance" value="0.0"/> <!-- default 0.01 -->
```
    
## TensorRT on Jetson TX1/TX2
텐서플로우는 구글에서 개발한 딥러닝용 프레임워크이다. Jetson TX1/TX2용으로 TensorRT를 제공하고 있으며 아래 과정으로 소스 다운로드 받고 빌드하면 된다.  
```
$ git clone http://github.com/dusty-nv/jetson-inference
$ cd jetson-inference
$ mkdir build
$ cd build
$ cmake ../
```
  
Compile the project..
```
$ cd jetson-inference/build	
$ make
``` 
  
Using zed camera image topic for detectnet-camera
```
```
  
## Arduino-Jetson TX2 communication via rosserial
rosserial for arduino 패키지를 설치한다.
```
$ sudo apt-get install ros-kinetic-rosserial-arduino ros-kinetic-rosserial
```
  
Run the rosserial python node that forwards your Arduino messages to the rest of ROS. Make sure to use the correct serial port and baud rate.
```
$ rosrun rosserial_python serial_node.py _port:=/dev/ttyACM0 _baud:=115200
```
  
You can find and test 'blink' example as described [ros wiki](http://wiki.ros.org/rosserial_arduino/Tutorials/Blink)
```
/*
 * rosserial Subscriber Example
 * Blinks an LED on callback
 */

#include <ros.h>
#include <std_msgs/Empty.h>

ros::NodeHandle nh;

void messageCb( const std_msgs::Empty& toggle_msg){
  digitalWrite(13, HIGH-digitalRead(13));   // blink the led
}

ros::Subscriber<std_msgs::Empty> sub("toggle_led", &messageCb );

void setup()
{
  pinMode(13, OUTPUT);
  nh.initNode();
  nh.subscribe(sub);
}
 
void loop()
{
  nh.spinOnce();
  delay(1);
}
```
  
## Turtlebot - Mapping and Navigation
```
$ sudo apt-get install ros-kinetic-turtlebot-bringup
$ sudo apt-get install ros-kinetic-turtlebot-navigation 
$ sudo apt-get install ros-kinetic-rtabmap-ros
```

## Run roslaunch at startup on Ubuntu 16.04
 - Case of: "Sweep LiDAR"
```
$ sudo gedit /etc/systemd/system/run-sweep-lidar.service
```
  
Make service unit as follow:
```
[Unit]
Restart=on-abort
After=mysql.service

[Service]
ExecStart=/usr/local/bin/run-sweep-lidar.sh

[Install]
WantedBy=default.target
```
  
Create shell script
```
$ sudo gedit /usr/local/bin/run-sweep-lidar.sh
```
  
Write shell script as follow:
```
#!/usr/bin/env bash
bash -c "source /home/nvidia/catkin_ws/devel/setup.bash && roslaunch sweep_ros sweep2scan.launch"
```
  
We need to make our script executable. Also, install systemd service unit and enable it so it will be executed at the boot time:
```
$ sudo chmod 744 /usr/local/bin/run-sweep-lidar.sh
$ sudo chmod 664 /etc/systemd/system/run-sweep-lidar.service
$ sudo systemctl daemon-reload
$ sudo systemctl enable run-sweep-lidar.service
```
  
## USB Symbolic link
리눅스에서 여러 대의 usb장치를 사용할 경우 ttyUSBx의 링크 번호가 매번 변경이 되는데 이를 강제적으로 고정하기 위해서 심볼릭 링크를 사용할 수 있는데 아래 세가지 정보를 확인해야 한다.  
```
1. Vendor ID
2. Product ID
3. Serial Number
```
  
우선 lsusb 명령을 통해 VendorID와 ProductID를 확인한다. lsusb 명령을 통해 아래와 같은 출력을 얻었다면, VendorID는 0403, ProductID는 6001이 된다.
```
$ lsusb
Bus 001 Device 013: ID 0403:6001 Future Technology Devices International, Ltd FT232 USB-Serial (UART) IC
```
  
Serial Number를 확인하기 위해서 아래 명령을 입력한다. 여기서는 12345678이 Serial Number가 된다.
```
$ udevadm info -a /dev/ttyUSB0 | grep '{serial}'
   ATTRS{serial}=="12345678"
```
  
리눅스 udev에서 사용할 rules를 추가하기 위해서 아래와 같이 에디터를 연다.
```
$ sudo gedit /etc/udev/rules.d/99-usb-serial.rules
```
  
아래 내용을 입력한다. 단, SYMLINK는 본인이 원하는 이름으로 설정하면 된다.
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="12345678", MODE="0666", SYMLINK+="ttyMyUSB"
```
  
아래 명령으로 규칙을 설정한다.
```
$ sudo udevadm control --reload-rules
```
  
정상적으로 적용되었는지 확인하기 위해서 USB장치를 뽑은 후 다시 연결하고 확인한다.
```
$ ls /dev/tty* -al
lrwxrwxrwx 1 root   root          7 Oct  8 10:32 /dev/ttyMyUSB -> ttyUSB0
crw-rw-rw- 1 root   dialout 188,  0 Oct  8 10:32 /dev/ttyUSB0
```
  
# Qt with Jetson TX2  
You can configure cross compile environment for Jetson TX2 from [this](https://www.innovative-dsp.com/ftp/jetson/Configure%20Cross%20Compiling%20for%20Nvidia%20Jetson%20TX2%20on%20Ubuntu%2064%20bit.pdf). Also, we can directly install Qt creator on Jetson TX2(tested with Qt-5.9.2 & Qt Creator 4.4.1).  
  
1. Download Qt 5.9.2 source from [here](http://download.qt.io/official_releases/qt/5.9/5.9.2/single/qt-everywhere-opensource-src-5.9.2.tar.xz).
2. Following compile steps as described [here](http://doc.qt.io/qt-5/linux-building.html).
3. Download Qt Creator source from [here](http://download.qt.io/official_releases/qtcreator/4.4/4.4.1/qt-creator-opensource-src-4.4.1.tar.gz).
4. Before compile, modify sources as pointed [here](https://devtalk.nvidia.com/default/topic/988568/qt-5-7-or-later-on-tx1-local-or-cross-/). You should modify 2 files, src/libs/3rdparty/botan/botan.cpp and botan.h 
5. Modify "botan.cpp" file to restrict link to X86 architectures as follow :
```
#if defined(BOTAN_TARGET_CPU_IS_X86_FAMILY)
  #include <cpuid.h> // cpuid.h does not exist under ARM64
#endif
```
  
# CAN Communication with Jetpack 3.1(R28.1)
You can load CAN bus driver in two ways, manually or automatically. You can enable CAN bus driver using following command.
```
$ sudo modprobe mttcan
$ sudo ip link set can0 up type can bitrate 500000
$ sudo ifconfig can0 up
```
  
if you want to enable CAN bus driver automatically, comment the "blacklist mttcan" in /etc/modprobe.d/blacklist-mttcan.conf. Steps described [here](https://devtalk.nvidia.com/default/topic/1025010/how-to-use-can-on-jetson-tx2-/?offset=2) can be followed to achieve proper CAN communication.  
  
For a quick setup please check the below J26 pin description on Jetson TX1/TX2.
![jetson-j26-pinout](https://user-images.githubusercontent.com/23667624/32313155-4d5e57e2-bf99-11e7-92be-8f5d0ca79acf.jpg)  
  
## 참고 사이트  
1. http://qiita.com/kndt84/items/a32d07350ad8184ea25e
2. https://devtalk.nvidia.com/default/topic/974063/jetson-tx1/caffe-failed-with-py-faster-rcnn-demo-py-on-tx1/
3. https://github.com/rbgirshick/py-faster-rcnn/issues/237
4. https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/
5. http://kirumang.tistory.com/27
6. https://github.com/ros-visualization/rviz/issues/955
7. http://researchmap.jp/jojgwe7tm-2001408/
8. https://github.com/rbgirshick/py-faster-rcnn/issues/236 (py-faster-rcnn 빌드 오류 해결)
9. https://lastone9182.github.io/2016/09/16/aws-caffe.html
10. https://devtalk.nvidia.com/default/topic/974063/jetson-tx1/caffe-failed-with-py-faster-rcnn-demo-py-on-tx1/ (Jetson tx1: py-faster-rcnn용 caffe 빌드 오류 해결)
11. https://github.com/rbgirshick/py-faster-rcnn/issues/155 (caffe test build 오류 해결)
12. https://myurasov.github.io/2016/11/27/ssd-tx1.html (Jetson TX1용 SSD 설치 및 실행)
13. https://github.com/rbgirshick/fast-rcnn/issues/52 (caffe boost 빌드 오류, Makefile.config 수정) 
14. https://diyprojects.io/install-arduino-ide-linux-arm-raspberry-pi-orange-pi-odroid/
15. https://forum.arduino.cc/index.php?topic=400808.0 (ARM ubuntu 16.04에 arduino ide 설치 방법)
16. https://github.com/headmelted/codebuilds/issues/15
17. http://docs.ros.org/kinetic/api/moveit_tutorials/html (ROS-kinetic Moveit 문서)
18. https://github.com/dusty-nv/jetson-inference#building-from-source-on-jetson
19. https://devtalk.nvidia.com/default/topic/1000106/jetson-tx2/opencv-convertto-failure/post/5171055/#5171055 (opencv4tegra 2.4.13 install on TX2)
20. https://askubuntu.com/questions/825354/unity-ubuntu-16-04-no-menu-bar-launcher-top-bar-dash-window-borders-disapp(unity - Ubuntu 16.04 no menu bar, launcher, top bar, dash, window borders disappeared)
21. https://answers.ros.org/question/245089/systemd-roslaunch/ (auto start roslaunch at boot) 
22. http://neurorobotictech.com/Products/Jetduino/ArduinoDueonJetsonTK1.aspx (Arduino IDE on Jetson)
23. http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html (Java Kit 8)
24. http://moveit.ros.org/moveit!/ros/2017/01/03/firstIndigoRelease.html (MoveIt error)
25. http://hintshop.ludvig.co.nz/show/persistent-names-usb-serial-devices (Persistent names for usb-serial devices)
26. https://devtalk.nvidia.com/default/topic/988568/qt-5-7-or-later-on-tx1-local-or-cross-/ (qt install issue)
27. https://devtalk.nvidia.com/default/topic/1025010/how-to-use-can-on-jetson-tx2-/?offset=2 (CAN setup)
28. https://code.google.com/archive/p/slide-33/downloads (Dynamixel library for arduino)
29. https://github.com/ipa-mdl/ros_canopen/tree/feature/can-filter (ros_canopen filter