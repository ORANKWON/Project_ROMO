# Welcome to the IdeaFestival2017 wiki!  
기억력의 한계로..나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TK1 flash 방법  
![Jetson TK1](http://kr.nvidia.com/content/tegra/images/jetson/jetson-tk1.png)  
준비사항: Ubuntu 14.04 버젼이 설치된 host PC(인터넷 연결 필요)  
리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 flash 가능..  
단, Jetpack을 이용한 설치는 경험상 비추.. [github에 정리된 방법](https://gist.github.com/jetsonhacks/2717a41f7e60a3405b34)을 참고하여 성공하였음.  
마지막 flash 단계에서 eMMC 16GB 용량을 모두 사용하고 싶으면 '-S 14580MiB' 옵션을 추가할 것  

```
$ sudo ./flash.sh -S 14580MiB jetson-tk1 mmcblk0p1
```  
  
Flash작업이 종료되면 재부팅을 한다. 소스코드 및 개발에 필요한 라이브러리 설치를 위하 저장소를 추가하자.  
  
```
$ sudo apt-add-repository universe
$ sudo apt-add-repository multiverse
$ sudo apt-get update
```  

## Install CUDA 6.5 & OpenCV4Tegra
CUDA 패키지 및 OpenCV4Tegra(2.4.10)를 [NVIDIA 개발자 사이트](https://developer.nvidia.com/linux-tegra-rel-21)에서 다운로드 한다.  
다음 명령어로 CUDA ToolKit을 설치하고 라이브러리 및 포함 경로를 설정한다.  

```
$ sudo dpkg -i cuda-repo-l4t-r21.2-6-5-prod_6.5-34_armhf.deb
$ sudo apt-get update
$ sudo apt-get install cuda-toolkit-6-5
$ sudo usermod -a -G video $USER

$ echo "# Add CUDA bin & library paths:" >> ~/.bashrc
$ echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc
$ echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH" >> ~/.bashrc
$ source ~/.bashrc
```  
  
다음 명령어로 OpenCV4Tegra를 설치한다.  

```
$ sudo dpkg -i libopencv4tegra-repo_l4t-r21_2.4.10.1_armhf.deb
$ sudo apt-get update
$ sudo apt-get install libopencv4tegra libopencv4tegra-dev
```  
  
설치가 완료된 이후 아래 명령어를 실행
```
$ sudo apt-get update  
$ sudo apt-get upgrade
```  

## Install ROS - Indigo  
ARM-ROS Indigo [설치가이드](http://wiki.ros.org/indigo/Installation/UbuntuARM)의 순서대로 설치를 진행하고 패키지 설치 단계에서 아래의 목록을 설치한다.  
(ROS Indigo 버젼에서 설치가능한 패키지 목록은 [여기서](http://repositories.ros.org/status_page/ros_indigo_arm.html) 확인할 수 있다.)
```
$ sudo apt-get install ros-indigo-ros-base python-rosdep python-rosinstall ros-indigo-pcl_ros ros-indigo-urdf  
ros-indigo-image-transport ros-indigo-image-transport-plugins
```  
  
## Install ZED SDK & ROS integration
![ZED - Stereolabs](https://www.stereolabs.com/img/product/ZED_product_main.jpg)  
ZED 스테레오 카메라 SDK-v1.2를 [다운로드](https://github.com/stereolabs/zed-ros-wrapper/releases/tag/v1.2.0) 한다. Jetson TK1에 대한 공식적인 지원은 v1.2에서 종료(TX1은 지속)되었다. 다운로드한 SDK를 아래 명령으로 설치한다. 
```
$ chmod +x ZED_SDK_JTK1_v*.run
$ ./ZED_SDK_JTK1_v*.run
```  
  

ZED를 ROS와 연동하기 위한 방법은 Stereolabs [공식사이트](https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/)에서 확인할 수 있다.  
다운로드한 zed-ros-wrapper를 ~/catkin_ws/src 폴더에 복사하고 다음 명령을 수행한다.
```
$ cd ~/catkin_ws
$ catkin_make
$ source ./devel/setup.bash
```
  
## 참고 사이트  
1. http://myzharbot.robot-home.it/blog/software/configuration-nvidia-jetson-tk1/  
2. https://www.stereolabs.com/blog/index.php/2015/09/24/getting-started-with-jetson-tk1-and-zed/
3. https://huangying-zhan.github.io/2016/08/16/Caffe-installation-and-practice-on-Jetson-TK1.html
4. https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/