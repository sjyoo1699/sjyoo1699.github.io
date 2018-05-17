---
layout: post
title: "180512 TASK RECORD FINAL"
date:   2018-05-12 22:00:30 +0900
categories: jekyll update
---

TaskRecord.java의 속성 중에 inRecents 라는 Boolean 속성이 있는데, 이 변수가 뜻하는 것은, 해당 Task가 현재 내가 작업중이던 Task인지를 나타내는 변수이다. 즉 isRecents가 false이면, 해당 Task는 만들어져 있지 않은 상태라는 것이다. 이 Recents는 list라고 주석에 명시되어있는데, 해당 list는 아마도 ActivityManager에 존재할 것 같다. 그리고 Task간의 연결을 나타내는 변수들이 존재해서 Task간의 연결을 할 수 있고, Task의 생성자는 총 3개인데, 하나는 voice interactor Task를 생성하는 생성자, 하나는 activity들의 list를 가지고 있지 않은, 빈 Task의 생성자, 하나는 일반적으로 생각하는 activity의 list들과 acitivity Stack을 가지는 TaskRecord를 만드는 생성자, 이렇게 3개의 생성자가 존재한다. 나머지 다른 함수들은 Activity의 화면에서의 크기, task의 정보, 등등을 관리하는 함수들이었다.

Task Record의 구조는 이제 어느정도 명확해졌는데, activity list를 가지고 있고, 이 list들은 task가 만들어질 때,  xml 파일을 파싱해서 그 구조대로 추가되는 activity list이다. 즉, 구조대로 생성되었으니 history를 가지는 list이다.  list의 제일 마지막 원소가 root activity가 된다.

그리고 acitivity Stack을 가지고 있는데, 이는 되돌아갈 activity를 저장하는 스택이 될 것이다. 이 스택은 수시로 최신화를 해줘야 한다고 주석에 기록 되어 있다.

이렇게 TaskRecord 내부의 구조는 어느정도 명확해졌고, TaskRecord간의 관계까지 알게 되었다. 다음에는 Activity Stack의 내부 구조를 확인해봐야 좀 더 구체적으로 알 수 있을 것 같다.

```

    /** List of all activities in the task arranged in history order */
    final ArrayList<ActivityRecord> mActivities;

    /** Current stack. Setter must always be used to update the value. */
    private ActivityStack mStack;

    /** Takes on same set of values as ActivityRecord.mActivityType */
    int taskType;


    int mAffiliatedTaskId; // taskId of parent affiliation or self if no parent.
    int mAffiliatedTaskColor; // color of the parent task affiliation.
    TaskRecord mPrevAffiliate; // previous task in affiliated chain.
    int mPrevAffiliateTaskId = INVALID_TASK_ID; // previous id for persistence.
    TaskRecord mNextAffiliate; // next task in affiliated chain.
    int mNextAffiliateTaskId = INVALID_TASK_ID; // next id for persistence.


    final int taskId;       // Unique identifier for this task.
    String affinity;        // The affinity name for this task, or null; may change identity.
    String rootAffinity;    // Initial base affinity, or null; does not change from initial root.
    final IVoiceInteractionSession voiceSession;    // Voice interaction session driving task
    final IVoiceInteractor voiceInteractor;         // Associated interactor to provide to app
    Intent intent;          // The original intent that started the task.
    Intent affinityIntent;  // Intent of affinity-moved activity that started this task.
    int effectiveUid;       // The current effective uid of the identity of this task.
    ComponentName origActivity; // The non-alias activity component of the intent.
    ComponentName realActivity; // The actual activity component that started the task.
    boolean realActivitySuspended; // True if the actual activity component that started the
                                   // task is suspended.
    long firstActiveTime;   // First time this task was active.
    long lastActiveTime;    // Last time this task was active, including sleep.
    boolean inRecents;      // Actually in the recents list?
    boolean isAvailable;    // Is the activity available to be launched?
    boolean rootWasReset;   // True if the intent at the root of the task had
                            // the FLAG_ACTIVITY_RESET_TASK_IF_NEEDED flag.
    boolean autoRemoveRecents;  // If true, we should automatically remove the task from
                                // recents when activity finishes
    boolean askedCompatMode;// Have asked the user about compat mode for this task.
    boolean hasBeenVisible; // Set if any activities in the task have been visible to the user.

    String stringName;      // caching of toString() result.
    int userId;             // user for which this task was created
    boolean mUserSetupComplete; // The user set-up is complete as of the last time the task activity
                                // was changed.

    int numFullscreen;      // Number of fullscreen activities.

```


