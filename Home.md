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