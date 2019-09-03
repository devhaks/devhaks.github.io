---
title: Docker 초급 (1)
date: 2019-06-30 23:45:53
tags:
  - docker

thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.png
---

Docker 를 사용하여 복잡했던 개발 환경 구축을 간단한 예제를 통해 구축해 봅니다.

<!-- excerpt -->

## 목차


* [Docker 란](#Docker-란)

 * [Images and containers](#Images-and-containers)

* [Docker 실습](#Docker-실습)

 * [Docker hub 가입](#Docker-hub-가입)

 * [Docker 설치](#Docker-설치)

 * [Docker 어플리케이션 컨테이너 생성](#Docker-어플리케이션-컨테이너-생성)

* [참고](#참고)

## Docker 란

Docker는 개발자와 시스템 관리자가 컨테이너를 사용하여 응용 프로그램을 개발, 배포 및 실행하기위한 플랫폼입니다.

Linux 컨테이너를 사용하여 응용 프로그램을 배포하는 것을 컨테이너화 라고 부릅니다. 컨테이너는 새로운 것은 아니지만 쉽게 응용 프로그램을 배포하는데 사용됩니다.

아래 컨테이너의 장점이 있기 때문에 컨테이너화가 점점 더 인기를 얻고 있습니다.

- Flexible(유연성) : 가장 복잡한 애플리케이션조차도 컨테이너화할 수 있습니다.

- Lightweight(경량) : 컨테이너는 호스트 커널을 활용하고 공유합니다.

- Interchangeable(교환 가능) : 업데이트 및 업그레이드를 즉시 배포 할 수 있습니다.

- Portable(휴대성) : 로컬로 구축하고, 클라우드에 배치하고, 어디서나 실행할 수 있습니다.

- Scalable(확장성) : 컨테이너 복제본을 늘리고 자동으로 배포 할 수 있습니다.

- Stackable(스택 가능) : 서비스를 세로 및 가로로 쌓을 수 있습니다.

#### Images and containers

컨테이너는 이미지를 실행하여 시작됩니다. 이미지는 응용 프로그램을 실행하는 데 필요한 모든 것을 포함하는 실행 가능한 패키지입니다. (코드, 런타임, 라이브러리, 환경 변수 및 구성 파일)

컨테이너는 이미지의 런타임 인스턴스입니다. 즉, 이미지가 실행될 때 메모리에 저장됩니다 (즉, 상태가있는 이미지 또는 사용자 프로세스). 

Linux에서와 마찬가지로 실행중인 컨테이너 목록을 `docker ps` 명령으로 볼 수 있습니다.


## Docker 실습

도커 실습 내용으로는 간단한 nodejs express 어플리케이션을 만들어 봅니다.

#### Docker hub 가입

실습 전, [Docker hub](https://hub.docker.com)에 가입 합니다. Docker hub는 Image 또는 여러 Image를 활용하여 build 한 또 다른 Image를 모아 놓은 저장소 입니다. 

Git hub 와 유사하다고 생각하시면 됩니다.

#### Docker 설치 

개발 OS 환경에 따라 [설치](https://hub.docker.com/?overlay=onboarding)를 진행 합니다. 설치가 완료되면, 아래 처럼 확인합니다. 

  ```sh
  $ docker --version
  Docker version 18.09.2, build 6247962
  ```

docker 명령어가 실행되지 않는다면, 터미널을 재시작 합니다.

#### Docker 어플리케이션 컨테이너 생성

- nodejs 이미지를 다운로드 받습니다. https://hub.docker.com/_/node

  ```sh
  $ docker pull node
  # 이미지 다운로드 과정 출력 생략
  ... 
  Status: Downloaded newer image for node:latest

  # 도커 이미지 목록
  $ docker images 
  REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
  node                        latest              70ea061fdf3a        39 hours ago        907MB
  ```

- Nodejs Image 로 컨테이너를 생성합니다.

  ```sh
  $ docker run -dit -p 3030:3030 --name myNodeApp 70ea061fdf3a
  151c943f1e46a52fbdf8a0dbb619077eed553467c2875c71da5f7b51823bf260
  ```

  - 옵션 설명
    - -d : 백그라운드에서 실행하기 위한 daemon 실행하기
    - -p : 컨테이너 포트 포워딩 설정. http://localhost:3030 으로 접속 시, 컨테이너의 3030포트로 포워딩하기
    - --name : 컨테이너의 이름 설정
    - 마지막에는 image ID `70ea061fdf3a`를 기입

- 컨테이너 생성 확인하기
  - `-a` 옵션으로 status 상관 없이 모든 컨테이너 출력.

  ```sh
  $ docker ps -a 
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
  151c943f1e46        70ea061fdf3a        "docker-entrypoint.s…"   37 seconds ago      Up 35 seconds       0.0.0.0:3030->3030/tcp       myNodeApp 
  ```

- 컨테이너 접속 및 node & npm 버전 확인
  - 컨테이너 이름 `myNodeApp` 대신 컨테이너 ID `151c943f1e46` 사용함.
  - 접속하면 기본적으로 root 계정이고 host 는 컨테이너 ID 로 지정됨. 
  - 컨테이너에는 vim 이 기본적으로 설치 되어 있지 않기 때문에 추가적으로 vim 을 설치함. 

  ```sh
  # bash 쉘로 컨테이너 접속하기
  $ docker exec -it myNodeApp bash 
  # 기본 root 계정
  root@151c943f1e46:/#
  # node & npm 버전 확인
  root@151c943f1e46:/# node -v && npm -v
  v12.5.0
  6.9.0
  # vim 패키지 설치
  root@151c943f1e46:/home/app# apt update && apt install -y vim
  ```

- 어플리케이션 파일 추가하기
  - 미리 준비한 git code 를 clone 함.

  ```sh
  root@151c943f1e46:/# cd /home 
  root@151c943f1e46:/home# git clone https://github.com/devhaks/myNode.git

  root@151c943f1e46:/home# cd myNode
  # 패키지 설치 
  root@151c943f1e46:/home/myNode# npm install
  # node app 실행
  root@151c943f1e46:/home/myNode# node index.js

  ===========================
    http://localhost:3030
  ===========================
  ```

- http://localhost:3030 로 접속하여 정상적으로 작동하는지 확인하기



## 참고
- Docker 가이드 https://docs.docker.com



다음 포스트로는 Dockerfile 과 dcoker-compose 를 사용하여 한층 더 편리한 개발 환경을 구축하도록 하겠습니다.
