---
title: 무작정 따라 해보는 배포 2
description: 무작정 따라 해본 배포, NGINX 그래서 그게 뭔데?
author: yunjae
date: 2026-02-10 22:13:00 +0900
categories: [deployment]
tags: [deployment, backend]
pin: true
math: true
mermaid: true
---

EC2의 IP주소에 + 포트번호(8080)를 붙이면 내가 만든 가상의 웹 서비스에 요청을 보낼 수 있다. 
하지만 그 누구도 웹 서비스에 접속할때 도메인 주소 + port번호를 입력하는 사람은 없다. 
http는 일반적으로 80포트를 이용한다.
이를 위해 NGINX의 리버스 프록시 기능을 활용해본다.

### NGINX와 리버스 프록시
엔진엑스...많이 들어봤는데 아직도 정확히 무슨 역할을 하는지 잘 모르겠다. ~~웹 서버라는건 아는데...~~  

지금 내가 만든 가상의 웹 서비스는 나의 서버(EC2)에서 8080 포트에서 실행되고 있다.

프록시는 "대리자"라는 뜻이다. 그럼 프록시가 뭘 대신해줄까? 

80 포트로 들어온 클라이언트의 요청을 받아 8080포트에 실행되고 있는 스프링 부트 서버(백엔드 서버)에게 요청을 전달해준다.

리버스 프록시 : 


- 
- 보안 : 서버 정보를 클라인트로부터 숨김이 가능
- 클라이언트 요청을 보내느곳은 reverse proxy, 클라이언트는 실제 서버는 알지 못함 
- 실제 서버의 IP가 노출 되지 않는다.



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

# 기존에 존재하는 jar 파일을 삭제하고, 특정 경로에 jar파일을 복사
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