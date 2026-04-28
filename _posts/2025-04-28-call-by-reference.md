---
title: 자바는 항상 Call by value이다....!!! 
description: 이게 Call by value임? 아니 잠깐만....Call by reference 아님??
author: yunjae
date: 2026-04-28 15:50:00 +0900
categories: [java]
tags: [java]
pin: true
mermaid: true
---

## Call by value

**부제 : 근데 이거 Call by reference 아님??**

Java는 Call by value라는 방식을 통해 매개변수 전달(Parameter Passing)을 수행한다. 즉, 값을 직접 복사해 변수를 전달하는 것이다. Reference type이 객체의 주소값을 가지고 있고, 객체에 대한 수정이 가능해 Call by reference와 비슷하게 동작한다고 느껴질 수도 있지만 자바는 Call by value만을 지원한다!!!!!!!!!!!!


### Primitive type

```java

public class TestReference {
	
	public static void swap(int a, int b) {
		int temp = a;
		a = b;
		b = temp;
		System.out.println("[swap] a = " + a + " " + "b = " + b);
	}

	public static void main(String[] args) {	
		int a = 5;
		int b = 10;
		
		System.out.println("[main] a = " + a + " " + "b = " + b);
		swap(a, b);
		System.out.println("[main] a = " + a + " " + "b = " + b);
	}
}

/*
	
	실행 결과
 	[main] a = 5 b = 10
 	[swap] a = 10 b = 5
 	[main] a = 5 b = 10

 * /
```
call by value이므로 swap 함수 내에서만 a와 b의 값이 바뀐 것을 알 수 있다.


### Reference type

```java

public class TestReference {
	
	public static class Node{
		int value;
		public Node(int value) {
			this.value = value;
		}
		
		@Override
		public String toString() {
			return String.format("NodeValue = %d", this.value);
		}
	}
	
	
	public static void change(Node node) {
		node.value = 428;
		System.out.println("[change] node = " + node);
	}
	
	
	public static void main(String[] args) {
	
		Node node = new Node(1);
		System.out.println("[main] node = " + node);
		change(node);
		System.out.println("[main] node = " + node);
		
	}
	
}

/*
	
	실행 결과
	[main] node = NodeValue = 1
	[change] node = NodeValue = 428
	[main] node = NodeValue = 428

 * /

```
Reference type을 매개변수로 넘기고, 함수에서 객체의 필드 값을 수정했을 때 main 함수에서도 객체의 필드 값이 수정된 것을 확인할 수 있다.

> 어????? 이러면 call by reference 아니야?? 

**아니다.** 매개변수 node가 `원본 주소값을 복사`해서 가지고 있기 때문에 node 변수를 통해 필드 값을 수정하면 원래 객체의 필드 값이 수정되는 것은 어쩌면 자명한 일이다. change 함수에서 참조하는 객체를 변경하면 어떻게 될까?? 

```java

public class TestReference {
	
	public static class Node{
		int value;
		public Node(int value) {
			this.value = value;
		}
		
		@Override
		public String toString() {
			return String.format("NodeValue = %d", this.value);
		}
	}
	
	
	public static void change(Node node) {
		node = new Node(428);
		System.out.println("[change] node = " + node);
	}
	
	
	public static void main(String[] args) {
		Node node = new Node(1);
		System.out.println("[main] node = " + node);
		change(node);
		System.out.println("[main] node = " + node);
	}
	
}

/*
	
	실행 결과
	[main] node = NodeValue = 1
	[change] node = NodeValue = 428
	[main] node = NodeValue = 1

 * /

```
함수 내부에서는 참조하는 객체가 변경되었지만, main 함수의 node 변수는 여전히 필드 값인 1인 객체를 가리키고 있다.... 즉, 매개변수로 넘긴 node는 주소값을 복사해 주소값을 가지는 변수에 불과하다는 것을 알 수 있다. ~~실제 주소를 넘기는 방법(Call by reference)이라면 main으로 돌아왔을 때도 변경된 객체를 가리키고 있었어야 했을 것이다.~~