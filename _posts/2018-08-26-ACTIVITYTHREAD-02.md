---
layout: post
title: "180826 ACTIVITYTHREAD 02"
date:   2018-08-26 14:00:30 +0900
categories: jekyll update
---

지난 포스팅에 이어서 ActivityThread에 대한 내용을 다룰 것이다.

Activity가 start 하는 과정에서 ActivityManager와 상호작용 하는 부분은 찾았지만,

다른 생명주기 때 상호작용하는 부분을 찾지 못하여서 이 부분에 대해 찾아볼 것이고

이 부분만 찾게 되면 이제 ActivityManager가 Activity에 대해 하는 일에 대해서는

구조적으로 그림이 그려지게 될 것 같다.

***

일단 ActivityThread에 대해 찾아본 결과 ActivityThread에 여러 다른 컴포넌트들이 존재함을

알게 되었다. 이에 대해 먼저 설명하자면,

안드로이드 UI는 기본적으로 싱글 스레드 모델로 작동한다.

따라서, 이 메인 스레드에서 긴 작업을 하는 것을 피하기 위해 여분의 스레드를 사용해야 하는데

다른 스레드에서 UI 스레드로 접근할 수 있도록 안드로이드에서 제공하는 스레드 간 통신 방법이 있다.

이에 사용하는 것이 **Looper**와 **Handler**다.

메인 스레드는 내부적으로 Looper를 가지며, 그 안에 **MessageQueue**가 포함된다.

MessageQueue는 다른 스레드나 혹은 자기 자신으로부터 받은 메세지를

선입선출 형식으로 보관하는 Queue다.

Looper는 MessageQueue에서 Message나 Runnable 객체를 차례대로 꺼내어서

Handler가 처리하도록 전달한다.

***
**Handler**
Handler는 Looper로부터 받은 Message를 실행, 처리하거나 다른 스레드로부터

메시지를 받아서 MessageQueue에 넣는 역할을 하는 스레드 간의 통신 장치다.

Handler 객체는 하나의 스레드와, 해당 스레드의 MessageQueue에 종속된다.

다른 스레드가 특정 스레드에게 메시지를 전달하려면 특정 스레드에 속한

Handler의 post나 sendMessage 등의 메소드를 호출하면 된다.

외부, 혹은 자기 스레드로부터 받은 메세지를 어떤 식으로 처리할 지는 handleMessage

메소드를 구현하여 정한다.
***
**Looper**
Looper는 무한히 루프를 돌며 자신이 속한 스레드의 Message Queue에 들어온 

Message나 Runnable 객체를 차례로 꺼내서 이를 처리할 Handler에 전달하는 역할을 한다.

메인 스레드는 Looper가 기본적으로 생성되어 있지만 새로 생성한 스레드는 기본적으로

Looper를 가지고 있지 않고 단지 run 메소드만 실행한 후 종료하기 때문에 메세지를

받을 수 없다.

따라서 기본 스레드에서 메세지를 전달받으려면 prepare 메소드를 통해 Looper를 생성하고

loop 메소드를 통해 Looper가 무한히 루프를 돌며 MessageQueue에 쌓인 Message나 Runnable

객체를 꺼내 Handler에 전달하도록 한다.

***
**Message & Runnable**
Message란 스레드 간 통신할 내용을 담는 객체이자 Queue에 들어갈 일의 단위로 Handler를

통해 보낼 수 있다. 일반적으로 Message가 필요할 때 새 Message 객체를 생성하면

성능 이슈가 생길 수 있으므로 안드로이드가 시스템에 만들어 둔 Message Pool의 객체를

재사용한다.

Runnable은 스레드의 run 메소드를 분리한 것이다. 따라서 Runnable 인터페이스는

run 메소드를 추상 메소드로 가지고 있으므로 상속받은 클래스는 run 코드를 구현해야 한다.

Message가 Object 같은 스레드 간 통신할 내용을 담으면, Runnable은 실행할

메소드와 그 내부에서 실행될 코드를 담는다.

***

위의 내용들을 기본적으로 숙지한 채로 ActivtyThread를 살펴보았다.

