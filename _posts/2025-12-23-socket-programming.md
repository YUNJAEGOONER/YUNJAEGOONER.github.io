---
title: 소켓 프로그래밍 1
description: 소켓 프로그래밍 개요 및 소켓 프로그래밍의 주요 함수
author: yunjae
date: 2025-12-23 11:33:00 +0900
categories: [network]
tags: [network, socket programming]
pin: true
math: true
mermaid: true
---


### 소켓 프로그래밍
소켓은 7계층에서 4계층(프로세스 간의 통신)의 기능을 활용하기 위한 API이다. 즉, 프로그래밍 언어로 전송 계층의 기능들을 쉽게 이용할 수 있게 해주는 도우미라고 생각하면 된다. 소켓 프로그래밍에서 사용되는 함수에는 어떤 것들이 있는지 살펴보자.

### 소켓의 종류 
소켓은 통신하는 대상이 누구인지에 따라 `AF_UNIX`, `AF_INET` 통신 방식에 따라 `SOCK_STREAM`, `SOCK_DGRAM`으로 나눌 수 있다. 즉 4가지 종류의 소켓이 조합될 수 있다.

- `AF_UNIX` : 하나의 호스트(시스템)내에서 서로 다른 프로세스간에 통신을 하는 경우로 하나의 호스트내에서 유일하게 존재하는 '경로명'를 주소로 활용한다. 

- `AF_INET` : 서로 떨어져 있는 호스트의 프로세스간에 통신을 하는 경우로 다른 호스트의 IP주소와 포트번호를 주소로 활용한다.

- `SOCK_STREAM` : TCP를 사용하는 경우 (연결 수립을 통한 신뢰성 보장)

- `SOCK_DGRAM` : UDP를 사용하는 경우 

```c
int sd = socket(AF_INET, SOCK_STREAM, 0); 
// 다른 호스트의 프로세스와 TCP 방식으로 통신
```

### 소켓 함수

```c
socket(int domain, int type, int protocol);

// domain : AF_UNIX, AF_INET
// type   : SOCK_STREAM, SOCK_DGRAM
```
소켓을 생성하는 함수로 정수형의 socket descriptor를 반환한다. socket descriptor는 통신을 위해 사용된다. 

```c
bind(int sd, const struct sockaddr * name, socklen_t namelen);

// sd : socket descriptor

// * name : 주소 정보가 들어 있는 구조체 
// sockaddr : sockaddr_un와 sockaddr_in을 sockaddr 타입으로 강제 형변환
```
socket() 함수를 통해 생성된 socket descriptor에 주소를 부여해 준다. 즉, 소켓 기술자에 주소를 매핑해 주기 위해 사용된다.

```c
listen(int sd, int backlog);

// backlog : 소켓에서 대기할 수 있는 클라이언트의 연결 요청 개수 (5로 설정하는것이 일반적)
```
소켓을 활성화하는 함수이다. 클라이언트의 접속을 받을 상태가 되는 것으로 연결형 서비스에서만 사용한다. (accpet를 호출한 준비가 되어 있음...)

```c
accept(int sd, struct sockaddr * addr socklen_t * addlen);

// addr : 연결을 요청한 클라이언트의 소켓 주소가 (운영체제에 의해?)자동으로 입력된다.
// 기본적으로 Blocking 방식으로 동작하는 accept함수를 누가 깨웠는지에 대한 정보가 자동으로 입력된다.
```
클라이언트의 connect 요청을 받기 위한 함수로, 특정 클라이언트와 1 : 1 통신을 위한 전용의 소켓 기술자가 반환된다. 

```c
connect(int sd, const struct sockaddr * name, socklen_t namelen);

//name : 어느 서버로 접속할지에 대한 정보 (연결을 요청할 서버의 정보)
```
클라이언트 프로세스가 서버 프로세스에 연결 요청을 할 때 사용하는 함수이다. 클라이언트는 통신을 위해서 서버에 대한 정보를 알고 있어야 한다.


> **accept와 connect의 관계**
> 
> **동작 흐름**
> 1. 서버가 accept()를 호출하고 blocking 상태가 된다. (accept는 기본적으로 blocking 상태로 동작한다.)
> 2. 클라이언트가 connect()를 호출 (connect의 매개 변수로 서버의 정보가 입력된다.)
> 3. blocking 상태의 서버가 깨어나면서 새로운 소켓이 생성되고, 서버는 해당 소켓을 활용해 클라이언트와 통신을 수행한다. (accept 함수는 클라이언트와의 통신을 위한 전용 소켓을 반환한다.)
>
> **정리**
> - accept는 서버에서 호출하는 함수이다. 
> - connect는 클라이언트에서 호출하는 함수이다.
> - 서버는 accept를 호출하고 먼저 기다려야 하고, 이후 클라이언트가 connect 호출해야 한다.
> - accept가 호출되기 전에 클라이언트가 connect를 호출하게 되면 에러가 발생한다.
{: .prompt-info }

```c
send();
recv();
```
메시지를 주고받을 때 사용하는 함수이다. TCP는 3-way Handshake를 통해 연결이 수립된 상태로 통신을 진행함으로 send/recv를 사용한다. UDP의 경우에는 연결을 수립하지 않은 상태에서 통신함으로 누구로부터 메시지를 받는지, 누구에게 메시지를 전송하는지 명시해야 하므로 sendto/recvfrom 함수를 통해 메시지를 주고받는다.
