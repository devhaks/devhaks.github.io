---
title: 도커를 배우자
tags:
 - docker
---

— 도커를 배우자 — 

<!-- excerpt -->

“컨테이너명” 대신 “컨테이너ID” 로 대채 가능.

# 이미지 다운로드
$ docker pull “이미지명”

# 이미지 삭제
$ docker image rm “이미지명"

# 컨테이너 생성
$ docker run -dit --name “컨테이너명” “이미지명”

# 컨테이너 접속
$ docker exec -it “컨테이너명” bash

# 컨테이너 중지
$ docker stop “컨테이너명”

# 컨테이너 삭제
$ docker rm “컨테이너명"

# 도커 네트워크 목록
$ docker network ls

# 볼륨 목록
$ docker volume ls 

# 볼륨 생성
$ docker volume create “볼륨명”

# 생성한 볼륨으로 컨테이너 생성
$ docker run -dit --name “컨테이너명” -v “볼륨명” “이미지명”

# 도커 파일 빌드하기
$ docker build “Dockerfile” 

# 도커 변경 내역 커밋
$ docker commit 

# 사용하지 않은 모든 도커 리소스 삭제
$ docker system prune // 모든 대상
$ docker container prune // 중지된 컨테이너만 삭제
$ docker image prune // 태깅되지 않은 이미지 삭제
