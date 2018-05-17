---
layout: post
title: "180517 ACTIVITY RECORD FIRST"
date:   2018-05-17 18:00:30 +0900
categories: jekyll update
---

Activity Record.java를 처음 살펴보았다. 처음 보는 것이라, 클래스 내부의 속성들을 먼저 봤는데,

ActivityRecord 객체가 ActivityManagerService 객체를 참조하고 있는 것을 발견하였다.

ActivityManagerService가 ActivityRecord 객체를 생성하고 소유하는데, ActivityRecord도 ActivityManagerService를 소유하면서 서로 양방향으로 참조를 하고 있다.

이런 구조가 되면, 좋지 않을꺼라 생각이 드는데, 이렇게 만든 이유를 아직 모르기 때문에

함수 구현 부분을 보며 이런 구조를 택한 이유를 봐야할 것 같다. 그 부분을 보고 다른 구조를 생각해볼 수 있다면 생각해보면 좋을 것 같다.

```
    final ActivityManagerService service; // owner
    final IApplicationToken.Stub appToken; // window manager token
    AppWindowContainerController mWindowContainerController;
    final ActivityInfo info; // all about me
    final ApplicationInfo appInfo; // information about activity's app
    final int launchedFromPid; // always the pid who started the activity.
    final int launchedFromUid; // always the uid who started the activity.
    final String launchedFromPackage; // always the package who started the activity.
    final int userId;          // Which user is this running for?
    final Intent intent;    // the original intent that generated us
    final ComponentName realActivity;  // the intent component, or target of an alias.
    final String shortComponentName; // the short component name of the intent
    final String resolvedType; // as per original caller;
    final String packageName; // the package implementing intent's component
    final String processName; // process where this component wants to run
    final String taskAffinity; // as per ActivityInfo.taskAffinity
    final boolean stateNotNeeded; // As per ActivityInfo.flags
    boolean fullscreen; // covers the full screen?
    final boolean noDisplay;  // activity is not displayed?
    private final boolean componentSpecified;  // did caller specify an explicit component?
    final boolean rootVoiceInteraction;  // was this the root activity of a voice interaction?
```


