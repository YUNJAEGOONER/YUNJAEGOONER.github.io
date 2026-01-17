---
title: NULL이라는데 이걸 세야 해? 말아야 해?
description: 
author: yunjae
date: 2024-10-01 21:10:00 +0900
categories: [db]
tags: [db]
pin: false
math: true
mermaid: true
---

### 집계함수 COUNT

COUNT는 집계함수 중 하나로, 컬럼명을 통해 행의 개수 셀 때 활용된다. COUNT와 관련된 재미있는 사실을 발견했다. 모든 컬럼을 포함하기 위한 `*`를 사용하는 경우와 특정 컬럼을 직접 지정하는 경우, 집계 방법이 달라진다는 것이다.

| 함수                | 설명                                                                 |
|---------------------|----------------------------------------------------------------------|
| `COUNT(*)`          | 테이블의 모든 행의 수를 계산한다. NULL 값을 갖는 행도 포함한다.                 |
| `COUNT(컬럼명)`| 특정 컬럼에서 NULL이 아닌 행의 수를 계산합니다. NULL 값은 제외된다.    |

### 예시
![count1](/assets/img/posts/count/count1.png)

**count(*)**
- **NULL 값을 갖는 행도 포함**하여 테이블에 존재하는 모든 행의 수를 계산한다.
![count2](/assets/img/posts/count/count2.png)


**count(특정 컬럼명)**
- 특정 컬럼에서 **NULL 값을 갖는 경우를 제외**하고 컬럼에 등장하는 값의 개수를 계산한다.
- 중복된 값도 포함하여 계산한다. 
- ex) count(num1) =  1, 3, 1 -> result : 3
![count3](/assets/img/posts/count/count3.png)

**count(`distinct` 특정 컬럼명)**
- 중복을 제외(`distinct`)하고 특정 컬럼에서 등장하는 값의 개수를 계산한다.
- ex) count(distinct num1) = 1, 3 -> result : 2
![count4](/assets/img/posts/count/count4.png)

**count(컬럼1, 컬럼2)**
- count함수의 인자로 복수개의 컬럼이 오는 경우 에러 발생
![count5](/assets/img/posts/count/count5.png)