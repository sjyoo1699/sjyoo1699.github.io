---
layout: post
title: "180715 STRUCTURE OF ACTIVITY MANAGER"
date:   2018-07-15 14:00:30 +0900
categories: jekyll update
---

지금까지 ActivityManager의 구조와 역할에 대해 알기 위해서 많은 노력을 하였다. 그리고 어느 정도 구조가 정리가 되었는데,

그 구조가 명확하지는 않았다.

하지만 이번에 우리 프로젝트의 멘토님(이원영 선배님)을 만나 이야기를 하고 ACTIVITY MANAGER의 구조와 여러 가지 애매했던 것들이 명확해졌다.

처음에 시스템이 부팅되면, 시스템 서비스가 시작되고 시스템 서비스는 ActivityManager를 실행하는데

ActivityManager의 구조는 아래와 같다.


![activitymanager](https://user-images.githubusercontent.com/28890428/42816029-4df84bbe-8a04-11e8-8c7b-abaea4390f38.PNG)


<ActivityManager의 구조>

위의 그림에서 TaskRecord와 ActivityRecord가 나오는데, 

지금까지는 Task 하나에 해당 Task의 정보를 가지고 있는 TaskRecord가 바인딩되고,

Activity도 마찬가지로 Activity 하나와 ActivityRecord가 하나씩 바인딩 되어 있는 것으로 알고 있었다.

하지만 멘토님과 이야기해본 결과 ActivityRecord, TaskRecord는 그 자체로 하나의 Task와 Activity인 것이고,

원칙적으로는 하나의 어플리케이션은 하나의 Task이다. 

하지만, Task의 플래그를 다른 Task에서 사용할 수 있도록 하면

하나의 Task에 두 개의 어플리케이션이 들어가게 되는 경우도 있다.

이 ActivityRecord를 ActivityManager가 WindowManager에게 그려달라고 보내면 WindowManager가 화면에 그려주게 된다.



***



현재 우리가 살펴보고 있는 경로는 

frameworks/base/core/java/android/app

frameworks/base/services/core/java/com/android/server/am

이렇게 두 개가 있는데, 이 경로에 대해서도 명확한 답을 들을 수 있었다.

소스코드가 워낙 방대하다 보니, 우리가 현재 보고 있는 경로가 제대로 보고 있는 게 맞는 지 의구심이 들었는데 

위의 두 경로 app 부분이 개발자가 만드는 어플의 실제 뼈대가 되는 부분이고, am이 Activity Manager에 대한 부분으로

저 두 경로를 살펴보면 된다고 말씀해주셔서, 우리의 프로젝트의 진행방향이 조금 더 확실해졌다.
