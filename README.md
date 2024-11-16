<!-- README.md -->
# Jetson CUDA 및 PaddleOCR 환경 구축 가이드

## 목차
- [개요](#개요)
- [필수: SD카드 Jetpack SDK 설치](#필수-sd카드-jetpack-sdk-설치)
- [필수: Jetpack Ubuntu 초기 설정](#필수-jetpack-ubuntu-초기-설정)
- [필수: Jetson 고정 IP 설정](#필수-jetson-고정-ip-설정)
- [필수: Jetson OpenCV CUDA 환경 설정](#필수-jetson-opencv-cuda-환경-설정)
  - [Python 3.7 설치 및 numpy 설치](#1-python-37-설치-및-numpy-설치)
  - [필요한 의존성 설치](#2-필요한-의존성-설치)
  - [OpenCV 소스 코드 다운로드 및 빌드](#3-opencv-소스-코드-다운로드-및-빌드)
  - [CMake를 사용한 OpenCV 구성](#4-cmake를-사용한-opencv-구성)
  - [빌드 및 설치](#5-빌드-및-설치)
  - [설치 확인](#6-설치-확인)
  - [필수 패키지 설치](#7-필수-패키지-설치)
  - [GPIO 권한 부여](#8-gpio-권한-부여)
  - [서버 코드 실행](#9-서버-코드-실행)
- [옵션: Paddlepaddle-GPU 설치](#옵션-paddlepaddle-gpu-설치)
  - [whl 파일 설치](#4-whl-파일-설치)
  - [오류 해결](#5-오류-해결)
- [옵션: PaddleOCR 설치](#옵션-paddleocr-설치)
  - [설치 확인 예제](#6-설치-확인-예제)
  - [TIP: 스왑 메모리 증가](#7-tip-스왑-메모리-증가)

## 개요
이 가이드는 NVIDIA Jetson에서 CUDA와 PaddleOCR 환경을 구축하는 방법에 대해 설명합니다. SD 카드 포맷, Jetpack SDK 설치, 고정 IP 설정, Python 3.7과 OpenCV CUDA 빌드, Paddlepaddle 및 PaddleOCR 설치 과정 등을 단계별로 안내합니다. Jetson을 이용한 딥러닝 프로젝트 환경을 구축하는 데 유용한 정보들을 포함하고 있습니다.

## 필수: SD카드 Jetpack SDK 설치
1. **SD 카드 포맷**: SD Card Formatter를 사용하여 64GB microSD 카드를 포맷합니다. [SD Card Formatter 다운로드](https://www.sdcard.org/downloads/formatter/sd-memory-card-formatter-for-windows-download/)

2. **Jetpack SDK 4.6.1 설치**: [Jetpack SDK 다운로드](https://developer.nvidia.com/embedded/jetpack-sdk-461)

3. **balenaEtcher 사용**: balenaEtcher를 사용하여 microSD 카드에 이미지 설치. [Etcher 다운로드](https://etcher.balena.io/)

## 필수: Jetpack Ubuntu 초기 설정
1. **모든 설정은 기본 값으로 설정**

## 필수: Jetson 고정 IP 설정
1. **System Settings**에서 네트워크 설정을 변경하여 고정 IP 설정.
2. **Network** -> **Wired 옵션** -> **IPv4 Settings에서 Manual**로 변경하여 원하는 IP 설정 후 저장.

## 필수: Jetson OpenCV CUDA 환경 설정

### 1. Python 3.7 설치 및 numpy 설치
```bash
sudo apt update
sudo apt install -y build-essential libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev libreadline-dev libsqlite3-dev libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev libffi-dev uuid-dev
cd /usr/src
sudo wget https://www.python.org/ftp/python/3.7.12/Python-3.7.12.tgz
sudo tar xzf Python-3.7.12.tgz
cd Python-3.7.12
sudo ./configure --enable-optimizations --enable-shared
sudo make altinstall
sudo ldconfig

python3.7 -m pip install numpy
```

### 2. 필요한 의존성 설치
OpenCV 빌드를 위한 필수 의존성을 설치합니다.
```bash
sudo apt-get update && sudo apt-get install -y build-essential cmake unzip pkg-config libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk-3-dev libatlas-base-dev gfortran python3-dev
```

### 3. OpenCV 소스 코드 다운로드 및 빌드
```bash
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.9.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.9.0.zip
unzip opencv.zip
unzip opencv_contrib.zip
cd opencv-4.9.0/
mkdir build
cd build
```

### 4. CMake를 사용한 OpenCV 구성
```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.9.0/modules \
      -D ENABLE_NEON=ON \
      -D BUILD_TESTS=OFF \
      -D WITH_CUDA=ON \
      -D CUDA_ARCH_BIN="5.3" \
      -D WITH_CUBLAS=ON \
      -D WITH_LIBV4L=ON \
      -D BUILD_opencv_python3=TRUE \
      -D PYTHON3_EXECUTABLE=/usr/local/bin/python3.7 \
      -D PYTHON3_INCLUDE_DIR=/usr/local/include/python3.7m \
      -D PYTHON3_LIBRARY=/usr/local/lib/libpython3.7m.so \
      -D PYTHON3_PACKAGES_PATH=/home/visionsensor/.local/lib/python3.7/site-packages \
      -D BUILD_EXAMPLES=ON ..
```

### 5. 빌드 및 설치
```bash
make -j$(nproc)
sudo make install
sudo ldconfig
```

### 6. 설치 확인
Python에서 OpenCV 설치 확인:
```bash
python3.7
import cv2
print(cv2.__version__)
print(cv2.cuda.getCudaEnabledDeviceCount())
```

### 7. 필수 패키지 설치
```bash
python3.7 -m pip install --upgrade pip
pip3.7 install python-multipart
python3.7 -m pip install fastapi uvicorn
sudo python3.7 -m pip install Jetson.GPIO
sudo apt-get install v4l-utils
```

### 8. GPIO 권한 부여
```bash
sudo usermod -aG gpio $USER
```

### 9. 서버 코드 실행
```bash
sudo python3.7 server.py
```

## 옵션: Paddlepaddle-GPU 설치
1. **paddlepaddle-gpu 다운로드**: [다운로드 링크](https://forums.developer.nvidia.com/t/paddlepaddle-for-jetson/242765)
2. **whl 파일 전송**: 데스크탑에서 pscp를 이용하여 Jetson으로 파일 전송
3. **whl 파일 설치**:
```bash
python3.7 -m pip install paddlepaddle_gpu-2.4.1-cp37-cp37m-linux_aarch64.whl
```

4. **오류 해결**:
```bash
pip3 install --upgrade pip setuptools
pip3 install numpy
pip install paddle-bfloat==0.1.7
```

## 옵션: PaddleOCR 설치
1. **paddleocr 설치**:
```bash
pip3 install paddleocr
```
2. **오류 해결**: Python 헤더 파일 관련 오류 발생 시 해결:
```bash
sudo apt-get install gcc python3-dev
pip3 install paddleocr
```
3. **환경 변수 설정 확인**: Python 헤더 파일 경로 설정:
```bash
export CPATH=/usr/include/python3.7m
```

4. **설치 확인 예제**: PaddleOCR가 GPU와 TensorRT를 사용하는지 확인하기 위한 코드 작성.

## 7. TIP: 스왑 메모리 증가
Jetson Nano의 성능 향상을 위해 스왑 메모리를 8GB로 증가시킵니다.
```bash
sudo fallocate -l 8G /var/swapfile8G
sudo chmod 600 /var/swapfile8G
sudo mkswap /var/swapfile8G
sudo swapon /var/swapfile8G
sudo bash -c 'echo "/var/swapfile8G swap swap defaults 0 0" >> /etc/fstab'
```

만약 `fallocate` 오류가 발생한다면 기존 스왑 파일을 비활성화 후 다시 시도합니다.
```bash
sudo swapoff -a
sudo fallocate -l 8G /var/swapfile8G
sudo chmod 600 /var/swapfile8G
sudo mkswap /var/swapfile8G
sudo swapon /var/swapfile8G
sudo bash -c 'echo "/var/swapfile8G swap swap defaults 0 0" >> /etc/fstab'
```
