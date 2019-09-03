---
title: Docker 초급 (2)
tags:
  - docker
date: 2019-07-31 23:45:53

thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.png
---

이전 글에서 명령어로 다소 귀찮았던 개발환경을 구축했다면 이번 글에서는 dockerfile 과 docker-compose 를 사용하여 편리하고 빠르게 개발 환경 구축해 봅니다.

<!-- excerpt -->

Docker 를 아직 다루는데 미숙하지만 사용할 수록 점점 매력에 빠져드네요 ㅎㅎ

## 목차

* [Dockerfile](#Dockerfile)

* [Dockerfile 실습](#Dockerfile-실습)

* [참고](#참고)


## Dockerfile

이전에 컨테이너를 만들려면 어떻게 했었는지 기억 하시나요? 명령어로 docker image를 다운로드하고 컨테이너 생성하고 옵션이 이것저것 붙어서 사용하기에 다소 불편했습니다.

이 불편함을 해소하고자 파일에 명령어를 기록하여 여러가지 작업들을 한번에 처리할 수 있습니다. 굉장히 편리합니다. 굳굳!!


## Dockerfile 실습


이전 글에서 사용한 명령어를 Dockerfile 에 작성하여 컨테이너를 생성하도록 하겠습니다.




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
