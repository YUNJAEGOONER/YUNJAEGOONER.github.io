---
title: 에라토스테네스의 체 (부제 - 지나고 보니 별거 아니네!)
description: 에라토스테네스의 체의 설명과 관련 문제 풀이 -  백준 1644
author: yunjae
date: 2026-01-10 21:10:00 +0900
categories: [algorithm]
tags: [algorithm]
pin: false
math: true
mermaid: true
---

### 에라토스테네스의 체

약 4년 전, 1학년 과목이었던 '프로그래밍 원리 및 실습' 과목에서 마주쳤던 기억이 어렴풋이 난다. 그때는 이름부터 어려워 구글링한 코드를 대충 외웠던 거 같은데...ㅎㅎ 지나고 보니 아주 간단한 알고리즘이었다. ~~그때는 chatGPT가 없던 구글링만 존재하던 낭만의 시절이었다.~~

에라토스테네스의 체는 소수를 구하기 위한(특히 N 이하의 모든 소수를 구할 때) 알고리즘이다. 1. 합성수는 특정 소수의 배수라는 아이디어와 2. 합성수의 약수를 찾기 위해서는 √n까지만 나누어 보면 된다는 두 가지 아이디어가 합쳐진 방식이다.

```java
//주어진 입력 N이하의 모든 소수를 구하기

import java.io.*;
public class Eratosthenes {

    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int N = Integer.parseInt(br.readLine());

        boolean [] numbers = new boolean[N + 1];

        for(int i = 2 ; i <= N ; i ++ ){
            numbers[i] = true;
        }

        for(int i = 2 ; i <= Math.sqrt(N) ; i ++ ){
            if(numbers[i]){
                int num = i * 2;
                while(num <= N){
                    numbers[num] = false; // 소수의 배수 지우기
                    num += i;
                }
            }
        }

        for(int i = 0 ; i < N + 1 ; i ++ ){
            if(numbers[i]){
                System.out.print(i + " ");
            }
        }

    }
}

```

### 관련 문제

**백준 1644**

이 문제는 연속되는 소수들의 합으로 특정수를 만들 수 있는 경우의 수를 찾는 문제로, 1. 주어진 수(N) 이하의 소수를 찾고 2. 슬라이딩 윈도우 알고리즘을 활용해 연속된 합을 구해 특정수를 만들 수 없는지를 판별하면 되는 문제이다.

주의할 점 : 주어진 N 이하의 소수가 없는 경우

**백준 1963**

주어진 소수에서 한 번에 한 자리수만 변경하여 목표하는 소수로 이동하는데 걸리는 최단 횟수를 구하는 문제이다. 1. 에라토스테네스의 체를 활요해 4자리 수의 소수를 모두 구하고 2. BFS를 활용하여 최단 횟수를 구하면 되는 문제이다. 한 자리수만 변경하는 과정을 떠올리고 구현(String으로 처리할 것인지, 정수형을 처리할 것인지에 대한 고민을 했다.)하는데 많은 시간이 들었다. 



