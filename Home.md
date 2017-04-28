# Welcome to the IdeaFestival2017 wiki!  
기억력의 한계로.. 나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TK1 flash 방법  
![Jetson TK1](http://kr.nvidia.com/content/tegra/images/jetson/jetson-tk1.png)  
준비사항: Ubuntu 14.04 버젼이 설치된 host PC(인터넷 연결 필요)  
리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 가능하다.  
(단, Jetpack을 이용한 flash는 경험상 비추)  
  
Flash방법은 NVIDIA 공식문서나 [github](https://gist.github.com/jetsonhacks/2717a41f7e60a3405b34)에 정리된 내용을 참고하여 성공하였음. Linux4Tegra(L4T) 최신버젼(R21.5)의 Board Support Package(BSP)와 Sample File System은 [여기](https://developer.nvidia.com/linux-tegra-r215)서 다운로드할 수 있다.  
    
마지막 flash 단계에서 eMMC 16GB 용량을 모두 사용하고 싶으면 '-S 14580MiB' 옵션을 추가할 것.
```
$ sudo ./flash.sh -S 14580MiB jetson-tk1 mmcblk0p1
```  
  
Flash작업이 종료되면 재부팅을 한다. 소스코드 및 개발에 필요한 라이브러리 설치를 위하 저장소를 추가하자.  
  
```
$ sudo apt-add-repository universe
$ sudo apt-add-repository multiverse
$ sudo apt-get update
```  
  
[Note] 처음 설치된 경우 한글입력이 안되는 상태이므로 <http://ledgku.tistory.com/24>를 참고한다.  


## Install CUDA, OpenCV4Tegra, cuDNN  
Jetson TK1 R21.5에서 지원하는 라이브러리 버젼은 다음과 같다.  
```
CUDA 6.5 (6.5.53)
OpenCV4Tegra 2.4 (2.4.13)
cuDNN v2
```
  
R21.4까지는 라이브러리들의 패키지 다운로드 경로를 제공하였는데 최신 버젼부터는 제공을 하지 않는다(구글링을 해보니 NVIDIA에서 Jetpack을 통해 일괄적으로 제공하는 것으로 정책을 변경한 것 같다). 그래도 방법은 다 있다ㅋㅋ  
바로 Jetpack을 통해서 위의 라이브러리 패키지 파일을 다운로드 하는 것인데 안타깝게도 Jetson 보드에서는 Jetpack이 실행되지 않으니 Linux host PC에서 다운로드할 수 밖에 없다. 자세한 방법은 아래 참조..    

먼저 NVIDIA 개발자 사이트의 [Jetpack Archivce](https://developer.nvidia.com/embedded/jetpack-archive)에서 설치하고자 하는 버젼을 선택해서 다운로드한다(2.3.1버젼으로 진행하였음). 자세한 설치 방법은 [공식 설치가이드](http://docs.nvidia.com/jetpack-l4t/index.html#developertools/mobile/jetpack/l4t/2.3/jetpack_l4t_install.htm)를 참고하자.  
  
다운로드 받을 항목을 선택하는 과정에서 필요한 것만 선택할 수 있다(OS는 이미 flash 했으니 다운받을 필요없다)  
단, flash OS 항목을 체크하게 되면 다운로드 및 인스톨 완료후 flash 단계로 넘어가니 'no action'으로 설정하자. 
![](https://cloud.githubusercontent.com/assets/23667624/25509576/f52895c0-2bf4-11e7-846a-4d9b4f00f83c.png)  
  
다운로드를 완료하면 아래 그림과 같이 'jetpack_download'폴더에 라이브러리 패키지 파일(*.deb)들이 생긴 것을 확인할 수 있다. CUDA, OpenCV4Tegra, cuDNN 파일만 USB 등을 이용해서 TK1 보드에 복사하자.
![](https://cloud.githubusercontent.com/assets/23667624/25509577/f75328ec-2bf4-11e7-9d63-91a99c3d32cb.png)
  
이제 다음 명령어로 CUDA ToolKit을 설치하고 라이브러리 및 포함 경로를 설정한다.  
```
$ sudo dpkg -i cuda-repo-l4t-r21.5-6-5-local_6.5-53_armhf.deb
$ sudo apt-get update
$ sudo apt-get install cuda-toolkit-6-5 -y
$ sudo usermod -a -G video $USER(기본설치의 경우 'ubuntu')

$ echo "# Add CUDA bin & library paths:" >> ~/.bashrc
$ echo "export PATH=/usr/local/cuda/bin:$PATH" >> ~/.bashrc
$ echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib:$LD_LIBRARY_PATH" >> ~/.bashrc
$ source ~/.bashrc
```  
  
다음 명령어로 OpenCV4Tegra를 설치한다.  

```
$ sudo dpkg -i libopencv4tegra-repo_2.4.13_armhf_l4t_r21.deb
$ sudo apt-get update
$ sudo apt-get install libopencv4tegra libopencv4tegra-dev
```  
  
설치가 완료된 이후 아래 명령어를 실행하자
```
$ sudo apt-get update  
$ sudo apt-get upgrade
```  

## Install ROS - Indigo  
ARM 버젼의 ROS Indigo [설치가이드](http://wiki.ros.org/indigo/Installation/UbuntuARM)에 따라 순서대로 설치를 진행하고 패키지 설치 단계에서 아래의 목록을 설치한다. (ROS Indigo 버젼에서 설치가능한 패키지 목록은 [여기서](http://repositories.ros.org/status_page/ros_indigo_arm.html) 확인할 수 있다)
```
$ sudo apt-get install ros-indigo-ros-base python-rosdep python-rosinstall ros-indigo-pcl_ros ros-indigo-urdf  
ros-indigo-image-transport ros-indigo-image-transport-plugins ros-indigo-robot-state-publisher
```  
  
ROS visualization을 위한 rqt 및 rviz 패키지도 설치한다(옵션사항)  
```
$sudo apt-get install ros-indigo-rqt* ros-indigo-rviz ros-indigo-robot-model
```
  
만약 rviz 실행시 Segmentation fault 오류가 발생하며 실행이 안될 경우 아래를 수행한다.
```
echo "unset GTK_IM_MODULE" >> ~/.bashrc
source ~/.bashrc
```
  
위의 명령으로도 rviz가 실행이 안되면 [libpcre3_8.35](http://ports.ubuntu.com/pool/main/p/pcre3/libpcre3_8.35-7.1ubuntu1_armhf.deb)을 업데이트 한다.  
```
$ sudo dpkg -i libpcre3_8.35-7.1ubuntu1_armhf.deb
$ sudo apt-get update
```  
 
  
## Install ZED SDK & ROS integration
![ZED - Stereolabs](https://www.stereolabs.com/img/product/ZED_product_main.jpg)  
ZED 스테레오 카메라의 Jetson TK1 SDK-v1.2를 [다운로드](https://www.stereolabs.com/developers/release/1.2/) 한다. Jetson TK1에 대한 공식적인 지원은 v1.2에서 종료(TX1은 지속)되었다. 다운로드한 SDK를 아래 명령으로 설치한다. 
```
$ chmod +x ZED_SDK_JTK1_v*.run
$ ./ZED_SDK_JTK1_v*.run
```  
  

ZED를 ROS와 연동하기 위한 방법은 Stereolabs [공식사이트](https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/)에서 확인할 수 있다.  SDK v1.2와 호환되는 ros-wrapper는 [GitHub](https://github.com/stereolabs/zed-ros-wrapper/releases/tag/v1.2.0)에서 다운로드할 수 있으며 다운로드한 zed-ros-wrapper를 ~/catkin_ws/src 폴더에 복사하고 다음 명령을 수행한다.
```
$ cd ~/catkin_ws
$ catkin_make
$ source ./devel/setup.bash
```  
  
성공적으로 컴파일이 되었다면 아래 명령으로 어플리케이션을 실행시켜 본다.  
```
roslaunch zed_wrapper zed.launch
```  
  
[Tips] 만일 아래와 같은 메시지가 출력된다면 zed firmware가 최신버젼이 아니라는 의미이다.
```
## ZED Firmware version is not up-to-date with current ZED SDK.
## VGA mode will be not consistent
## You have to update the firmware with the latest version (v1142) using the ZED Explorer
```  
  
이 경우에는 메시지에 나와 있듯이 ZED Explorer를 우선 실행시킨다.  
우측 상단의 톱니바퀴 형태의 아이콘을 클릭 > Firmware 탭에서 업데이트 시켜준다.  
단, 확인결과 펌웨어 업데이트는 Jetson TK1에서 할 수 없고 Windows나 Ubuntu같은 데스크탑 환경에서만 가능하다!  
업데이트에 필요한 바이너리 파일(zed_fw_vXXXX_spi.bin)은 ZED SDK가 설치된 아래 경로에서 찾을 수 있다.  
```
/usr/local/zed/firmware
```  


## 참고 사이트  
1. http://myzharbot.robot-home.it/blog/software/configuration-nvidia-jetson-tk1/  
2. https://www.stereolabs.com/blog/index.php/2015/09/24/getting-started-with-jetson-tk1-and-zed/
3. https://huangying-zhan.github.io/2016/08/16/Caffe-installation-and-practice-on-Jetson-TK1.html
4. https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/
5. http://kirumang.tistory.com/27