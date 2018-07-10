---
layout: post
title: "180706 WHERE TO RUN THE ACTIVITY 01"
date:   2018-07-06 14:00:30 +0900
categories: jekyll update
---

아직 ActivityRecord.java를 마무리하지 못했는데, AcitivityRecord.java나 현재 우리 팀이 살펴보고 있는

경로(/frameworks/base/services/core/java/com/android/server/am/)에 있는 파일들이 모두

실제 어플을 구동하는 것과는 거리가 먼 것 같다고 느꼈다. 나 또한, ActivityRecord에서 해답을 찾기는 힘들거라 판단하였다.

우리 팀의 목표인 안드로이드 OS의 성능 개선을 성취하기 위해서는 Android 스케쥴링 알고리즘을 바꾸거나, 실제 앱이 구동하는 곳에서 

비효율적인 무엇인가를 찾는 것이 좋을 것이라 생각한다. 하지만, 지금까지 살펴본 결과 스케쥴링 알고리즘에서는 특별한 알고리즘이 사용된

것도 아니고, 굉장히 간단한 알고리즘이기 때문에 성능을 개선할 여지를 찾기는 힘들 것으로 보인다.

따라서, 실제 어플리케이션이 구동되는 곳을 찾아보는 것이 진행속도에 박차를 가할 수 있을 것이다 생각했다.
