---
layout: post
title:  "Java Memory 간단히 살펴보기"
date:   2018-06-04 18:45:13 +0900
description: "JVM이 관리하는 메모리의 구조는 어떻게 되어있는지? JVM의 객체는 lifecycle에 따라 JVM 메모리 내에서 어떻게 이동되는지를 설명한다."
categories: develop
---


### Java 애플리케이션이 실행되는 동안 JVM의 메모리에서는 어떤일이 일어나고 있나?

Java 애플리케이션을 실행하는 경우, JVM 메모리에는 여러가지 데이터가 로드되고 해제된다.

JVM 의 메모리는 어떤객체를 저장하고 어떤 용도로 사용되는지에 따라 여러 영역으로 나뉘어져 있다.


<br>

### JVM의 메모리에는 어떤 영역들이 있나?

JVM 메모리는 저장하는 데이터 및 용도에 따라 여러 영역으로 나뉘어져있다. Java 8 을 기준으로 JVM 메모리 영역에 변화(Metaspace 등장)가 있었으며, 각 메모리 영역별 특정에 대해 알아보자.

<br>


### JVM 메모리 각 영역별 역할 및 특징

JVM Memory 영역은 Runtime Data Area 라고도 불리며, 여러 영역으로 나뉘어져 있다.

<br>

#### Method Area

JVM에서 읽은 클래스와 인터페이스의 정보가 저장되는 영역이며, **클래스 생성자 및 메소드의 코드(바이트 코드) 등** 이 저장된다. 클래스의 인스턴스가 생성된 후, 메소드가 실행되는 순간 클래스의 정보가 Method Area 에 저장된다.

Method Area는 **모든 Thread에 의해 공유되는 영역** 이며, JVM이 시작될 때 생성된다.

> Method Area 는 JVM 제품(벤더)에 따라 구현이 다르다.
> Hotspot JVM(Oracle)의 Method Area 는 Permanent Area(PermGen)이라고 한다.

<br>

###### Method Area - Runtime Constant Pool

Method Area 는 내부에 Runtime Constant Pool 영역을 가지고 있다. Runtime Constant Pool 영역에는 **클래스/인터페이스의 메소드, 필드, 문자열 상수등의 레퍼런스** 가 저장되며, 이들의 물리적인 메모리 위치를 참조할 경우에 사용된다.

<br>

#### Heap

new 연산자 등으로 생성된 **객체(인스턴스)와 배열 등** 을 저장되는 영역이다. Heap 영역에 저장된 객체(인스턴스)나 배열은 다른 객체에서 참조될 수 있다.

GC 가 발생하는 영역이며, 참조(레퍼런스)가 없는 객체들은 GC과정을 통해 메모리에서 제거된다. Heap 영역 또한 내부적으로 여러 영역으로 나뉘어져 있으며, 이는 객체의 lifecycle 및 GC 와 연관되어 있다.

> **튜닝 옵션** : -Xms, -Xmx

<br>

#### JVM Language Stacks

메소드 호출시 수행중인 **메소드 데이터(지역변수, 지역객체 레퍼런스, 메소드 파라미터, 메소드 리턴값 등)** 를 저장하기위한 영역이다.

Stack 영역은 **Thread별로 각각 독립적으로 생성** 된다. Stack 영역에는 메소드 진입시마다 메소드 데이터(지역변수, 지역객체 레퍼런스, 메소드 파라미터, 메소드 리턴값 등)를 포함하는 **Stack Frame** 이 생성되어 Push 되며, 메소스 생성이 완료되면 Stack Frame은 pop 되어 사라진다.

<br>

#### PC Registers

각 **Thread 마다 할당되는 영역이** 며, Thread가 시작될 때 생성된다.

PC Registers 영역에는 Thread가 실행할 JVM 명령(바이트 코드 명령)의 주소를 저장하게 된다. Java 코드가 컴파일 과정을 통해 변환된 결과물(바이트코드)는 여러 바이트코드 명령들이 나열된 형태가 되는데, JVM은 이러한 명령을 하나씩 실행하며, Java 애플리케이션을 실행해 나간다.

<br>

#### Native Method Area

Java 외의 언어로 작성된 코드(Native Code, JNI로 실행되는 코드)를 위한 Stack 영역이다. 각 언어에 맞는 Stack이 생성된다.

<br>

![Alt JVMMemory]({{"/img/jvm_memory/JVMMemory.png"| relative_url}})
*JVM Memory 구조*

<br>


### Heap 영역 자세히 보기

Heap 영역은 new 연산자 등으로 생성된 **객체(인스턴스)와 배열 등** 을 저장되는 영역으로서 GC가 발생한다.

GC는 한정적인 메모리 자원을 효율적으로 사용하기 위해 더이상 불필요한 리소스들을 메모리에서 제거하는 작업을 의미한다. 이 때, 불필요한 리소스들을 추적/관리하기 위해 Heap의 각 영역들이 필요하다.

> ###### 튜닝 옵션
> -Xms : 초기 heap size
> –Xmx : 최대 heap size

<br>

![Alt JVMHeap]({{"/img/jvm_memory/JVMHeap.png"| relative_url}})
*JVM Heap 구조*

<br>

#### Eden 영역 (Young 영역)

새로 생성된 대부분의 객체가 처음 위치하는 영역을 의미한다. Eden 영역에서 정기적으로 GC가 발생한 이후, 살아남은 객체들은 Survivor1 또는 Survivor2 중 선택된 하나의 영역으로만 이동하여 계속 쌓이게 된다.

<br>

#### Survivor1, Survivor2 영역 (Young 영역)

Survivor1 또는 Survivor2 중 하나의 영역이 꽉 차게되면, 그 중 살아남은 객체가 비워진 Survivor 영역으로 이동한다. 이때 참조가 없는 객체들을 메모리에서 정리된다.

예를 들면, Survivor1 가 꽉찬 경우 살아남은 객체만 Survivor2 로 이동하게 되는 것이다.

이러한 매커니즘 때문에, Survivor1 또는 Survivor2는 항상 비워진 상태가 되며, Survivor 영역 중 하나의 영역이 완전히 비워지지 않았다면, 문제가 있는 것이다.

<br>

> ###### Minor GC
> Eden 영역 또는 Survivor1, Survivor2 영역 등, Young 영역에서 발생하는 GC를 minor GC라고 한다.
>
> ###### 튜닝 옵션
> -XX:NewSize : 최소 new size (Eden+Survivor 영역)
> -XX:MaxNewSize : 최대 new size
> -XX:SurvivorRatio : New/Survivor영역 비율


<br>


#### Old 영역

Survivor1 또는 Survivor2 영역을 왔다갔다 하는 과정에서 끝까지 살아남은 객체만이 Old 영역으로 이동하게 된다. 보통 Old 영역은 Young 영역보다 크게 할당하며, 이러한 이유로 Old 영역의 GC는 Young 영역보다 적게 발생한다.

<br>

> ###### Major GC
> Old 영역, Permanent 영역에서 발생하는 GC를 Major GC (Full GC)라 한다.


<br>


#### Permanent 영역

클래스 로더에 의해 로드된 클래스들이 저장되는 공간이다. Java8 에서는 Permanent 영역이 좀 더 다듬어져 Metaspace 라는 영역으로 교체되었다.

> ###### 튜닝 옵션
> -XX:PermSize : 초기 Perm size
> -XX:MaxPermSize : 최대 Perm size


<br>

### JVM 메모리 영역내에서 객체의 Lifecycle

JVM 메모리 영역내에서 객체의 Lifecycle의 흐름을 보면 아래와 같다.

![Alt JVMObjectLifecycle]({{"/img/jvm_memory/JVMObjectLifecycle.png"| relative_url}})


<br>

### Java 8 의 Metaspace 영역

#### PermGen 영역의 OutOfMemoryError 이슈

기존의 PermGen 영역에는 Class/Method의 Meta정보, Static Object, 상수화된 String Object, Class와 관련된 배열 객체 Meta 정보, JVM 내부적인 객체들과 최적화컴파일러(JIT)의 최적화 정보 등이 저장되었다. 그 중 **Static Object** 를 저장하는 경우 객체의 모든 부분이 PermGen에 저장되기 때문에 개발자의 실수로 OOM이 발생하는 경우가 많았다.

```java
// someObjects의 모든 객체가 PermGen 영역에 저장되어 OOM 발생 가능성이 높아졌다.
static List<SomeObject> someObjects = Arrays.asList(
                                              new SomeObject(),
                                              new SomeObject(),
                                              new SomeObject()
                                            );
```

때문에, Java 8에서 Metaspace가 도입되면서 Static Object 및 상수화된 String Object를 heap 영역으로 옮김으로써, 최대한 GC가 될 수 있도록 하였다.


> ###### 튜닝 옵션
> -XX:MetaspaceSize : JVM이 사용하는 네이티브 메모리
> -XX:MaxMetaspaceSize : metaspace의 최대 메모리

<br>

### 참고

- [아틴블로그 - JVM 메모리구조](http://atin.tistory.com/625) - JVM Memory 구조의 도식화가 잘 되어 있음
- [Naver D2 - Java Garbage Collection](https://d2.naver.com/helloworld/1329) - JVM Memory 구조 및 VM 제품별 특성 및 GC 등에 대해 설명되어 있음
- [젼의 IT이야기 - Java/JVM 메모리 구(부제: 성능개선을 위한 GC의 활용)](http://stophyun.tistory.com/37) - Java 프로그램 실행과정 및 Java 코드에 따른 메모리 상태등에 대한 예제가 잘 설명되어 있음
- [벽보고 욕이라도 하자 - JAVA8 Permanent 영역은 어디로 가는가](https://yckwon2nd.blogspot.com/2015/03/java8-permanent.html) - Java8 에서 Metaspace가 나온 이유에 대해서 설명하고 있음
- [Programming is Fun - Java8 : PermGen에서 Metaspace로](http://netframework.tistory.com/entry/Java8-PermGen%EC%97%90%EC%84%9C-Metaspace%EB%A1%9C) - Metaspace 의 특성 및 PermGen과 비교한 내용
