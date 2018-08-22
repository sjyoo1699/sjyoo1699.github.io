---
layout: post
title: "180819 ACTIVITYTHREAD AND INSTRUMENTATION"
date:   2018-08-19 14:00:30 +0900
categories: jekyll update
---

개발자가 구현하는 Activity 단에서부터 Activity가 instance화 되고 실행되는 부분을 추적하였다.

이 과정에서 ActivityManager까지 연결되는 연결부분을 찾기가 힘들었고, WHERE TO RUN THE ACTIVITY 04

포스팅에서 해당 부분을 찾았었다. 

startActivity 함수 콜을 따라간 것이었는데 단순히 start 부분이 아닌 스케쥴링 과정에서

상호 작용하는 부분을 찾고자 ActivityThread와 Instrumentation의 코드를 보았다.

***

