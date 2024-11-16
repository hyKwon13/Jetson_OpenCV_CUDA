# Jetson CUDA 및 PaddleOCR 환경 구축


## 목차
- [개요](#개요)
- [필수: SD카드 Jetpack SDK 설치](#필수-sd카드-jetpack-sdk-설치)
- [필수: Jetpack Ubuntu 초기 설정](#필수-jetpack-ubuntu-초기-설정)
- [필수: Jetson 고정 IP 설정](#필수-jetson-고정-ip-설정)
- [필수: Jetson OpenCV CUDA 환경 설정](#필수-jetson-opencv-cuda-환경-설정)
- [옵션: Paddlepaddle-GPU 설치](#옵션-paddlepaddle-gpu-설치)
- [옵션: PaddleOCR 설치](#옵션-paddleocr-설치)

## 개요
이 가이드는 NVIDIA Jetson에서 CUDA와 PaddleOCR 환경을 구축하는 방법에 대해 설명합니다. SD 카드 포맷, Jetpack SDK 설치, 고정 IP 설정, Python 3.7과 OpenCV CUDA 빌드, Paddlepaddle 및 PaddleOCR 설치 과정 등을 단계별로 안내합니다. Jetson을 이용한 딥러닝 프로젝트 환경을 구축하는 데 유용한 정보들을 포함하고 있습니다.

## 필수: SD카드 Jetpack SDK 설치
1. **SD 카드 포맷**: SD Card Formatter를 사용하여 64GB microSD 카드를 포맷
    - [SD Card Formatter 다운로드](https://www.sdcard.org/downloads/formatter/sd-memory-card-formatter-for-windows-download/)
    - 포맷 버튼을 클릭하여 포맷을 완료
    - ![이미지](https://github.com/hyKwon13/Jetson-nano_PaddleOCR_CUDA/assets/117807382/40fec450-6ca2-48fe-b878-b73b55925ef2)

2. **Jetpack SDK 4.6.1 설치**:
    - [Jetpack SDK 다운로드](https://developer.nvidia.com/embedded/jetpack-sdk-461)

3. **balenaEtcher 사용**: balenaEtcher 프로그램을 사용하여 microSD 카드에 이미지 설치.
    - [Etcher 다운로드](https://etcher.balena.io/)

## 필수: Jetpack Ubuntu 초기 설정
1. **모든 설정은 기본 값으로 설정**

## 필수: Jetson 고정 IP 설정

1. **초기 화면에서 우측 상단의 System Settings 클릭**
    - ![KakaoTalk_20241108_145417910_05](https://github.com/hyKwon13/python3.7_opencv/blob/f46a90015cf6355457f6459d08adf51f12636937/KakaoTalk_20241108_145417910_05.jpg)

2. **Network 클릭**
    - ![KakaoTalk_20241108_145417910_04](https://github.com/hyKwon13/python3.7_opencv/blob/f46a90015cf6355457f6459d08adf51f12636937/KakaoTalk_20241108_145417910_04.jpg)

3. **3번째 wired에서 option 클릭**
    - ![KakaoTalk_20241108_145417910_03](https://github.com/hyKwon13/python3.7_opencv/blob/f46a90015cf6355457f6459d08adf51f12636937/KakaoTalk_20241108_145417910_03.jpg)

4. **IPv4 Settings 에서 Manual 클릭**
    - ![KakaoTalk_20241108_145417910_02](https://github.com/hyKwon13/python3.7_opencv/blob/f46a90015cf6355457f6459d08adf51f12636937/KakaoTalk_20241108_145417910_02.jpg)

5. **원하는 IP 설정한 뒤 Netmask 255.255.255.0, Gateway 192.168.2.1로 저장 후 재부팅 시 IP 변경됨**
    - ![KakaoTalk_20241108_145417910_01](https://github.com/hyKwon13/python3.7_opencv/blob/f46a90015cf6355457f6459d08adf51f12636937/KakaoTalk_20241108_145417910_01.jpg)


## 필수: 젯슨 OpenCV CUDA 환경 설정

### 1. Python 3.7 설치 및 numpy 설치 
- 주의: Jetson은 ARM 타입 아키텍처이므로 x86 python 설치시 OpenCV CUDA 빌드 불가능
```bash
sudo apt update

sudo apt install -y build-essential libssl-dev zlib1g-dev \
libncurses5-dev libncursesw5-dev libreadline-dev libsqlite3-dev \
libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev \
libffi-dev uuid-dev

cd /usr/src
sudo wget https://www.python.org/ftp/python/3.7.12/Python-3.7.12.tgz
sudo tar xzf Python-3.7.12.tgz
cd Python-3.7.12
sudo ./configure --enable-optimizations --enable-shared
sudo make altinstall
sudo ldconfig

sudo apt update
sudo apt install python3-pip

python3.7 -m pip install numpy
```
  
### 2. 필요한 의존성 설치
OpenCV를 빌드하기 전에 필요한 의존성을 설치합니다. 이 단계는 OpenCV의 기능을 모두 지원하기 위해 다양한 라이브러리를 설치합니다.

```bash
sudo apt-get update && sudo apt-get install -y build-essential cmake unzip pkg-config libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk-3-dev libatlas-base-dev gfortran python3-dev
```

### 3. OpenCV 소스 코드 다운로드 및 빌드
OpenCV와 OpenCV Contrib 모듈의 최신 버전을 다운로드하고 빌드합니다.

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
CMake를 사용하여 OpenCV를 CUDA와 함께 빌드하도록 구성합니다. 주의할 점은 모든 경로가 정확하고, CUDA 관련 옵션을 올바르게 설정하는 것입니다.

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.9.0/modules \
      -D ENABLE_NEON=ON \
      -D BUILD_TESTS=OFF \
      -D WITH_CUDA=ON \
      -D CUDA_ARCH_BIN="5.3" \
      -D CUDA_ARCH_PTX="" \
      -D WITH_CUBLAS=ON \
      -D WITH_LIBV4L=ON \
      -D BUILD_opencv_python3=TRUE \
      -D PYTHON3_EXECUTABLE=/usr/local/bin/python3.7 \
      -D PYTHON3_INCLUDE_DIR=/usr/local/include/python3.7m \
      -D PYTHON3_LIBRARY=/usr/local/lib/libpython3.7m.so \
      -D PYTHON3_PACKAGES_PATH=/home/visionsensor/.local/lib/python3.7/site-packages \
      -D BUILD_opencv_python2=OFF \
      -D BUILD_EXAMPLES=ON ..
```

### 5. 빌드 및 설치

```bash
make -j$(nproc) # 1~2시간 정도 소요됩니다.
sudo make install
sudo ldconfig
```

### 6. 설치 확인
Python에서 OpenCV가 올바르게 설치되었는지 확인합니다.

```bash
import cv2
print(cv2.__version__)
print(cv2.cuda.getCudaEnabledDeviceCount())
```

## 옵션: Paddlepaddle-GPU 설치
Python 3.7에 paddlepaddle-gpu를 설치하기 위해 whl 파일을 직접 다운로드하여 설치하였습니다.

1. **paddlepaddle-gpu 다운로드**: [다운로드 링크](https://forums.developer.nvidia.com/t/paddlepaddle-for-jetson/242765)
    - ![이미지](https://github.com/hyKwon13/Jetson-nano_PaddleOCR_CUDA/assets/117807382/fd80b418-f0a8-4eb9-ad90-5e9ecf788fbb)


2. **whl 파일 전송**

   데스크탑에서 ssh를 통해 jetson nano를 사용중이므로 cmd에서 pscp 명령어를 사용하여 whl 파일을 jetson nano로 전송합니다.
   ```bash
    pscp C:\temp\* root@192.168.2.222:/temp
    ```

4. **whl 파일 설치**
    ```bash
    python3.7 -m pip install paddlepaddle_gpu-2.4.1-cp37-cp37m-linux_aarch64.whl
    ```

5. **오류 해결**
    ```bash
    Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-apmufue2/paddle-bfloat/
    ```

    이 오류는 paddle-bfloat 패키지를 설치하는 과정에서 발생한 문제로, 주로 필요한 빌드 의존성 패키지가 누락되었거나, 설치 과정에서 문제가 발생했을 때 나타납니다. 아래의 방법들을 통해 문제를 해결할 수 있습니다.

    1. **pip 및 setuptools 업그레이드**
        ```bash
        pip3 install --upgrade pip setuptools
        ```

    2. **numpy 미리 설치**
        ```bash
        pip3 install numpy
        ```

    3. **paddle-bfloat 패키지 직접 설치**
        ```bash
        pip install paddle-bfloat==0.1.7
        pip install paddlepaddle_gpu-2.4.1-cp37-cp37m-linux_aarch64.whl
        ```

## 옵션: PaddleOCR 설치

1. **paddleocr 설치**
    ```bash
    pip3 install paddleocr
    ```

2. **오류 해결**
    ```bash
    Failed to build psutil
    ERROR: Could not build wheels for psutil, which is required to install pyproject.toml-based projects
    ```
    이 오류는 `psutil` 패키지를 빌드하는 과정에서 Python 헤더 파일을 찾을 수 없어서 발생합니다. 다음 명령어로 해결할 수 있습니다:

    ```bash
    sudo apt-get install gcc python3-dev
    pip3 install paddleocr
    ```

    만약 해결되지 않으면 추가 의존성을 설치해보세요:
    ```bash
    sudo apt-get install build-essential
    pip install paddleocr
    ```

3. **여전히 오류 발생 시**
    `python3-dev` 패키지가 누락된 경우 다시 설치해보세요.
    ```bash
    sudo apt-get install python3.7-dev
    pip install psutil
    ```

4. **환경 변수 설정 확인**
    Python 헤더 파일을 찾지 못할 경우, CPATH 환경 변수를 설정해보세요.
    ```bash
    export CPATH=/usr/include/python3.7m
    ```

    이후 다시 `psutil`을 설치합니다.
    ```bash
    pip install psutil
    ```

5. **OpenCV 환경 변수 설정**
    ```plaintext
    ImportError: OpenCV loader: missing configuration file: ['config-3.7.py', 'config-3.py']. Check OpenCV installation.
    ```
    이 오류가 발생하면 OpenCV 설치 경로를 확인하고, Python 3.7 환경 변수를 설정합니다.
    ```bash
    export PYTHONPATH=~/project/paddle3.7/lib/python3.7/site-packages:$PYTHONPATH
    ```

6. **설치 확인 예제**
    다음 예제를 통해 PaddleOCR가 GPU와 TensorRT를 사용하는지 확인하세요.
    
    ```python
    import cv2
    from paddleocr import PaddleOCR
    
    def draw_rectangle(event, x, y, flags, param):
        global drawing, top_left_pt, bottom_right_pt, rectangles
        if event == cv2.EVENT_LBUTTONDOWN:
            drawing = True
            top_left_pt = (x, y)
            bottom_right_pt = (x, y)
        elif event == cv2.EVENT_MOUSEMOVE:
            if drawing:
                bottom_right_pt = (x, y)
        elif event == cv2.EVENT_LBUTTONUP:
            drawing = False
            bottom_right_pt = (x, y)
            rectangles.append((top_left_pt, bottom_right_pt))
    
    def perform_ocr(frame, top_left_pt, bottom_right_pt, ocr):
        if top_left_pt == (-1, -1) or bottom_right_pt == (-1, -1):
            return
        roi = frame[top_left_pt[1]:bottom_right_pt[1], top_left_pt[0]:bottom_right_pt[0]]
        if roi.size == 0:
            return
        result = ocr.ocr(roi, cls=False)
        result = result[0]
        if result is not None:
            for line in result:
                print(line[1][0])
                cv2.putText(frame, line[1][0], (top_left_pt[0], bottom_right_pt[1]+21), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
    
    cv2.namedWindow('paddleocr')
    cv2.setMouseCallback('paddleocr', draw_rectangle)
    cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
    
    drawing = False
    top_left_pt, bottom_right_pt = (-1, -1), (-1, -1)
    rectangles = [] 
    
    ocr = PaddleOCR(
        use_gpu=True,
        use_tensorrt=True,
        use_angle_cls=False,
        lang='en',
        show_log=False,
    )
    
    for _ in range(30):
        _, _ = cap.read()
    
    while True:
        ret, frame = cap.read()
    
        for rect in rectangles:
            cv2.rectangle(frame, rect[0], rect[1], (0, 255, 0), 2)
        
        if drawing and top_left_pt != (-1, -1) and bottom_right_pt != (-1, -1):
            cv2.rectangle(frame, top_left_pt, bottom_right_pt, (0, 255, 0), 2)
        if not drawing and top_left_pt != (-1, -1) and bottom_right_pt != (-1, -1):
            perform_ocr(frame, top_left_pt, bottom_right_pt, ocr)
        cv2.imshow('paddleocr', frame)
        if cv2.waitKey(1) & 0xFF == 27:
            break
    
    cap.release()
    cv2.destroyAllWindows()
    ```

7. **TIP: 스왑 메모리 증가**
    Jetson Nano에서 스왑 메모리를 8GB로 늘려 성능을 향상시킬 수 있습니다.
    
    ```bash
    sudo fallocate -l 8G /var/swapfile8G
    sudo chmod 600 /var/swapfile8G
    sudo mkswap /var/swapfile8G
    sudo swapon /var/swapfile8G
    sudo bash -c 'echo "/var/swapfile8G swap swap defaults 0 0" >> /etc/fstab'
    ```
    
    만약 `fallocate` 오류가 발생한다면:
    ```plaintext
    fallocate: fallocate failed: Text file busy
    ```
    
    기존 스왑 파일을 비활성화하고 다시 시도하세요.
    ```bash
    sudo swapoff -a
    sudo fallocate -l 8G /var/swapfile8G
    sudo chmod 600 /var/swapfile8G
    sudo mkswap /var/swapfile8G
    sudo swapon /var/swapfile8G
    sudo bash -c 'echo "/var/swapfile8G swap swap defaults 0 0" >> /etc/fstab'
    ```
