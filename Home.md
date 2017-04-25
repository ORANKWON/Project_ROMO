# Welcome to the IdeaFestival2017 wiki!  
기억력의 한계로..나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TK1 flash 방법  
![Jetson TK1](http://kr.nvidia.com/content/tegra/images/jetson/jetson-tk1.png)  
준비사항: Ubuntu 14.04 버젼이 설치된 host PC(인터넷 연결 필요)  
리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 flash 가능..  
단, Jetpack을 이용한 설치는 경험상 비추.. [github에 정리된 방법](https://gist.github.com/jetsonhacks/2717a41f7e60a3405b34)을 참고하여 성공하였음.  
마지막 flash 단계에서 eMMC 16GB 용량을 모두 사용하고 싶으면 '-S 14580MiB' 옵션을 추가할 것  

```
sudo ./flash.sh -S 14580MiB jetson-tk1 mmcblk0p1
```  
  
Flash작업이 종료되면 재부팅을 한다. 소스코드 및 개발에 필요한 라이브러리 설치를 위하 저장소를 추가하자.  
  
```
sudo apt-add-repository universe
sudo apt-add-repository multiverse
sudo apt-get update
```  

## Install CUDA 6.5 & OpenCV4Tegra
CUDA 패키지 및 OpenCV4Tegra(2.4.10)를 [NVIDIA 개발자 사이트](https://developer.nvidia.com/linux-tegra-rel-21)에서 다운로드 한후에 [설치가이드](https://huangying-zhan.github.io/2016/08/16/Caffe-installation-and-practice-on-Jetson-TK1.html)를 따라 설치한다.  
  
다음 명령어를 이용하여 CUDA ToolKit을 설치하고 라이브러리 및 포함 경로를 설정한다.  

```
$ mkdir /home/ubuntu/pkg4caffe
$ cd /home/ubuntu/pkg4caffe

$ sudo dpkg -i cuda-repo-l4t-r21.2-6-5-prod_6.5-34_armhf.deb
$ sudo apt-get update
$ sudo apt-get install cuda-toolkit-6-5
$ sudo usermod -a -G video $USER

$ echo "# Add CUDA bin & library paths:" >> ~/.bashrc
$ echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc
$ echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH" >> ~/.bashrc
$ source ~/.bashrc
```  
  
다음 명령어를 이용하여 OpenCV4Tegra를 설치한다.  

```
$ cd /home/ubuntu/Desktop/pkg4caffe

$ sudo dpkg -i libopencv4tegra-repo_l4t-r21_2.4.10.1_armhf.deb
$ sudo apt-get update
$ sudo apt-get install libopencv4tegra libopencv4tegra-dev
```  
  
설치가 완료된 이후 아래 명령어를 실행
```
$sudo apt-get update  
$sudo apt-get upgrade
```  

## Install ROS - Indigo  
ROS Indigo [설치가이드](http://wiki.ros.org/indigo/Installation/UbuntuARM)의 순서대로 설치를 진행하고 패키지 설치 단계에서 아래의 목록을 설치한다.  
(ROS Indigo 버젼에서 설치가능한 패키지 목록은 [여기서](http://repositories.ros.org/status_page/ros_indigo_arm.html) 확인할 수 있다.)
```
$sudo apt-get install ros-indigo-ros-base python-rosdep python-rosinstall ros-indigo-pcl_ros ros-indigo-urdf  
ros-indigo-image-transport ros-indigo-image-transport-plugins
```  
  
## Install ZED SDK  
![ZED - Stereolabs](https://www.stereolabs.com/img/product/ZED_product_main.jpg)  
ZED 스테레오 카메라 SDK-v1.2를 [다운로드](https://www.stereolabs.com/developers/release/1.2/) 한다. Jetson TK1에 대한 공식적인 지원은 v1.2에서 종료(TX1은 지속)되었다.  
ZED를 ROS와 연동하기 위한 방법은 Stereolabs [공식사이트](https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/)에서 확인할 수 있다.  
다운로드 받은 ZED-ROS-wrapper를 ~/catkin_ws/src 폴더에 복사한다. CMakeLists.txt 파일을 열어 다음 부분을 찾는다.  
```
find_package(ZED 2.0 REQUIRED) 

##For Jetson, OpenCV4Tegra is based on OpenCV2.4
exec_program(uname ARGS -p OUTPUT_VARIABLE CMAKE_SYSTEM_NAME2)
if ( CMAKE_SYSTEM_NAME2 MATCHES "aarch64" ) # Jetson TX1
    SET(OCV_VERSION "2.4")
    SET(CUDA_VERSION "8.0")
    SET(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
else() # Ubuntu Desktop
    SET(OCV_VERSION "3.1")
    SET(CUDA_VERSION "8.0")
endif()
```  
  
아래와 같이 수정한다.  
```  
find_package(ZED 1.2 REQUIRED) # Jetson TK1

##For Jetson, OpenCV4Tegra is based on OpenCV2.4
exec_program(uname ARGS -p OUTPUT_VARIABLE CMAKE_SYSTEM_NAME2)
if ( CMAKE_SYSTEM_NAME2 MATCHES "aarch64" ) # Jetson TX1
    SET(OCV_VERSION "2.4")
    SET(CUDA_VERSION "8.0")
    SET(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
elseif ( CMAKE_SYSTEM_NAME2 MATCHES "armv7l" ) # Jetson TK1
    SET(OCV_VERSION "2.4")
    SET(CUDA_VERSION "6.5")
    SET(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
else() # Ubuntu Desktop
    SET(OCV_VERSION "3.1")
    SET(CUDA_VERSION "8.0")
endif()
```
  
## 참고 사이트  
1. http://myzharbot.robot-home.it/blog/software/configuration-nvidia-jetson-tk1/  
2. https://www.stereolabs.com/blog/index.php/2015/09/24/getting-started-with-jetson-tk1-and-zed/