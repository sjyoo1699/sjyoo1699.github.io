---
layout: post
title: "180717 WHERE TO RUN THE ACTIVITY 03"
date:   2018-07-17 14:00:30 +0900
categories: jekyll update
---

저번 포스팅에 이어서, **Instrumentaion** 클래스의 nested class **ActivityResult** 클래스의 

**execStartActivity** 함수를 살펴보았다.

frameworks/base/core/java/android/app/Instrumentaion.java

먼저 nested class인 ActivityResult의 모습이다.

```
    /**
     * Description of a Activity execution result to return to the original
     * activity.
     */
    public static final class ActivityResult {
        /**
         * Create a new activity result.  See {@link Activity#setResult} for 
         * more information. 
         *  
         * @param resultCode The result code to propagate back to the
         * originating activity, often RESULT_CANCELED or RESULT_OK
         * @param resultData The data to propagate back to the originating
         * activity.
         */
        public ActivityResult(int resultCode, Intent resultData) {
            mResultCode = resultCode;
            mResultData = resultData;
        }
        /**
         * Retrieve the result code contained in this result.
         */
        public int getResultCode() {
            return mResultCode;
        }
        /**
         * Retrieve the data contained in this result.
         */
        public Intent getResultData() {
            return mResultData;
        }
        private final int mResultCode;
        private final Intent mResultData;
    }
```

내부 속성으로 mResultCode와 mResultData를 가지고 있다.

***

아래는 execStartActivity의 모습이다.

```
    /**
     * Execute a startActivity call made by the application.  The default 
     * implementation takes care of updating any active {@link ActivityMonitor}
     * objects and dispatches this call to the system activity manager; you can
     * override this to watch for the application to start an activity, and 
     * modify what happens when it does. 
     *
     * <p>This method returns an {@link ActivityResult} object, which you can 
     * use when intercepting application calls to avoid performing the start 
     * activity action but still return the result the application is 
     * expecting.  To do this, override this method to catch the call to start 
     * activity so that it returns a new ActivityResult containing the results 
     * you would like the application to see, and don't call up to the super 
     * class.  Note that an application is only expecting a result if 
     * <var>requestCode</var> is &gt;= 0.
     *
     * <p>This method throws {@link android.content.ActivityNotFoundException}
     * if there was no Activity found to run the given Intent.
     *
     * @param who The Context from which the activity is being started.
     * @param contextThread The main thread of the Context from which the activity
     *                      is being started.
     * @param token Internal token identifying to the system who is starting 
     *              the activity; may be null.
     * @param target Which activity is performing the start (and thus receiving 
     *               any result); may be null if this call is not being made
     *               from an activity.
     * @param intent The actual Intent to start.
     * @param requestCode Identifier for this request's result; less than zero 
     *                    if the caller is not expecting a result.
     * @param options Addition options.
     *
     * @return To force the return of a particular result, return an 
     *         ActivityResult object containing the desired data; otherwise
     *         return null.  The default implementation always returns null.
     *
     * @throws android.content.ActivityNotFoundException
     *
     * @see Activity#startActivity(Intent)
     * @see Activity#startActivityForResult(Intent, int)
     * @see Activity#startActivityFromChild
     *
     * {@hide}
     */
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        Uri referrer = target != null ? target.onProvideReferrer() : null;
        if (referrer != null) {
            intent.putExtra(Intent.EXTRA_REFERRER, referrer);
        }
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    ActivityResult result = null;
                    if (am.ignoreMatchingSpecificIntents()) {
                        result = am.onStartActivity(intent);
                    }
                    if (result != null) {
                        am.mHits++;
                        return result;
                    } else if (am.match(who, null, intent)) {
                        am.mHits++;
                        if (am.isBlocking()) {
                            return requestCode >= 0 ? am.getResult() : null;
                        }
                        break;
                    }
                }
            }
        }
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            int result = ActivityManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }

```

위의 코드를 보면 ActivityMonitor의 함수를 통해 ActivityResult를 만들고 리턴하는 경우와,

ActivityManager를 호출하여 int 형의 result라는 코드를 만들어낸 후, checkStartActivityResult의

인자로 주어서 호출한 뒤, ActivityResult는 null을 리턴하는 경우가 있다.

먼저, 첫번 째 경우에 대해 알아 보기 위해 AcivityMonitor의 코드를 살펴보았다.

***
```
    /**
     * Information about a particular kind of Intent that is being monitored.
     * An instance of this class is added to the 
     * current instrumentation through {@link #addMonitor}; after being added, 
     * when a new activity is being started the monitor will be checked and, if 
     * matching, its hit count updated and (optionally) the call stopped and a 
     * canned result returned.
     * 
     * <p>An ActivityMonitor can also be used to look for the creation of an
     * activity, through the {@link #waitForActivity} method.  This will return
     * after a matching activity has been created with that activity object.
     */
    public static class ActivityMonitor {
        private final IntentFilter mWhich;
        private final String mClass;
        private final ActivityResult mResult;
        private final boolean mBlock;
        private final boolean mIgnoreMatchingSpecificIntents;
        // This is protected by 'Instrumentation.this.mSync'.
        /*package*/ int mHits = 0;
        // This is protected by 'this'.
        /*package*/ Activity mLastActivity = null;
        /**
         * Create a new ActivityMonitor that looks for a particular kind of 
         * intent to be started.
         *
         * @param which The set of intents this monitor is responsible for.
         * @param result A canned result to return if the monitor is hit; can 
         *               be null.
         * @param block Controls whether the monitor should block the activity 
         *              start (returning its canned result) or let the call
         *              proceed.
         *  
         * @see Instrumentation#addMonitor 
         */
        public ActivityMonitor(
            IntentFilter which, ActivityResult result, boolean block) {
            mWhich = which;
            mClass = null;
            mResult = result;
            mBlock = block;
            mIgnoreMatchingSpecificIntents = false;
        }
        /**
         * Create a new ActivityMonitor that looks for a specific activity 
         * class to be started. 
         *  
         * @param cls The activity class this monitor is responsible for.
         * @param result A canned result to return if the monitor is hit; can 
         *               be null.
         * @param block Controls whether the monitor should block the activity 
         *              start (returning its canned result) or let the call
         *              proceed.
         *  
         * @see Instrumentation#addMonitor 
         */
        public ActivityMonitor(
            String cls, ActivityResult result, boolean block) {
            mWhich = null;
            mClass = cls;
            mResult = result;
            mBlock = block;
            mIgnoreMatchingSpecificIntents = false;
        }
        /**
         * Create a new ActivityMonitor that can be used for intercepting any activity to be
         * started.
         *
         * <p> When an activity is started, {@link #onStartActivity(Intent)} will be called on
         * instances created using this constructor to see if it is a hit.
         *
         * @see #onStartActivity(Intent)
         */
        public ActivityMonitor() {
            mWhich = null;
            mClass = null;
            mResult = null;
            mBlock = false;
            mIgnoreMatchingSpecificIntents = true;
        }
```
위의 설명을 보면, ActivityMonitor는 새로운 Activity가 시작될 때, 체크되고, 매칭이 된다면 

hitCount를 업데이트 하고, result가 return 된다고 한다.

위의 코드는 ActivityMonitor의 속성 부분과 Constructor 부분이다. 아래의 Constructor가 default이고,

mIgnoreMatchingSpecificIntents 속성이 true가 되는 Constructor이다.

해당 Constructor의 설명을 보면

***
Create a new ActivityMonitor that can be used for intercepting any activity to be started.
***

이렇게 나와 있는데, ActivityMonitor를 체크하다가 위의 컨스트럭터로 만든 ActivityMonitor를 만나게 되면

ActivityResult를 바로 리턴해버린다.

아래는 위의 상황에서 사용되는 함수이다.

```
        /**
         * Used for intercepting any started activity.
         *
         * <p> A non-null return value here will be considered a hit for this monitor.
         * By default this will return {@code null} and subclasses can override this to return
         * a non-null value if the intent needs to be intercepted.
         *
         * <p> Whenever a new activity is started, this method will be called on instances created
         * using {@link #Instrumentation.ActivityMonitor()} to check if there is a match. In case
         * of a match, the activity start will be blocked and the returned result will be used.
         *
         * @param intent The intent used for starting the activity.
         * @return The {@link ActivityResult} that needs to be used in case of a match.
         */
        public ActivityResult onStartActivity(Intent intent) {
            return null;
        }
```

ActivityMonitor를 살펴본 결과 execStartActivity의 두 경로 중 ActivityMonior를 사용하는 경로는

이미 만들어져서 실행 중이었다가, 중지된 Activity를 찾아서 다시 실행시키는 데 사용하는 것 같고,

ActivityManager에 요청하는 경로가 아예 새로운 Activity를 만드는 것 같다.

***

이제 이렇게 해서 만들어낸 ActivityResult를 thread에 올리는 작업을 하게 될텐데,

멘토님께서 말씀하시길 ActivityManager와 개발자가 만드는 액티비티의 연결고리가 thread 부분에서 

끊어지기 때문에 그 부분이 보기 힘들거라고 하셨다.

따라서 thread 부분을 잘 살펴봐야 할 것 같고, 다음 포스팅에서는 

ActivityManager의 startActivity 함수와 checkStartActivity 함수에 대해서 알아보고,

thread 부분에 관해서 다루겠다.
