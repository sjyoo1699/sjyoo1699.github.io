---
layout: post
title: "180817 SYNCHRONIZATION"
date:   2018-08-17 14:00:30 +0900
categories: jekyll update
---

ActivityThread.java와 Instrumentation.java 의 코드를 보면서

동기화 코드가 굉장히 많이 나왔다.

지금까지 이러한 multi thread 코딩을 많이 접해본 적이 없기 때문에, 동기화에 대해 공부를 해보았다.

***

java의 동기화 방법에는 volatile, atomic, synchronized 이렇게 3가지가 있다. 

각각에 대해서 살펴보겠다.

***

synchronized는 하나의 스레드만 lock을 얻어서 수행이 끝날 때까지

다른 스레드들은 대기하게 된다.

***

volatile은 변수 앞에 선언하여 사용할 수 있다.

volatile은 변수를 읽고 쓰는 과정이 모두 메인 메모리에서만 처리가 된다.

항상 cpu 메모리가 아닌 메인 메모리에서 변수를 읽어 오기 때문에 변수의 가시성 문제는 해결이 되지만

경쟁 상태 문제는 해결이 되지 않는다.

즉, volatile의 경우에는 하나의 스레드가 쓰기를 하고, 다른 스레드는 읽기만을 할 경우에 이용할 수 있다.

cpu 캐시 메모리보다 메인 메모리의 비용이 더 크기 때문에, 성능에 영향을 줄 수 있다.

***

atomic 클래스는 CAS(compare-and-swap) 기반으로 되어 있다.

값을 변경할 때, 읽었던 값을 기억하고 있다가 변경 직전에 메모리 내의 값을 확인하여

기억해 놓은 값과 같은 경우에만 처리를 하고, 그렇지 않으면 처리를 하지 않는다.

Atomic 자료형은 modern CPU가 기본적으로 제공한다고 한다.(하드웨어적으로)

따라서, synchronized를 사용하지 않고 atomic 만으로 처리가 가능하다면

성능이 더 빨라질 것으로 기대된다.

***

동기화 방법에 대해 알아본 결과, ActivityThread.java와 Instrumentation.java의 

synchronized block 들 중에서 Atomic 클래스로 바꿀 수 있는 경우가 있다면,

시도해보는 것이 좋을 것 같다.
