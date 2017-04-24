# Welcome to the IdeaFestival2017 wiki!  
기억력의 한계로..나중을 위해 개발 과정의 삽질 및 성공 내용 기록함.   
## Jetson TK1 flash 방법  
![](http://kr.nvidia.com/content/tegra/images/jetson/jetson-tk1.png)  
준비사항: Ubuntu 14.04 버젼이 설치된 host PC(인터넷 연결 필요)  
리눅스 pc가 없는 경우 윈도우즈 상에서 vmware나 virtualbox로도 flash 가능..  
단, Jetpack을 이용한 설치는 경험상 비추.. 아래 github에 정리된 방법으로 성공하였음.  

https://gist.github.com/jetsonhacks/2717a41f7e60a3405b34
  
마지막 flash 단계에서 eMMC 16GB 용량을 모두 사용하고 싶으면 '-S 14580MiB' 옵션을 추가할 것  
```sudo ./flash.sh -S 14580MiB jetson-tk1 mmcblk0p1```  
  


## Install CUDA 6.5 & OpenCV4Tegra
CUDA 패키지 및 OpenCV를 설치과정은 아래 링크를 참고하였다.  

<https://huangying-zhan.github.io/2016/08/16/Caffe-installation-and-practice-on-Jetson-TK1.html>  

CUDA ToolKit & OpenCV4Tegra 다운로드: [NVIDIA 개발자 사이트](https://developer.nvidia.com/linux-tegra-rel-21)  
  
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
$ cd /home/ubuntu/Desktop/packages_for_caffe

$ sudo dpkg -i libopencv4tegra-repo_l4t-r21_2.4.10.1_armhf.deb
$ sudo apt-get update
$ sudo apt-get install libopencv4tegra libopencv4tegra-dev
```  