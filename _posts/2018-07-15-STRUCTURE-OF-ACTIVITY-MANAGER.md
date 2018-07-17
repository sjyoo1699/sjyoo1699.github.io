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

