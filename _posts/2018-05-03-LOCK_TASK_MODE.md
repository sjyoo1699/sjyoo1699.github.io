---
layout: post
title: 180503
---

<h3> LOCK_TASK_MODE 상태의 의미는 무엇일까 </h3>

/frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java에 선언된 부분이다.

```
import static android.app.ActivityManager.LOCK_TASK_MODE_LOCKED;
import static android.app.ActivityManager.LOCK_TASK_MODE_NONE;
import static android.app.ActivityManager.LOCK_TASK_MODE_PINNED;
```

