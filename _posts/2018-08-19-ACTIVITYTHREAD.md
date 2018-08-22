---
layout: post
title: "180819 ACTIVITYTHREAD"
date:   2018-08-19 14:00:30 +0900
categories: jekyll update
---

개발자가 만든 Activity에서부터 시작하여 ActivityManager로 명령이 전달되기까지의 과정을 찾다가,

WHERE TO RUN THE ACTIVITY 04 포스팅에서 execStartActivity 함수 내에서 한번 ActivityManager의 함수를

콜하는 부분을 찾았었다.

이 부분은 단순히 start 할 때 뿐이라서 다른 과정에서 ActivityManager와 상호작용 하는 부분을 찾고자

ActivityThread와 Instrumentation 코드를 보았다.

위의 execStartActivity 함수는 Instrumentation.java 내부에 있는 함수였는데,

이 함수 이외에 ActivityManager와 상호작용하는 부분은 존재하지 않았다.

따라서, ActivityThread 코드를 보니 ActivityManager 객체를 사용하고 있었다.

여기서 사용하는 ActivityManager는 am 패키지 내부에 있는 ActivityManager는 아니고

ActivityThread.java와 같은 패키지 내에 있는 ActivityManager.java 클래스이다.

그리고 ActivityThread 내부에서 IActivityManager라는 인터페이스와 ActivityManagerNative라는 

객체를 찾았다. ActivityManagerNative는 IActivityManager를 구현한 클래스이다.

ActivityManager와의 연결고리를 찾기 위해 굉장히 많은 시간을 쓰고 헤맸다.

생명주기에 관련해서 해당 생명주기 때 ActivityManager와 상호작용을 무조건 할 것이라 생각해서,

생명주기 관련 함수들을 Activity부터 쫓아가 보고, Instrumentation의 생명주기 함수들을

다 따라가 봐도, ActivityManager와의 연결고리는 찾을 수 없었다.

그러던 중 IActivityManager 인터페이스와 ActivityManagerNative 클래스를 찾았는데,

IActivityManager가 ActivityManager를 사용할 수 있게 해주는 인터페이스로,

사실상 실제 ActivityManager로 봐도 될 것 같다.

ActivityThread 내부에서 사용되는 코드이다.

***
```
IActivityManager am = ActivityManager.getService();
```

위와 같은 방식으로 ActivityManager를 이용하고 있다.

아래는 ActivityManager의 getService() 함수이다.

```
    /**
     * @hide
     */
    @UnsupportedAppUsage
    public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }
```

```
    @UnsupportedAppUsage
    private static final Singleton<IActivityManager> IActivityManagerSingleton =
            new Singleton<IActivityManager>() {
                @Override
                protected IActivityManager create() {
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };
```

위와 같이 IActivityManager 클래스는 ActivityManager 를 사용할 수 있게 해주는 인터페이스인 것으로 보인다.

***

아래는 ActivityThread 내부에 있는 handleMessage 함수이다.

```
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    Looper.myLooper().quit();
                    break;
                case RECEIVER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastReceiveComp");
                    handleReceiver((ReceiverData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CREATE_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceCreate: " + String.valueOf(msg.obj)));
                    handleCreateService((CreateServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UNBIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceUnbind");
                    handleUnbindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SERVICE_ARGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceStart: " + String.valueOf(msg.obj)));
                    handleServiceArgs((ServiceArgsData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case STOP_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceStop");
                    handleStopService((IBinder)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CONFIGURATION_CHANGED:
                    handleConfigurationChanged((Configuration) msg.obj);
                    break;
                case CLEAN_UP_CONTEXT:
                    ContextCleanupInfo cci = (ContextCleanupInfo)msg.obj;
                    cci.context.performFinalCleanup(cci.who, cci.what);
                    break;
                case GC_WHEN_IDLE:
                    scheduleGcIdler();
                    break;
                case DUMP_SERVICE:
                    handleDumpService((DumpComponentInfo)msg.obj);
                    break;
                case LOW_MEMORY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "lowMemory");
                    handleLowMemory();
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case PROFILER_CONTROL:
                    handleProfilerControl(msg.arg1 != 0, (ProfilerInfo)msg.obj, msg.arg2);
                    break;
                case CREATE_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupCreateAgent");
                    handleCreateBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case DESTROY_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupDestroyAgent");
                    handleDestroyBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SUICIDE:
                    Process.killProcess(Process.myPid());
                    break;
                case REMOVE_PROVIDER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "providerRemove");
                    completeRemoveProvider((ProviderRefCount)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ENABLE_JIT:
                    ensureJitEnabled();
                    break;
                case DISPATCH_PACKAGE_BROADCAST:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastPackage");
                    handleDispatchPackageBroadcast(msg.arg1, (String[])msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SCHEDULE_CRASH:
                    throw new RemoteServiceException((String)msg.obj);
                case DUMP_HEAP:
                    handleDumpHeap((DumpHeapData) msg.obj);
                    break;
                case DUMP_ACTIVITY:
                    handleDumpActivity((DumpComponentInfo)msg.obj);
                    break;
                case DUMP_PROVIDER:
                    handleDumpProvider((DumpComponentInfo)msg.obj);
                    break;
                case SLEEPING:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "sleeping");
                    handleSleeping((IBinder)msg.obj, msg.arg1 != 0);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SET_CORE_SETTINGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "setCoreSettings");
                    handleSetCoreSettings((Bundle) msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UPDATE_PACKAGE_COMPATIBILITY_INFO:
                    handleUpdatePackageCompatibilityInfo((UpdateCompatibilityData)msg.obj);
                    break;
                case UNSTABLE_PROVIDER_DIED:
                    handleUnstableProviderDied((IBinder)msg.obj, false);
                    break;
                case REQUEST_ASSIST_CONTEXT_EXTRAS:
                    handleRequestAssistContextExtras((RequestAssistContextExtras)msg.obj);
                    break;
                case TRANSLUCENT_CONVERSION_COMPLETE:
                    handleTranslucentConversionComplete((IBinder)msg.obj, msg.arg1 == 1);
                    break;
                case INSTALL_PROVIDER:
                    handleInstallProvider((ProviderInfo) msg.obj);
                    break;
                case ON_NEW_ACTIVITY_OPTIONS:
                    Pair<IBinder, ActivityOptions> pair = (Pair<IBinder, ActivityOptions>) msg.obj;
                    onNewActivityOptions(pair.first, pair.second);
                    break;
                case ENTER_ANIMATION_COMPLETE:
                    handleEnterAnimationComplete((IBinder) msg.obj);
                    break;
                case START_BINDER_TRACKING:
                    handleStartBinderTracking();
                    break;
                case STOP_BINDER_TRACKING_AND_DUMP:
                    handleStopBinderTrackingAndDump((ParcelFileDescriptor) msg.obj);
                    break;
                case LOCAL_VOICE_INTERACTION_STARTED:
                    handleLocalVoiceInteractionStarted((IBinder) ((SomeArgs) msg.obj).arg1,
                            (IVoiceInteractor) ((SomeArgs) msg.obj).arg2);
                    break;
                case ATTACH_AGENT: {
                    Application app = getApplication();
                    handleAttachAgent((String) msg.obj, app != null ? app.mLoadedApk : null);
                    break;
                }
                case APPLICATION_INFO_CHANGED:
                    mUpdatingSystemConfig = true;
                    try {
                        handleApplicationInfoChanged((ApplicationInfo) msg.obj);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    break;
                case RUN_ISOLATED_ENTRY_POINT:
                    handleRunIsolatedEntryPoint((String) ((SomeArgs) msg.obj).arg1,
                            (String[]) ((SomeArgs) msg.obj).arg2);
                    break;
                case EXECUTE_TRANSACTION:
                    final ClientTransaction transaction = (ClientTransaction) msg.obj;
                    mTransactionExecutor.execute(transaction);
                    if (isSystem()) {
                        // Client transactions inside system process are recycled on the client side
                        // instead of ClientLifecycleManager to avoid being cleared before this
                        // message is handled.
                        transaction.recycle();
                    }
                    // TODO(lifecycler): Recycle locally scheduled transactions.
                    break;
                case RELAUNCH_ACTIVITY:
                    handleRelaunchActivityLocally((IBinder) msg.obj);
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }
    }

```

위의 모든 case에서 ActivityManager와 상호작용을 하는 것은 아닌 것 같지만,

Service 핸들링 관련 함수가 많이 보이는데, Service 관련해서는 ActivityManager와 상호작용을 다 하고 있다.

반면, Activity에 관한 case들은 잘 보이지 않는데 handleSleeping 부분이 있다.

이 함수에서는 Activity에 대해 다루고 있다.

```
    // TODO: This method should be changed to use {@link #performStopActivityInner} to perform to
    // stop operation on the activity to reduce code duplication and the chance of fixing a bug in
    // one place and missing the other.
    private void handleSleeping(IBinder token, boolean sleeping) {
        ActivityClientRecord r = mActivities.get(token);
        if (r == null) {
            Log.w(TAG, "handleSleeping: no activity for token " + token);
            return;
        }
        if (sleeping) {
            if (!r.stopped && !r.isPreHoneycomb()) {
                callActivityOnStop(r, true /* saveState */, "sleeping");
            }
            // Make sure any pending writes are now committed.
            if (!r.isPreHoneycomb()) {
                QueuedWork.waitToFinish();
            }
            // Tell activity manager we slept.
            try {
                ActivityManager.getService().activitySlept(r.token);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        } else {
            if (r.stopped && r.activity.mVisibleFromServer) {
                r.activity.performRestart(true /* start */, "handleSleeping");
                r.setState(ON_START);
            }
        }
    }
```

위 함수를 보면 첫 번째 조건문에서는 인자로 받은 token에 해당되는 Activity가 있는지

확인을 하고 있고, 두 번째 조건문에서 sleeping 이 true인지 확인한다.

내부에서 sleeping 상태에 들어간다고 ActivityManager에게 알려주는 부분이 있다.

이 외에도 생명주기를 핸들링하는 handle로 시작하는 함수들이 많고,

그 내부에 ActivityManager와 상호작용하는 코드들이 있는데 위의 handleMessage 함수 내부에

case에는 존재하지 않는 함수들이 많다. 어디서 호출되고 언제 사용되는 지

알아보면, ActivityThread에서 Activity부터 ActivityManager까지 연결되는 연결고리에

대한 해답이 나올 것 같다.
