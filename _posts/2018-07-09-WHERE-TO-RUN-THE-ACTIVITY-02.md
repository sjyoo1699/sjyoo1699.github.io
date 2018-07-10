---
layout: post
title: "180709 WHERE TO RUN THE ACTIVITY 02"
date:   2018-07-09 14:00:30 +0900
categories: jekyll update
---

Activity를 스타트하는 방법은 여러가지가 있겠지만, 일반적으로 개발자가 개발한 Application에서는 Acitivity에서 startActivity라는

함수 호출을 통해 다른 액티비티를 시작한다.

따라서, 먼저 저 함수를 찾아보고자 한다.

startActivity 함수는 우리가 개발할 때 작성하는 Activity 클래스 내부에서 사용할 수 있는데,

이 클래스는 Activity클래스를 상속받는다. Activity의 위치는 

framewokrk/base/core/java/android/app/Activity.java

이곳이다. 이곳에서 startActivity 함수를 찾아보았다.

```
     /**
     * Launch a new activity.  You will not receive any information about when
     * the activity exits.  This implementation overrides the base version,
     * providing information about
     * the activity performing the launch.  Because of this additional
     * information, the {@link Intent#FLAG_ACTIVITY_NEW_TASK} launch flag is not
     * required; if not specified, the new activity will be added to the
     * task of the caller.
     *
     * <p>This method throws {@link android.content.ActivityNotFoundException}
     * if there was no Activity found to run the given Intent.
     *
     * @param intent The intent to start.
     * @param options Additional options for how the Activity should be started.
     * See {@link android.content.Context#startActivity(Intent, Bundle)}
     * Context.startActivity(Intent, Bundle)} for more details.
     *
     * @throws android.content.ActivityNotFoundException
     *
     * @see #startActivity(Intent)
     * @see #startActivityForResult
     */
    @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }
```

위의 코드가 startActivity의 함수 설명과 함수 구현부분이다. 내부를 보면 startActivity는

startActivityForResult를 default 값을 넣어서 호출하고 있다.

아래는 startActivityForResult 함수이다.

```
    /**
     * Launch an activity for which you would like a result when it finished.
     * When this activity exits, your
     * onActivityResult() method will be called with the given requestCode.
     * Using a negative requestCode is the same as calling
     * {@link #startActivity} (the activity is not launched as a sub-activity).
     *
     * <p>Note that this method should only be used with Intent protocols
     * that are defined to return a result.  In other protocols (such as
     * {@link Intent#ACTION_MAIN} or {@link Intent#ACTION_VIEW}), you may
     * not get the result when you expect.  For example, if the activity you
     * are launching uses {@link Intent#FLAG_ACTIVITY_NEW_TASK}, it will not
     * run in your task and thus you will immediately receive a cancel result.
     *
     * <p>As a special case, if you call startActivityForResult() with a requestCode
     * >= 0 during the initial onCreate(Bundle savedInstanceState)/onResume() of your
     * activity, then your window will not be displayed until a result is
     * returned back from the started activity.  This is to avoid visible
     * flickering when redirecting to another activity.
     *
     * <p>This method throws {@link android.content.ActivityNotFoundException}
     * if there was no Activity found to run the given Intent.
     *
     * @param intent The intent to start.
     * @param requestCode If >= 0, this code will be returned in
     *                    onActivityResult() when the activity exits.
     * @param options Additional options for how the Activity should be started.
     * See {@link android.content.Context#startActivity(Intent, Bundle)}
     * Context.startActivity(Intent, Bundle)} for more details.
     *
     * @throws android.content.ActivityNotFoundException
     *
     * @see #startActivity
     */
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                // If this start is requesting a result, we can avoid making
                // the activity visible until the result is received.  Setting
                // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
                // activity hidden during this time, to avoid flickering.
                // This can only be done when a result is requested because
                // that guarantees we will get information back when the
                // activity is finished, no matter what happens to it.
                mStartedActivity = true;
            }
            cancelInputsAndStartExitTransition(options);
            // TODO Consider clearing/flushing other event sources and events for child windows.
        } else {
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                // Note we want to go through this method for compatibility with
                // existing applications that may have overridden it.
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }
```

위의 코드에서 mParent는 Activity이다.

Instrumentation 클래스와 ActivityThread(mMainThread) 클래스는 무엇인지 아직 모르겠지만, 

Instrumentation의 하위 ActivityResult 객체를 만들어서 Thread에 보내는데,

이 ActivityResult가 실제 Activity를 뜻하는 것 같다.

따라서, Instrumentaion 클래스와 AcitivityThread 클래스를 살펴봐야 할 것 같다.

특히 두 클래스가 위 코드와 같이 활용되는 부분이 Activity.java의 전반적인 startActivity 계열

함수에 모두 사용이 되고 있다.

먼저 Instrumentation 클래스에 대해 android developers에서 찾아보면 이러한 설명이 나온다.

Base class for implementing application instrumentation code. 
When running with instrumentation turned on, 
this class will be instantiated for you before any of the application code, 
allowing you to monitor all of the interaction the system has with the application. 
An Instrumentation implementation is described to the system through an AndroidManifest.xml's <instrumentation> tag.
  
그리고 이 클래스의 nested class로는

Instrumentation.ActivityMonitor

Information about a particular kind of Intent that is being monitored. 

Instrumentation.ActivityResult

Description of a Activity execution result to return to the original activity.

위 2개가 있다.

Instrumentation 과 ActivityThread 클래스의 코드를 살펴보고 다음 블로그에 자세히 포스팅하겠다.

<https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/Instrumentation.java>
<https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/ActivityThread.java>
