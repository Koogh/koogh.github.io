---
title: "ROS2_Humble_Nav2_Docker_Setting" 
date: 2026-02-19 23:55:00 +0900 
categories: [Setting] 
tags: [ros2, docker, navigation2] 
math: true
---
이 포스트에서는 ROS 2 Humble 에서 Nav2 스택을 사용하여 개발할 수 있는 Docker 환경을 구축하는 방법을 단계별로 안내합니다. 
Docker를 사용하면 호스트 시스템에 ROS 2를 직접 설치할 필요 없이, 격리되고 일관된 개발 환경을 쉽게 구성하고 관리할 수 있습니다.

## 1. Dockerfile 작성 (`Dockerfile.humble_nav2`)

먼저, 필요한 모든 패키지와 설정이 포함된 Docker 이미지를 생성하기 위한 `Dockerfile`을 작성합니다.

- **Base**: osrf/ros:humble-desktop (GUI 툴 지원)
- **Essential Tools**: git, vim, terminator, gdb 등 개발 필수 도구 포함
- **Packages**: Nav2, SLAM-Toolbox, TurtleBot3(Waffle) 시뮬레이션 환경
- **Custom Config**: ROS_DOMAIN_ID 고정 및 자동 환경 소싱

```dockerfile
# Base Image: ROS 2 Humble Desktop (GUI 툴 포함)
FROM osrf/ros:humble-desktop

# 환경 변수 설정 (대화형 프롬프트 방지)
ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=humble

# 기본 개발 도구, SLAM 및 Nav2 관련 패키지 설치
RUN apt-get update && apt-get install -y \
    git \
    wget \
    curl \
    vim \
    terminator \
    build-essential \
    cmake \
    gdb \
    python3-pip \
    python3-rosdep \
    python3-vcstool \
    python3-colcon-common-extensions \
    ros-${ROS_DISTRO}-navigation2 \
    ros-${ROS_DISTRO}-nav2-bringup \
    ros-${ROS_DISTRO}-slam-toolbox \
    ros-${ROS_DISTRO}-turtlebot3-gazebo \
    ros-${ROS_DISTRO}-turtlebot3-navigation2 \
    ros-${ROS_DEBIAN_FRONTEND=noninteractive}-turtlebot3-simulations \
    && rm -rf /var/lib/apt/lists/*

# 컨테이너 실행 시 ROS 2 환경 자동 소싱 및 기본 설정
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc
RUN echo "export ROS_DOMAIN_ID=30" >> ~/.bashrc
RUN echo "export TURTLEBOT3_MODEL=waffle" >> ~/.bashrc

# 기본 작업 디렉토리 설정
WORKDIR /root/ros2_ws

CMD ["bash"]
```

## 2. Docker 이미지 빌드 (`build_humble_nav2_image.sh`)

위에서 작성한 `Dockerfile`을 사용하여 Docker 이미지를 빌드합니다. 
간단한 셸 스크립트를 작성하여 빌드 과정을 자동화할 수 있습니다.

- **이미지 이름**: `humble_nav2:2602`로 지정합니다. 이 이미지 이름(humble_nav2)과 태그(2602)는 편한 방식으로 바꾸면 됩니다.
- **빌드 옵션**: `--no-cache` 옵션을 사용하여 빌드 실패하여 다시 빌드 시도할때 깨끗한 백지 상태에서 이미지 빌드가 가능하도록 합니다.

```bash
#!/bin/bash
# 이미지 이름: humble_nav2:2602
# 타겟 Dockerfile: Dockerfile.humble_nav2

echo "ROS 2 Humble Nav2 Docker 이미지를 캐시 없이 새로 빌드합니다..."
docker build --no-cache -f Dockerfile.humble_nav2 -t humble_nav2:2602 .
```

터미널에서 아래 명령어를 실행하여 이미지를 빌드하세요.

```bash
bash build_humble_nav2_image.sh
```

## 3. Docker 컨테이너 실행 (`docker_run_humble_nav2.sh`)

이제 빌드된 이미지를 사용하여 Docker 컨테이너를 실행합니다. 이 스크립트는 컨테이너의 현재 상태를 확인하여, 실행 중이면 접속(`exec`)하고, 중지 상태이면 다시 시작하며, 컨테이너가 없으면 새로 생성(`run`)하는 기능을 포함하고 있어 편리합니다.

- **GUI 지원**: 호스트의 X 서버와 디스플레이 환경을 컨테이너와 공유하여 RViz2나 Gazebo 같은 GUI 프로그램을 실행할 수 있도록 설정합니다.
- **볼륨 마운트**:
    - `ros2_ws` 디렉토리를 호스트와 컨테이너 간에 공유하여 작업 내용을 영구적으로 보존합니다.
    - 호스트의 홈 디렉토리(`$HOME`)와 마운트 디렉토리(`/mnt`)를 공유하여 파일 접근성을 높입니다.
- **네트워크**: `--net=host` 옵션을 사용하여 호스트와 동일한 네트워크 환경을 사용함으로써 ROS 2 노드 간 통신을 간편하게 합니다.

```bash
#!/bin/bash

IMAGE_NAME="humble_nav2:2602"
CONTAINER_NAME="humble_nav2"
WORKSPACE_DIR="$(pwd)/ros2_ws"

# 공유할 디렉토리가 없으면 생성
mkdir -p "$WORKSPACE_DIR"

# GUI 애플리케이션(RViz2, Gazebo) 실행을 위한 X서버 접근 허용
xhost +local:root > /dev/null

# 컨테이너 상태 확인
CONTAINER_STATE=$(docker inspect -f '{{.State.Status}}' $CONTAINER_NAME 2>/dev/null)

if [ "$CONTAINER_STATE" == "running" ]; then
    echo "▶ 이미 실행 중인 컨테이너($CONTAINER_NAME)에 exec으로 진입합니다."
    docker exec -it $CONTAINER_NAME bash

elif [ "$CONTAINER_STATE" == "exited" ] || [ "$CONTAINER_STATE" == "created" ]; then
    echo "▶ 정지된 컨테이너($CONTAINER_NAME)를 시작하고 exec으로 진입합니다."
    docker start $CONTAINER_NAME
    docker exec -it $CONTAINER_NAME bash

else
    echo "▶ 새로운 컨테이너($CONTAINER_NAME)를 생성하고 run으로 진입합니다."
    docker run -it \
        --name $CONTAINER_NAME \
        --privileged \
        --net=host \
        --env="DISPLAY=$DISPLAY" \
        --env="QT_X11_NO_MITSHM=1" \
        --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
        --volume="$WORKSPACE_DIR:/root/ros2_ws" \
        --volume="$HOME:/myhome" \
        --volume="/mnt:/mymount" \
        $IMAGE_NAME \
        bash
fi

# 컨테이너 종료 후 X서버 접근 권한 원복
# xhost -local:root > /dev/null
# echo "X서버 접근 권한을 원복했습니다."
```

아래 명령어로 스크립트를 실행하여 컨테이너에 접속할 수 있습니다.

```bash
bash docker_run_humble_nav2.sh
```

## 마무리

이제 여러분은 ROS 2 Humble과 Nav2 개발을 위한 모든 준비가 완료된 Docker 컨테이너 환경을 갖게 되었습니다.
이 포스트가 여러분의 ROS 2 개발 환경 구축에 도움이 되기를 바랍니다!

