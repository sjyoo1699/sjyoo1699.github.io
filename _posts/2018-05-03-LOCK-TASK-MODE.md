---
layout: post
title: "180503 <LOCK_TASK_MODE>"
date:   2018-05-03 20:06:30 +0900
categories: jekyll update
---
/frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java에 선언된 부분이다.

```
import static android.app.ActivityManager.LOCK_TASK_MODE_LOCKED;
import static android.app.ActivityManager.LOCK_TASK_MODE_NONE;
import static android.app.ActivityManager.LOCK_TASK_MODE_PINNED;
```

위에 선언된 부분을 ActivityManager 클래스에 가서 주석을 확인해보았는데, 한번에 알아보기가 힘들었고, 이 부분에 대해서 추측하기를 태스크가 포커스를 가지고 있지 않으면 LOCKED 된 상태가 되고, 포커스를 가지고 있지 않지만 실행 중인 태스크라면 PINNED가 되는 것이 아닐까 하고 추측하였다. 이 부분에 대해서는 ActivityManagerSupervisor.java를 조금 더 봐야 알 것 같다.
