# Welcome to the IdeaFestival2017 wiki!  
기억력의 한계로.. 나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TX2 flash 방법  
![Jetson TX2](https://news.developer.nvidia.com/wp-content/uploads/2017/03/NVIDIA-Jetson-TX2-Developer-Kit.png)  
준비사항: Ubuntu 14.04 or 16.04 버젼이 설치된 host PC(인터넷 연결 필요)  
리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 가능하다.  
  
먼저 NVIDIA 개발자 사이트에서 JetPack 3.0 설치 파일을 [다운로드](https://developer.nvidia.com/embedded/jetpack)한다. 그리고 아래와 같이 실행권한을 설정한 후 실행하면 아래와 같은 화면이 나오는데 여기서 TX2 보드를 선택한다.  
```
$ chmod +x ./JetPack-L4T-3.0-linux-x64.run
$ ./JetPack-L4T-3.0-linux-x64.run
```  
  
![](https://cloud.githubusercontent.com/assets/23667624/26388538/dddc23be-408f-11e7-92f7-813072f17998.png)
  
그리고 다음 그림과 같이 필요한 항목을 선택하여 다운로드한다. OS flash는 JetPack을 통하지 않고 수동으로 진행할 것이므로 생략하도록 한다. 다운로드를 완료하면 'jetpack_download'폴더에 라이브러리 패키지 파일(*.deb)들이 생긴 것을 확인할 수 있다.  
* Root File System
* Drivers
* CUDA Toolkit
* OpenCV 4 Tegra
* cuDNN  
![](https://cloud.githubusercontent.com/assets/23667624/26388554/f0e458fa-408f-11e7-89b3-d1c5420b2695.png)

Jetson TX2의 flash방법은 [NVIDIA 공식문서](http://developer2.download.nvidia.com/embedded/L4T/r27_Release_v1.0/BSP/l4t_quick_start_guide.txt?2mqXqZYk2lRkqV54f6GeNyhy4RgV9594dHWPQAUAyCjGnRpw6TlhzRpg7OY7eI-bp4AZf-n3gc1x5-SRn0f1DbnSsgdymb93JSA_78ja9w6DJ1Np5VYzeh49E12qJO9W2p7x0GFUfJ0xCDq9FSv1GioO5-RF58lG64c)에 정리된 순서에 따라 진행하면 OK. 마지막 flash 단계에서 eMMC 32GB 용량을 모두 사용하고 싶으면 '-S 28GiB' 옵션을 추가할 것.
```
$ sudo ./flash.sh -S 28GiB jetson-tx2 mmcblk0p1
```  
  
Flash작업이 종료되면 보드는 자동으로 재부팅을 시작하고 우분투 GUI화면이 표시될 것이다. 아래 명령으로 저장소 추가한다.  
```
$ sudo apt-add-repository universe
$ sudo apt-add-repository multiverse
$ sudo apt-get update
$ sudo apt-get upgrade
``` 
  
위에서 다운로드 받은 CUDA, OpenCV4Tegra, cuDNN 패키지 설치 파일(*.deb)을 USB 등을 이용해서 TX2 보드에 복사하고,   

[Note] 처음 설치된 경우 한글입력이 안되는 상태이므로 <http://ledgku.tistory.com/24>를 참고한다.  
  
## Install CUDA, OpenCV4Tegra, cuDNN  
Jetson TX2 JetPack 3.0에서 지원하는 라이브러리 버젼은 다음과 같다.  
```
CUDA 8.0 (8.0.33)
OpenCV4Tegra 2.4 (2.4.13)
cuDNN v5.1
```
    
이제 다음 명령으로 CUDA ToolKit을 설치하고 라이브러리 및 포함 경로를 설정한다.  
```
$ sudo dpkg -i cuda-repo-l4t-8-0-local_8.0.34-1_arm64.deb 
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
Built on Fri_Jul_15_14:52:12_CDT_2016
Cuda compilation tools, release 8.0, V8.0.33
```
  
다음 명령어로 OpenCV4Tegra를 설치한다.  
```
$ sudo dpkg -i libopencv4tegra-repo_2.4.13-17-g5317135_arm64_l4t-r24.deb
$ sudo apt-get update
$ sudo apt-get install libopencv4tegra libopencv4tegra-dev
```  
  
cuDNN은 별도의 패키지 다운로드 및 설치 과정이 없다. 다운로드한 파일의 압축을 풀고 라이브러리 및 헤더 파일이 포함된 폴더(cudnn-6.5-linux-ARMv7-v2)로 이동후에 CUDA가 설치된 폴더에 복사해두면 편리하다. 그 이유는 CUDA 설치 과정에서 LD_LIBRARY_PATH를 설정하기 때문이다.
```
$ cd cudnn-6.5-linux-ARMv7-v2
$ sudo cp cudnn.h /usr/local/cuda-6.5/include
$ sudo cp libcudnn* /usr/local/cuda-6.5/lib
```
    
설치가 완료된 이후 아래 명령어를 실행하자
```
$ sudo apt-get update  
$ sudo apt-get upgrade
```  

## Install ROS - Indigo  
ARM 버젼의 ROS Indigo [설치가이드](http://wiki.ros.org/indigo/Installation/UbuntuARM)에 따라 순서대로 설치를 진행하고 ros-indigo-desktop 패키지를 설치한다. (ROS Indigo 버젼에서 설치가능한 패키지 목록은 [여기서](http://repositories.ros.org/status_page/ros_indigo_arm.html) 확인할 수 있다)
```
$ sudo apt-get install ros-indigo-desktop ros-indigo-pcl-ros
```  
  
Jetson TK1보드에서 rviz를 실행하기 위해 아래 옵션을 설정한다.
```
$ echo "# for using RVIZ" >> ~/.bashrc
$ echo "unset GTK_IM_MODULE" >> ~/.bashrc
$ source ~/.bashrc
```
  
만약 위와 같이 설정했음에도 rviz 실행시 Segmentation Fault 메시지가 발생한다면 rviz가 업데이트 되면서 libpcre 라이브러리와 충돌하는 문제이므로 아래 패키지를 다운로드하여 업데이트 한다.  
```
$ wget http://launchpadlibrarian.net/182261128/libpcre3_8.35-3ubuntu1_armhf.deb
$ wget http://launchpadlibrarian.net/182261132/libpcre3-dev_8.35-3ubuntu1_armhf.deb
$ wget http://launchpadlibrarian.net/182261135/libpcre3-dbg_8.35-3ubuntu1_armhf.deb
$ wget http://launchpadlibrarian.net/182261131/libpcrecpp0_8.35-3ubuntu1_armhf.deb
$ sudo dpkg -i libpcre*8.35*.deb
$ sudo apt-get update
```  
 
  
## Install ZED SDK & ROS integration
![ZED - Stereolabs](https://www.stereolabs.com/img/product/ZED_product_main.jpg)  
ZED 스테레오 카메라의 Jetson TX2 SDK-v2.0.1를 [다운로드](https://www.stereolabs.com/developers/release/2.0/#sdkdownloads_anchor) 한다. 
```
$ chmod +x ZED_SDK_JTX2_v*.run
$ ./ZED_SDK_JTX2_v*.run
```  
  
ZED 스테레오 카메라는 리소스를 많이 사용하 따라서 TK1 보드의 성능을 최대치로 개방(?)하기 위해서 'maxPerformance.sh'이라는 파일을 생성한다. (여기서는 gedit를 사용하였지만 아무 편집기(vim 등)를 사용해도 무방하다)
```
$ sudo gedit /usr/local/bin/maxPerformance.sh
```
  
파일이 열리면 아래 내용을 입력하고 저장후 닫는다.
```
#!/bin/sh

# Set CPU to full performance on NVIDIA Jetson TK1 Development Kit
if [ $(id -u) != 0 ]; then
echo "This script requires root permissions"
echo "$ sudo "$0""
exit
fi

# To obtain full performance on the CPU (eg: for performance measurements or benchmarking or when you don't care about power draw), you can disable CPU scaling and force the 4 main CPU cores to always run at max performance until reboot:
echo 0 > /sys/devices/system/cpu/cpuquiet/tegra_cpuquiet/enable
echo 1 > /sys/devices/system/cpu/cpu0/online
echo 1 > /sys/devices/system/cpu/cpu1/online
echo 1 > /sys/devices/system/cpu/cpu2/online
echo 1 > /sys/devices/system/cpu/cpu3/online
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Clock the GPUs to max speed
echo 852000000 > /sys/kernel/debug/clock/override.gbus/rate
echo 1 > /sys/kernel/debug/clock/override.gbus/state
```
  
TK1 보드가 부팅후에 위의 스크립트가 실행되도록 하기 위해서 /etc/rc.local 파일에 다음 내용을 추가한다. (파일을 열어서 제일 하단의 'exit 0' 바로 위에 입력한다)
```
# Turn up the CPU and GPU for max performance
/usr/local/bin/maxPerformance.sh
```
  
ZED 스테레오 카메라는 USB 3.0으로 통신하므로 TK1 보드가 USB 3.0포트를 사용할 수 있도록 아래 명령을 수행한다. (편집기로 /boot/extlinux/extlinux.conf 파일을 직접 열고 수정해도 무방하다)
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
  
부팅후 스크립트가 자동적으로 실행하기 위해서 /etc/rc.local 파일의 제일 하단('exit 0' 바로 위)에 아래 내용을 입력한다.
```
# disable USB autosuspend
/usr/local/bin/disableUSBAutosuspend.sh
```
  
ZED를 ROS와 연동하기 위한 방법은 Stereolabs [공식사이트](https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/)에서 확인할 수 있다.  SDK v1.2와 호환되는 ros-wrapper는 [GitHub](https://github.com/stereolabs/zed-ros-wrapper/releases/tag/v1.2.0)에서 다운로드할 수 있으며, 다운로드한 zed-ros-wrapper를 ~/catkin_ws/src 폴더에 복사하고 다음 명령을 수행한다.
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
  
이 경우에는 메시지에 나와 있듯이 ZED Explorer를 우선 실행시키고 우측 상단의 톱니바퀴 형태의 아이콘을 클릭 > Firmware 탭에서 업데이트 시켜준다. (확인결과 펌웨어 업데이트는 Jetson TK1에서 할 수 없고 Windows나 Ubuntu같은 데스크탑 환경에서만 가능하다) 업데이트에 필요한 바이너리 파일(zed_fw_vXXXX_spi.bin)은 ZED SDK가 설치된 아래 경로에서 찾을 수 있다.  
```
/usr/local/zed/firmware
```  

## Install Fast-RCNN with Caffe & pyCaffe support
Jetson TK1에 설치하기가 제일 까다로웠던 부분이다. 아래 설명한 순서대로 진행하는 것을 추천한다.  
  
Caffe는 Berkeley 대학에서 관리하고 있는 딥러닝 라이브러리이며 google의 tensorflow와 함께 사용자층이 매우 두텁다. C++로 직접 구현할 수도 있고 Python과 Matlab 인터페이스도 잘 구현되어 있다. 먼저 아래와 같이 Caffe 구동에 필요한 라이브러리들을 설치한다.
```
$ sudo apt-get install libprotobuf-dev protobuf-compiler gfortran libboost-dev cmake 
libleveldb-dev libsnappy-dev libboost-thread-dev libboost-system-dev  
libatlas-base-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev  
liblmdb-dev gcc-4.7 g++-4.7 libboost-all-dev  
```

다음은 py-faster-rcnn을 설치하는 과정이다.  
  
1. Jetson TK1용으로 수정된 faster-rcnn을 git clone한다.
```
$ git clone https://github.com/joeking11829/py-faster-rcnn-tk1.git
```
  
2.편의상 폴더명을 이후부터 $FRCN 이라 표시한다.  
  
3. Cython을 빌드한다.  
```
$ cd $FRCN/lib
$ make
```
  
4. $FRCN내의 caffe 폴더로 이동한다.
```
$ cd $FRCN/caffe-fast-rcnn
```
  
5. gcc 4.6버젼을 설치한다.
```
$ sudo apt-get install gcc-4.6 g++-4.6 gcc-4.6-multilib g++-4.6-multilib
```
  
6. Makefile.config를 생성한다.
```
$ cp Makefile.config.example Makefile.config
```
  
7. g++4.6을 사용하도록 설정한다.
```
$ sed -i "s/# CUSTOM_CXX := g++/CUSTOM_CXX := g++-4.6/" Makefile.config
```
  
8.gedit와 같은 편집기로 Makefile.config 파일을 오픈하고 pycaffe 및 cuDNN을 사용하기 위해서 아래 부분을 찾아서 주석처리 되어 있는 부분을 해제한다.
```
# In your Makefile.config, make sure to have this line uncommented
WITH_PYTHON_LAYER := 1

# Unrelatedly, it's also recommended that you use CUDNN
USE_CUDNN := 1
```
  
9. $FRCN/caffe-fast-rcnn/src/caffe/util/db_lmdb.cpp파일을 열어서 LMDB_MAP_SIZE를 1099511627776에서 536870912로 수정한다.(참고: Aaron Schumacher의 글 [The NVIDIA Jetson TK1 with Caffe on MNIST](http://planspace.org/20150614-the_nvidia_jetson_tk1_with_caffe_on_mnist/) )  

10. 변경후 파일을 닫고 아래 명령으로 컴파일한다. (시간이 상당히 걸리므로 차 한잔~)
```
$ make all -j4
$ make test -j4
$ make runtest -j4
```
  
Caffe가 제대로 동작하는지 확인하기 위해서 아래 명령으로 벤치마킹 테스트를 해본다. 출력되는 수행시간 결과는 10번의 iteration의 합이므로 10으로 나누어주면 한 이미지당 영상처리 시간을 구할 수 있다. (TK1의 경우 약 23ms 정도 나오는 듯 싶다)
```
$ build/tools/caffe time --model=models/bvlc_alexnet/deploy.prototxt --gpu=0
```
  
pycaffe 설치는 아래 과정으로 진행한다.
```
$ cd caffe-for-cudnn-v2.5.48/python
$ sudo apt-get install python-pip
$ for req in $(cat requirements.txt); do sudo pip install $req; done
$ cd ../
$ make all
$ sudo apt-get install python-numpy
$ make pycaffe
```
  
## 참고 사이트  
1. http://myzharbot.robot-home.it/blog/software/configuration-nvidia-jetson-tk1/  
2. https://www.stereolabs.com/blog/index.php/2015/09/24/getting-started-with-jetson-tk1-and-zed/
3. https://huangying-zhan.github.io/2016/08/16/Caffe-installation-and-practice-on-Jetson-TK1.html
4. https://www.stereolabs.com/blog/index.php/2015/09/07/use-your-zed-camera-with-ros/
5. http://kirumang.tistory.com/27
6. https://github.com/ros-visualization/rviz/issues/955
7. http://researchmap.jp/jojgwe7tm-2001408/
8. https://github.com/rbgirshick/py-faster-rcnn/issues/236 (py-faster-rcnn 빌드 오류 해결)
9. https://lastone9182.github.io/2016/09/16/aws-caffe.html
10. https://github.com/joeking11829/py-faster-rcnn-tk1.git (Jetson tk1용 py-faster-rcnn)