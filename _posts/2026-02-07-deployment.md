---
title: 무작정 따라해보는 배포1
description: 무장정 따라해본 배포, 배포 스크립트를 활용한 배포
author: yunjae
date: 2026-02-07 21:30:00 +0900
categories: [deployment]
tags: [deployment, backend]
pin: true
math: true
mermaid: true
---
지난 카테캠 2단계에서 배포를 처음 해보고, "무작정 따라 하는 배포"라는 제목으로 배포기를 포스팅하려고 했는데...미루고 미루다 여기까지 와버렸다. ~~초안까지 썻던 것은 안비밀~~ 


### 배포란 무엇일까?
우리가 특정 서비스를 이용할 수 있는 이유는 우리가 이용하려는 웹 애플리케이션이 컴퓨터(서버)에서 백그라운드로 실행되고 있기 때문이다. www.devyunjaejjang.com을 주소창에 검색하게 되면 DNS라는것을 통해 해당 서비스가 실행되고 있는 컴퓨터의 주소(IP주소)를 알아내는데, 그 주소가 바로 서비스가 실행되고 있는 컴퓨터의 주소라고 생각하면 된다. 쉽게 요약하면, 내가 개발한 웹 애플리케이션을 다른 사람이 이용하려면 어떠한 컴퓨터에서 해당 웹 애플리케이션이 실행 상태여야 하고, 그렇게 만드는 과정을 배포라고 한다. 

### 배포 스크립트
내 웹 애플리케이션을 배포하는 과정은 크게 1.내 서버(EC2)에 최신 버전의 프로젝트를 다운(git pull) 2. 내 프로젝트를 빌드(소스 코드를 실행 가능한 형태로 변환, JAR) 3. 빌드된 파일 실행 으로 볼 수 있다. 이를 위해 직접 EC2에 접속해 해당 과정을 직접 수행할 수도 있지만, 매번 이를 수행하기가 번거로움으로 배포 스크립트라는 것을 작성한다. 해당 과정에 필요한 명령어들을 하나의 파일에 저장하고, 해당 파일만을 실행함으로써 3가지 동작이 수행되도록 하는 것이다. 

EC2 : 배포를 위해서는 개발한 프로그램(웹 애플리케이션)을 계속해서 실행해 줄 컴퓨터가 필요하다. 이를 위해 컴퓨터(인스턴스)를 AWS로부터 대여할 것이다. 

```bash
#!/bin/bash

PROJECT_PATH=/home/ubuntu/yunjae
PROJECT_NAME=app
DEPLOY_DIR=deploy

# 프로젝트 경로로 이동하기
cd $PROJECT_PATH/$PROJECT_NAME

echo "> 1. 원격 저장소로부터 프로젝트 가져오기"
git pull origin main

echo "> 2. 프로젝트 빌드"
./gradlew build

cd $PROJECT_PATH

# 기존에 존재하는 jar 파일을 삭제
rm -f $PROJECT_PATH/$DEPLOY_DIR/*.jar
cp $PROJECT_PATH/$PROJECT_NAME/build/libs/*.jar $PROJECT_PATH/$DEPLOY_DIR

# 기존에 실행중이던 애플리케이션을 종료
CUR_PID=$(pgrep -f "${PROJECT_NAME}.*.jar")

if [ -z "$CUR_PID" ]; then
    echo ""
else 
    kill -15 $CUR_PID
    sleep 5
fi

# 실행할 jar파일의 파일명을 알아내기
JAR_NAME=$(ls -tr $PROJECT_PATH/$DEPLOY_DIR | grep jar | tail -n 1) 

echo "> 3. 애플리케이션 실행"
nohup java -jar $PROJECT_PATH/$DEPLOY_DIR/$JAR_NAME &
```

다음과 같은 ~~어쨋든 실행이 되는~~ 배포 스크립트를 작성한 후에, 해당 스크립트에 실행 권한을 부여해 주면 `chmod +x deploy.sh`, 스크립트를 실행 `./deploy.sh`할 수 있다. 보안 설정을 통해 8080 포트를 열어주게 되면 외부에서도 나의 서비스를 이용할 수 있다.

![deploy1](/assets/img/posts/deploy/deploy1.png)