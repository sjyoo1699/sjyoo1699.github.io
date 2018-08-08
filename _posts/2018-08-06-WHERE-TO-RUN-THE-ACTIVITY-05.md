---
layout: post
title: "180806 WHERE TO RUN THE ACTIVITY 05"
date:   2018-08-06 14:00:30 +0900
categories: jekyll update
---

저번 포스팅에서 실제 Activity가 만들어지고 실행되는 부분을 찾기 위해, 추적을 하였지만

깔끔하게 그림이 그려지지도 않았고, 매끄럽게 연결되지 않는 듯한 느낌이었다.

멘토님이 해주신 말씀이 생각이 났는데, Activity가 start 되는 과정이 thread 부분에서

끊기기 때문에, 이 부분을 잘 찾아야 할 것이라고 하셨다.

따라서, 이번 포스팅에서는 **ActivityThread.java**를 살펴보았다.

***

ActivityThread.java를 살펴보던 중 아래와 같은 함수를 발견하였다.

```
    /**  Core implementation of activity launch. */
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ActivityInfo aInfo = r.activityInfo;
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);
        }
        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }
        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }
        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            if (localLOGV) Slog.v(TAG, "Performing launch of " + r);
            if (localLOGV) Slog.v(
                    TAG, r + ": app=" + app
                    + ", appName=" + app.getPackageName()
                    + ", pkg=" + r.packageInfo.getPackageName()
                    + ", comp=" + r.intent.getComponent().toShortString()
                    + ", dir=" + r.packageInfo.getAppDir());
            if (activity != null) {
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (r.overrideConfig != null) {
                    config.updateFrom(r.overrideConfig);
                }
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                checkAndBlockForNetworkAccess();
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }
                activity.mCalled = false;
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                if (!activity.mCalled) {
                    throw new SuperNotCalledException(
                        "Activity " + r.intent.getComponent().toShortString() +
                        " did not call through to super.onCreate()");
                }
                r.activity = activity;
            }
            r.setState(ON_CREATE);
            mActivities.put(r.token, r);
        } catch (SuperNotCalledException e) {
            throw e;
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to start activity " + component
                    + ": " + e.toString(), e);
            }
        }
        return activity;
    }
```

주석으로 Core implementation of activity launch. 라고 적혀있다.

이 함수를 찾기까지 힘들었고, 찾고나서 매우 기뻤다.

위 함수가 실제 Activity를 instance화 시키는 코드다.

우선 위 함수의 인자를 보면, ActivityClientRecord와 Intent를 받는다.

ActivityClientRecord는 ActivityThread.java의 nested class인데, 그 선언은 아래와 같다.

***
```
    /** Activity client record, used for bookkeeping for the real {@link Activity} instance. */
    public static final class ActivityClientRecord {
        public IBinder token;
        int ident;
        Intent intent;
        String referrer;
        IVoiceInteractor voiceInteractor;
        Bundle state;
        PersistableBundle persistentState;
        Activity activity;
        Window window;
        Activity parent;
        String embeddedID;
        Activity.NonConfigurationInstances lastNonConfigurationInstances;
        // TODO(lifecycler): Use mLifecycleState instead.
        boolean paused;
        boolean stopped;
        boolean hideForNow;
        Configuration newConfig;
        Configuration createdConfig;
        Configuration overrideConfig;
        // Used for consolidating configs before sending on to Activity.
        private Configuration tmpConfig = new Configuration();
        // Callback used for updating activity override config.
        ViewRootImpl.ActivityConfigCallback configCallback;
        ActivityClientRecord nextIdle;
        ProfilerInfo profilerInfo;
        ActivityInfo activityInfo;
        CompatibilityInfo compatInfo;
        public LoadedApk packageInfo;
        List<ResultInfo> pendingResults;
        List<ReferrerIntent> pendingIntents;
        boolean startsNotResumed;
        public final boolean isForward;
        int pendingConfigChanges;
        Window mPendingRemoveWindow;
        WindowManager mPendingRemoveWindowManager;
        boolean mPreserveWindow;
        @LifecycleState
        private int mLifecycleState = PRE_ON_CREATE;
        @VisibleForTesting
        public ActivityClientRecord() {
            this.isForward = false;
            init();
        }
        public ActivityClientRecord(IBinder token, Intent intent, int ident,
                ActivityInfo info, Configuration overrideConfig, CompatibilityInfo compatInfo,
                String referrer, IVoiceInteractor voiceInteractor, Bundle state,
                PersistableBundle persistentState, List<ResultInfo> pendingResults,
                List<ReferrerIntent> pendingNewIntents, boolean isForward,
                ProfilerInfo profilerInfo, ClientTransactionHandler client) {
            this.token = token;
            this.ident = ident;
            this.intent = intent;
            this.referrer = referrer;
            this.voiceInteractor = voiceInteractor;
            this.activityInfo = info;
            this.compatInfo = compatInfo;
            this.state = state;
            this.persistentState = persistentState;
            this.pendingResults = pendingResults;
            this.pendingIntents = pendingNewIntents;
            this.isForward = isForward;
            this.profilerInfo = profilerInfo;
            this.overrideConfig = overrideConfig;
            this.packageInfo = client.getPackageInfoNoCheck(activityInfo.applicationInfo,
                    compatInfo);
            init();
        }
        /** Common initializer for all constructors. */
        private void init() {
            parent = null;
            embeddedID = null;
            paused = false;
            stopped = false;
            hideForNow = false;
            nextIdle = null;
            configCallback = (Configuration overrideConfig, int newDisplayId) -> {
                if (activity == null) {
                    throw new IllegalStateException(
                            "Received config update for non-existing activity");
                }
                activity.mMainThread.handleActivityConfigurationChanged(token, overrideConfig,
                        newDisplayId);
            };
        }
        /** Get the current lifecycle state. */
        public int getLifecycleState() {
            return mLifecycleState;
        }
        /** Update the current lifecycle state for internal bookkeeping. */
        public void setState(@LifecycleState int newLifecycleState) {
            mLifecycleState = newLifecycleState;
            switch (mLifecycleState) {
                case ON_CREATE:
                    paused = true;
                    stopped = true;
                    break;
                case ON_START:
                    paused = true;
                    stopped = false;
                    break;
                case ON_RESUME:
                    paused = false;
                    stopped = false;
                    break;
                case ON_PAUSE:
                    paused = true;
                    stopped = false;
                    break;
                case ON_STOP:
                    paused = true;
                    stopped = true;
                    break;
            }
        }
        private boolean isPreHoneycomb() {
            return activity != null && activity.getApplicationInfo().targetSdkVersion
                    < android.os.Build.VERSION_CODES.HONEYCOMB;
        }
        private boolean isPreP() {
            return activity != null && activity.getApplicationInfo().targetSdkVersion
                    < android.os.Build.VERSION_CODES.P;
        }
        public boolean isPersistable() {
            return activityInfo.persistableMode == ActivityInfo.PERSIST_ACROSS_REBOOTS;
        }
        public boolean isVisibleFromServer() {
            return activity != null && activity.mVisibleFromServer;
        }
        public String toString() {
            ComponentName componentName = intent != null ? intent.getComponent() : null;
            return "ActivityRecord{"
                + Integer.toHexString(System.identityHashCode(this))
                + " token=" + token + " " + (componentName == null
                        ? "no component name" : componentName.toShortString())
                + "}";
        }
        public String getStateString() {
            StringBuilder sb = new StringBuilder();
            sb.append("ActivityClientRecord{");
            sb.append("paused=").append(paused);
            sb.append(", stopped=").append(stopped);
            sb.append(", hideForNow=").append(hideForNow);
            sb.append(", startsNotResumed=").append(startsNotResumed);
            sb.append(", isForward=").append(isForward);
            sb.append(", pendingConfigChanges=").append(pendingConfigChanges);
            sb.append(", preserveWindow=").append(mPreserveWindow);
            if (activity != null) {
                sb.append(", Activity{");
                sb.append("resumed=").append(activity.mResumed);
                sb.append(", stopped=").append(activity.mStopped);
                sb.append(", finished=").append(activity.isFinishing());
                sb.append(", destroyed=").append(activity.isDestroyed());
                sb.append(", startedActivity=").append(activity.mStartedActivity);
                sb.append(", temporaryPause=").append(activity.mTemporaryPause);
                sb.append(", changingConfigurations=").append(activity.mChangingConfigurations);
                sb.append("}");
            }
            sb.append("}");
            return sb.toString();
        }
    }
```

지금까지 살펴보았던 ActivityRecord의 개념과 비슷한 개념인 것으로 보이고,

추가적으로 액티비티의 생명주기를 가지고 있다.

***
다시 performLaunchActivity 함수로 돌아가면,

Activity를 실행하기 위한 정보를 ActivityClientRecord 객체를 통해서 받는다.

그리고 첫 번째 try, catch 문을 보면 이전에도 다루었던,

Instrumentation이 나온다.

이 부분에서 이어지는 듯 해서 Instrumentation.newActivity 함수를 찾아보았다.

***
아래는 Instrumentation 클래스의 newActivity 함수이다.

```
    /**
     * Perform instantiation of the process's {@link Activity} object.  The
     * default implementation provides the normal system behavior.
     * 
     * @param cl The ClassLoader with which to instantiate the object.
     * @param className The name of the class implementing the Activity
     *                  object.
     * @param intent The Intent object that specified the activity class being
     *               instantiated.
     * 
     * @return The newly instantiated Activity object.
     */
    public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
        String pkg = intent != null && intent.getComponent() != null
                ? intent.getComponent().getPackageName() : null;
        return getFactory(pkg).instantiateActivity(cl, className, intent);
    }
```

위 함수를 보면 getFactory 라는 함수에 인자로 package name을 전달하고 리턴받은 객체의

instantiateActivity라는 함수를 호출한다.

먼저, getFactory라는 함수는 아래와 같다.

```
    private AppComponentFactory getFactory(String pkg) {
        if (pkg == null) {
            Log.e(TAG, "No pkg specified, disabling AppComponentFactory");
            return AppComponentFactory.DEFAULT;
        }
        if (mThread == null) {
            Log.e(TAG, "Uninitialized ActivityThread, likely app-created Instrumentation,"
                    + " disabling AppComponentFactory", new Throwable());
            return AppComponentFactory.DEFAULT;
        }
        LoadedApk apk = mThread.peekPackageInfo(pkg, true);
        // This is in the case of starting up "android".
        if (apk == null) apk = mThread.getSystemContext().mPackageInfo;
        return apk.getAppFactory();
    }
```

리턴 값이 AppComponentFactory 객체이다.

아래는 AppComponentFactory에 대해서 AndroidDevelopers에서 찾아본 내용이다.

***
Interface used to control the instantiation of manifest elements.

See also:

instantiateApplication(ClassLoader, String)
instantiateActivity(ClassLoader, String, Intent)
instantiateService(ClassLoader, String, Intent)
instantiateReceiver(ClassLoader, String, Intent)
instantiateProvider(ClassLoader, String)
***

아래는 AppComnentActivity 클래스의 instatiateActivity 함수이다

```
    /**
     * Allows application to override the creation of activities. This can be used to
     * perform things such as dependency injection or class loader changes to these
     * classes.
     * <p>
     * This method is only intended to provide a hook for instantiation. It does not provide
     * earlier access to the Activity object. The returned object will not be initialized
     * as a Context yet and should not be used to interact with other android APIs.
     *
     * @param cl        The default classloader to use for instantiation.
     * @param className The class to be instantiated.
     * @param intent    Intent creating the class.
     */
    public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
            @Nullable Intent intent)
            throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        return (Activity) cl.loadClass(className).newInstance();
    }
```

클래스로더를 통해서 클래스를 만들어주고 리턴하는데, Activity 클래스를 리턴한다.

다시 제일 처음 함수로 돌아가면, 최종적으로 Activity 객체가 리턴된다.

제일 처음 함수의 다음을 보면 StrictMode의 incrementExpectedActivityCount 함수를 호출한다.

```
    /** @hide */
    public static void incrementExpectedActivityCount(Class klass) {
        if (klass == null) {
            return;
        }
        synchronized (StrictMode.class) {
            if ((sVmPolicy.mask & DETECT_VM_ACTIVITY_LEAKS) == 0) {
                return;
            }
            Integer expected = sExpectedActivityInstanceCount.get(klass);
            Integer newExpected = expected == null ? 1 : expected + 1;
            sExpectedActivityInstanceCount.put(klass, newExpected);
        }
    }
```

위에서 sExpectedActivityInstacneCount는 해쉬맵인데, 클래스와 Integer 값인 expected를 저장한다.

액티비티에 번호를 매기고 순서대로 실행될 수 있도록 저장하는 것 같다.

그 밑의 intent 관련 함수들은 아직 찾지 못하여서, 다음 포스팅에서 다루도록 하겠다.
