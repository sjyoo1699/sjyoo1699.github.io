---
layout: post
title: "180729 WHERE TO RUN THE ACTIVITY 04"
date:   2018-07-29 14:00:30 +0900
categories: jekyll update
---

WHERE TO RUN THE ACTIVITY 03 포스팅에 이어서, 이번 포스팅에서는 AcltivityManagerService에서

**startActivity 함수를 호출하는 과정**과, **thread 부분**에 대해서 다룰 것이다.

먼저 ActivityResult 클래스에서 execStartActivity 함수를 보면 

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

밑에 try 문에서 ActivityService의 startActivity 함수를 호출한다.

위의 부분은 이미 만들어진 Activity에 대한 실행 내용이고, 밑에 try 부분이 새로 만들어지는

Activity에 대한 부분일 것이라고 추측했는데, 이에 대해 알아보기 위해 

startActivity 함수를 찾아보았다.

***
startActivity 라는 이름의 함수가 여러 개 있었는데, 위의 함수에서 사용된 startActivity 함수는

아래의 함수이다.

```
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }

    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        enforceNotIsolatedCaller("startActivity");
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(),
                userId, false, ALLOW_FULL_ONLY, "startActivity", null);
        // TODO: Switch to user app stacks here.
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, "startActivityAsUser");
    }
```

인자로 받은 정보들로 startActivityAsUser 함수를 콜하고 그 값을 리턴한다.

startActivityAsUser 함수를 보면 인자로 받은 값들을 이용하여, ActivityStarter의 startActivityMayWait라는

이름의 함수를 call 하고 그 값을 리턴하는 것을 볼 수 있다.

따라서 ActivityStarter의 startActivityMayWait 함수를 찾아보았다.

***
```
    final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, WaitResult outResult,
            Configuration globalConfig, Bundle bOptions, boolean ignoreTargetSecurity, int userId,
            TaskRecord inTask, String reason) {
        // Refuse possible leaked file descriptors
        if (intent != null && intent.hasFileDescriptors()) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }
        mSupervisor.mActivityMetricsLogger.notifyActivityLaunching();
        boolean componentSpecified = intent.getComponent() != null;

        // Save a copy in case ephemeral needs it
        final Intent ephemeralIntent = new Intent(intent);
        // Don't modify the client's object!
        intent = new Intent(intent);
        if (componentSpecified
                && intent.getData() != null
                && Intent.ACTION_VIEW.equals(intent.getAction())
                && mService.getPackageManagerInternalLocked()
                        .isInstantAppInstallerComponent(intent.getComponent())) {
            // intercept intents targeted directly to the ephemeral installer the
            // ephemeral installer should never be started with a raw URL; instead
            // adjust the intent so it looks like a "normal" instant app launch
            intent.setComponent(null /*component*/);
            componentSpecified = false;
        }

        ResolveInfo rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId);
        if (rInfo == null) {
            UserInfo userInfo = mSupervisor.getUserInfo(userId);
            if (userInfo != null && userInfo.isManagedProfile()) {
                // Special case for managed profiles, if attempting to launch non-cryto aware
                // app in a locked managed profile from an unlocked parent allow it to resolve
                // as user will be sent via confirm credentials to unlock the profile.
                UserManager userManager = UserManager.get(mService.mContext);
                boolean profileLockedAndParentUnlockingOrUnlocked = false;
                long token = Binder.clearCallingIdentity();
                try {
                    UserInfo parent = userManager.getProfileParent(userId);
                    profileLockedAndParentUnlockingOrUnlocked = (parent != null)
                            && userManager.isUserUnlockingOrUnlocked(parent.id)
                            && !userManager.isUserUnlockingOrUnlocked(userId);
                } finally {
                    Binder.restoreCallingIdentity(token);
                }
                if (profileLockedAndParentUnlockingOrUnlocked) {
                    rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId,
                            PackageManager.MATCH_DIRECT_BOOT_AWARE
                                    | PackageManager.MATCH_DIRECT_BOOT_UNAWARE);
                }
            }
        }
        // Collect information about the target of the Intent.
        ActivityInfo aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, profilerInfo);

        ActivityOptions options = ActivityOptions.fromBundle(bOptions);
        synchronized (mService) {
            final int realCallingPid = Binder.getCallingPid();
            final int realCallingUid = Binder.getCallingUid();
            int callingPid;
            if (callingUid >= 0) {
                callingPid = -1;
            } else if (caller == null) {
                callingPid = realCallingPid;
                callingUid = realCallingUid;
            } else {
                callingPid = callingUid = -1;
            }

            final ActivityStack stack = mSupervisor.mFocusedStack;
            stack.mConfigWillChange = globalConfig != null
                    && mService.getGlobalConfiguration().diff(globalConfig) != 0;
            if (DEBUG_CONFIGURATION) Slog.v(TAG_CONFIGURATION,
                    "Starting activity when config will change = " + stack.mConfigWillChange);

            final long origId = Binder.clearCallingIdentity();

            if (aInfo != null &&
                    (aInfo.applicationInfo.privateFlags
                            & ApplicationInfo.PRIVATE_FLAG_CANT_SAVE_STATE) != 0) {
                // This may be a heavy-weight process!  Check to see if we already
                // have another, different heavy-weight process running.
                if (aInfo.processName.equals(aInfo.applicationInfo.packageName)) {
                    final ProcessRecord heavy = mService.mHeavyWeightProcess;
                    if (heavy != null && (heavy.info.uid != aInfo.applicationInfo.uid
                            || !heavy.processName.equals(aInfo.processName))) {
                        int appCallingUid = callingUid;
                        if (caller != null) {
                            ProcessRecord callerApp = mService.getRecordForAppLocked(caller);
                            if (callerApp != null) {
                                appCallingUid = callerApp.info.uid;
                            } else {
                                Slog.w(TAG, "Unable to find app for caller " + caller
                                        + " (pid=" + callingPid + ") when starting: "
                                        + intent.toString());
                                ActivityOptions.abort(options);
                                return ActivityManager.START_PERMISSION_DENIED;
                            }
                        }

                        IIntentSender target = mService.getIntentSenderLocked(
                                ActivityManager.INTENT_SENDER_ACTIVITY, "android",
                                appCallingUid, userId, null, null, 0, new Intent[] { intent },
                                new String[] { resolvedType }, PendingIntent.FLAG_CANCEL_CURRENT
                                        | PendingIntent.FLAG_ONE_SHOT, null);

                        Intent newIntent = new Intent();
                        if (requestCode >= 0) {
                            // Caller is requesting a result.
                            newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_HAS_RESULT, true);
                        }
                        newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_INTENT,
                                new IntentSender(target));
                        if (heavy.activities.size() > 0) {
                            ActivityRecord hist = heavy.activities.get(0);
                            newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_CUR_APP,
                                    hist.packageName);
                            newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_CUR_TASK,
                                    hist.getTask().taskId);
                        }
                        newIntent.putExtra(HeavyWeightSwitcherActivity.KEY_NEW_APP,
                                aInfo.packageName);
                        newIntent.setFlags(intent.getFlags());
                        newIntent.setClassName("android",
                                HeavyWeightSwitcherActivity.class.getName());
                        intent = newIntent;
                        resolvedType = null;
                        caller = null;
                        callingUid = Binder.getCallingUid();
                        callingPid = Binder.getCallingPid();
                        componentSpecified = true;
                        rInfo = mSupervisor.resolveIntent(intent, null /*resolvedType*/, userId);
                        aInfo = rInfo != null ? rInfo.activityInfo : null;
                        if (aInfo != null) {
                            aInfo = mService.getActivityInfoForUser(aInfo, userId);
                        }
                    }
                }
            }

            final ActivityRecord[] outRecord = new ActivityRecord[1];
            int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                    aInfo, rInfo, voiceSession, voiceInteractor,
                    resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                    options, ignoreTargetSecurity, componentSpecified, outRecord, inTask,
                    reason);

            Binder.restoreCallingIdentity(origId);

            if (stack.mConfigWillChange) {
                // If the caller also wants to switch to a new configuration,
                // do so now.  This allows a clean switch, as we are waiting
                // for the current activity to pause (so we will not destroy
                // it), and have not yet started the next activity.
                mService.enforceCallingPermission(android.Manifest.permission.CHANGE_CONFIGURATION,
                        "updateConfiguration()");
                stack.mConfigWillChange = false;
                if (DEBUG_CONFIGURATION) Slog.v(TAG_CONFIGURATION,
                        "Updating to new configuration after starting activity.");
                mService.updateConfigurationLocked(globalConfig, null, false);
            }

            if (outResult != null) {
                outResult.result = res;
                if (res == ActivityManager.START_SUCCESS) {
                    mSupervisor.mWaitingActivityLaunched.add(outResult);
                    do {
                        try {
                            mService.wait();
                        } catch (InterruptedException e) {
                        }
                    } while (outResult.result != START_TASK_TO_FRONT
                            && !outResult.timeout && outResult.who == null);
                    if (outResult.result == START_TASK_TO_FRONT) {
                        res = START_TASK_TO_FRONT;
                    }
                }
                if (res == START_TASK_TO_FRONT) {
                    final ActivityRecord r = outRecord[0];

                    // ActivityRecord may represent a different activity, but it should not be in
                    // the resumed state.
                    if (r.nowVisible && r.state == RESUMED) {
                        outResult.timeout = false;
                        outResult.who = r.realActivity;
                        outResult.totalTime = 0;
                        outResult.thisTime = 0;
                    } else {
                        outResult.thisTime = SystemClock.uptimeMillis();
                        mSupervisor.waitActivityVisible(r.realActivity, outResult);
                        // Note: the timeout variable is not currently not ever set.
                        do {
                            try {
                                mService.wait();
                            } catch (InterruptedException e) {
                            }
                        } while (!outResult.timeout && outResult.who == null);
                    }
                }
            }

            mSupervisor.mActivityMetricsLogger.notifyActivityLaunched(res, outRecord[0]);
            return res;
        }
    }

```

위 코드에서 동기화하는 부분이 실제 액티비티를 만들어서 실행시키는 부분인 것 같다.

mService는 ActivityManagerService이다.

위의 코드를 보면 실행을 준비하는 부분과, 실제 실행을 하는 부분이 있는데,

mSupervisor.mWaitingActivityLaunched.add(outResult);

이 부분이 StackSupervisor에게 결과 ActvitiyRecord를 넘겨주고, StackSupervisor가 해당 ActivityRecord를

추가할 때까지 mService는 기다리는 코드가 아마 실제 실행을 하는 부분인 것 같다.

그래서 위의 mWaitingActivityLaunched를 찾아보았다.

***
```
    /** List of processes waiting to find out about the next launched activity. */
    final ArrayList<WaitResult> mWaitingActivityLaunched = new ArrayList<>();
```

위의 함수를 보면 outResult.result가 START_TASK_TO_FRONT 가 되거나, timeout이 되거나, outResultWho가 null이

아니게 될 때까지 기다린다.

따라서 StackSupervisor에서 mWaitingActivityLaunched 가 사용되는 곳을 찾아보았다.

***
```
    void reportTaskToFrontNoLaunch(ActivityRecord r) {
        boolean changed = false;
        for (int i = mWaitingActivityLaunched.size() - 1; i >= 0; i--) {
            WaitResult w = mWaitingActivityLaunched.remove(i);
            if (w.who == null) {
                changed = true;
                // Set result to START_TASK_TO_FRONT so that startActivityMayWait() knows that
                // the starting activity ends up moving another activity to front, and it should
                // wait for this new activity to become visible instead.
                // Do not modify other fields.
                w.result = START_TASK_TO_FRONT;
            }
        }
        if (changed) {
            mService.notifyAll();
        }
    }

    void reportActivityLaunchedLocked(boolean timeout, ActivityRecord r,
            long thisTime, long totalTime) {
        boolean changed = false;
        for (int i = mWaitingActivityLaunched.size() - 1; i >= 0; i--) {
            WaitResult w = mWaitingActivityLaunched.remove(i);
            if (w.who == null) {
                changed = true;
                w.timeout = timeout;
                if (r != null) {
                    w.who = new ComponentName(r.info.packageName, r.info.name);
                }
                w.thisTime = thisTime;
                w.totalTime = totalTime;
                // Do not modify w.result.
            }
        }
        if (changed) {
            mService.notifyAll();
        }
    }
```

위의 두 함수가 mService를 깨우는 함수인데, 위의 함수는 정상적으로 실행이 된 경우고,

아래의 함수는 실행이 Locked된 상황인 것 같다.

따라서, 위의 reportTaskToFrontNoLaunch 함수가 어디서 call 되는 지 찾아보았다.

call이 되는 곳은 ActivityStarter 클래스였다.

```
    void postStartActivityProcessing(
            ActivityRecord r, int result, int prevFocusedStackId, ActivityRecord sourceRecord,
            ActivityStack targetStack) {
        if (ActivityManager.isStartResultFatalError(result)) {
            return;
        }
        // We're waiting for an activity launch to finish, but that activity simply
        // brought another activity to front. Let startActivityMayWait() know about
        // this, so it waits for the new activity to become visible instead.
        if (result == START_TASK_TO_FRONT && !mSupervisor.mWaitingActivityLaunched.isEmpty()) {
            mSupervisor.reportTaskToFrontNoLaunch(mStartActivity);
        }
        int startedActivityStackId = INVALID_STACK_ID;
        final ActivityStack currentStack = r.getStack();
        if (currentStack != null) {
            startedActivityStackId = currentStack.mStackId;
        } else if (mTargetStack != null) {
            startedActivityStackId = targetStack.mStackId;
        }
        if (startedActivityStackId == DOCKED_STACK_ID) {
            final ActivityStack homeStack = mSupervisor.getStack(HOME_STACK_ID);
            final boolean homeStackVisible = homeStack != null && homeStack.isVisible();
            if (homeStackVisible) {
                // We launch an activity while being in home stack, which means either launcher or
                // recents into docked stack. We don't want the launched activity to be alone in a
                // docked stack, so we want to immediately launch recents too.
                if (DEBUG_RECENTS) Slog.d(TAG, "Scheduling recents launch.");
                mWindowManager.showRecentApps(true /* fromHome */);
            }
            return;
        }
        boolean clearedTask = (mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
                == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK) && (mReuseTask != null);
        if (startedActivityStackId == PINNED_STACK_ID && (result == START_TASK_TO_FRONT
                || result == START_DELIVERED_TO_TOP || clearedTask)) {
            // The activity was already running in the pinned stack so it wasn't started, but either
            // brought to the front or the new intent was delivered to it since it was already in
            // front. Notify anyone interested in this piece of information.
            mService.mTaskChangeNotificationController.notifyPinnedActivityRestartAttempt(
                    clearedTask);
            return;
        }
    }
```

위 함수가 call 되는 곳을 다시 찾아보니, ActivityStackSupervisor의 startActivityFromRecentsInner 함수였다.

```
    final int startActivityFromRecentsInner(int taskId, Bundle bOptions) {
        final TaskRecord task;
        final int callingUid;
        final String callingPackage;
        final Intent intent;
        final int userId;
        final ActivityOptions activityOptions = (bOptions != null)
                ? new ActivityOptions(bOptions) : null;
        final int launchStackId = (activityOptions != null)
                ? activityOptions.getLaunchStackId() : INVALID_STACK_ID;
        if (StackId.isHomeOrRecentsStack(launchStackId)) {
            throw new IllegalArgumentException("startActivityFromRecentsInner: Task "
                    + taskId + " can't be launch in the home/recents stack.");
        }
        mWindowManager.deferSurfaceLayout();
        try {
            if (launchStackId == DOCKED_STACK_ID) {
                mWindowManager.setDockedStackCreateState(
                        activityOptions.getDockCreateMode(), null /* initialBounds */);
                // Defer updating the stack in which recents is until the app transition is done, to
                // not run into issues where we still need to draw the task in recents but the
                // docked stack is already created.
                deferUpdateBounds(RECENTS_STACK_ID);
                mWindowManager.prepareAppTransition(TRANSIT_DOCK_TASK_FROM_RECENTS, false);
            }
            task = anyTaskForIdLocked(taskId, MATCH_TASK_IN_STACKS_OR_RECENT_TASKS_AND_RESTORE,
                    launchStackId);
            if (task == null) {
                continueUpdateBounds(RECENTS_STACK_ID);
                mWindowManager.executeAppTransition();
                throw new IllegalArgumentException(
                        "startActivityFromRecentsInner: Task " + taskId + " not found.");
            }
            // Since we don't have an actual source record here, we assume that the currently
            // focused activity was the source.
            final ActivityStack focusedStack = getFocusedStack();
            final ActivityRecord sourceRecord =
                    focusedStack != null ? focusedStack.topActivity() : null;
            if (launchStackId != INVALID_STACK_ID) {
                if (task.getStackId() != launchStackId) {
                    task.reparent(launchStackId, ON_TOP, REPARENT_MOVE_STACK_TO_FRONT, ANIMATE,
                            DEFER_RESUME, "startActivityFromRecents");
                }
            }
            // If the user must confirm credentials (e.g. when first launching a work app and the
            // Work Challenge is present) let startActivityInPackage handle the intercepting.
            if (!mService.mUserController.shouldConfirmCredentials(task.userId)
                    && task.getRootActivity() != null) {
                final ActivityRecord targetActivity = task.getTopActivity();
                mService.mActivityStarter.sendPowerHintForLaunchStartIfNeeded(true /* forceSend */,
                        targetActivity);
                mActivityMetricsLogger.notifyActivityLaunching();
                mService.moveTaskToFrontLocked(task.taskId, 0, bOptions, true /* fromRecents */);
                mActivityMetricsLogger.notifyActivityLaunched(ActivityManager.START_TASK_TO_FRONT,
                        targetActivity);
                // If we are launching the task in the docked stack, put it into resizing mode so
                // the window renders full-screen with the background filling the void. Also only
                // call this at the end to make sure that tasks exists on the window manager side.
                if (launchStackId == DOCKED_STACK_ID) {
                    setResizingDuringAnimation(task);
                }
                mService.mActivityStarter.postStartActivityProcessing(task.getTopActivity(),
                        ActivityManager.START_TASK_TO_FRONT,
                        sourceRecord != null
                                ? sourceRecord.getTask().getStackId() : INVALID_STACK_ID,
                        sourceRecord, task.getStack());
                return ActivityManager.START_TASK_TO_FRONT;
            }
            callingUid = task.mCallingUid;
            callingPackage = task.mCallingPackage;
            intent = task.intent;
            intent.addFlags(Intent.FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY);
            userId = task.userId;
            int result = mService.startActivityInPackage(callingUid, callingPackage, intent, null,
                    null, null, 0, 0, bOptions, userId, task, "startActivityFromRecents");
            if (launchStackId == DOCKED_STACK_ID) {
                setResizingDuringAnimation(task);
            }
            return result;
        } finally {
            mWindowManager.continueSurfaceLayout();
        }
    }
```
위 함수에서 실제로 액티비티가 화면에 보여지는 듯 하다.

위 함수가 call 되는 곳은 아직 찾지 못하였고, 

ActivityManager에서 위의 startActivityMayWait 함수가 call 되는 곳을 찾았다.

```
    @Override
    public final int startActivityFromRecents(int taskId, Bundle options) {
        if (checkCallingPermission(START_TASKS_FROM_RECENTS) != PackageManager.PERMISSION_GRANTED) {
            String msg = "Permission Denial: startActivityFromRecents called without " +
                    START_TASKS_FROM_RECENTS;
            Slog.w(TAG, msg);
            throw new SecurityException(msg);
        }
        return startActivityFromRecentsInner(taskId, options);
    }
    final int startActivityFromRecentsInner(int taskId, Bundle options) {
        final TaskRecord task;
        final int callingUid;
        final String callingPackage;
        final Intent intent;
        final int userId;
        synchronized (this) {
            task = recentTaskForIdLocked(taskId);
            if (task == null) {
                throw new IllegalArgumentException("Task " + taskId + " not found.");
            }
            callingUid = task.mCallingUid;
            callingPackage = task.mCallingPackage;
            intent = task.intent;
            intent.addFlags(Intent.FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY);
            userId = task.userId;
        }
        return startActivityInPackage(callingUid, callingPackage, intent, null, null, null, 0, 0,
                options, userId, null, task);
    }
    final int startActivityInPackage(int uid, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo,
            String resultWho, int requestCode, int startFlags, Bundle options, int userId,
            IActivityContainer container, TaskRecord inTask) {
        userId = handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId,
                false, ALLOW_FULL_ONLY, "startActivityInPackage", null);
        // TODO: Switch to user app stacks here.
        int ret = mStackSupervisor.startActivityMayWait(null, uid, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                null, null, null, options, userId, container, inTask);
        return ret;
    }
```

***

execStartActivity 함수에서 checkStartActivityResult 함수를 사용하는데,

단순히 activity가 잘 실행이 되었나를 판단하는 함수인 듯 하다.

```
    /** @hide */
    public static void checkStartActivityResult(int res, Object intent) {
        if (!ActivityManager.isStartResultFatalError(res)) {
            return;
        }
        switch (res) {
            case ActivityManager.START_INTENT_NOT_RESOLVED:
            case ActivityManager.START_CLASS_NOT_FOUND:
                if (intent instanceof Intent && ((Intent)intent).getComponent() != null)
                    throw new ActivityNotFoundException(
                            "Unable to find explicit activity class "
                            + ((Intent)intent).getComponent().toShortString()
                            + "; have you declared this activity in your AndroidManifest.xml?");
                throw new ActivityNotFoundException(
                        "No Activity found to handle " + intent);
            case ActivityManager.START_PERMISSION_DENIED:
                throw new SecurityException("Not allowed to start activity "
                        + intent);
            case ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT:
                throw new AndroidRuntimeException(
                        "FORWARD_RESULT_FLAG used while also requesting a result");
            case ActivityManager.START_NOT_ACTIVITY:
                throw new IllegalArgumentException(
                        "PendingIntent is not an activity");
            case ActivityManager.START_NOT_VOICE_COMPATIBLE:
                throw new SecurityException(
                        "Starting under voice control not allowed for: " + intent);
            case ActivityManager.START_VOICE_NOT_ACTIVE_SESSION:
                throw new IllegalStateException(
                        "Session calling startVoiceActivity does not match active session");
            case ActivityManager.START_VOICE_HIDDEN_SESSION:
                throw new IllegalStateException(
                        "Cannot start voice activity on a hidden session");
            case ActivityManager.START_ASSISTANT_NOT_ACTIVE_SESSION:
                throw new IllegalStateException(
                        "Session calling startAssistantActivity does not match active session");
            case ActivityManager.START_ASSISTANT_HIDDEN_SESSION:
                throw new IllegalStateException(
                        "Cannot start assistant activity on a hidden session");
            case ActivityManager.START_CANCELED:
                throw new AndroidRuntimeException("Activity could not be started for "
                        + intent);
            default:
                throw new AndroidRuntimeException("Unknown error code "
                        + res + " when starting " + intent);
        }
    }
```

startActivityMayWait 함수 내에서 이미 액티비티가 만들어지고 실행이 되고,

checkStartActivityResult 함수는 그 결과를 체크하는 용도의 함수인 듯 하다.

***

계속 함수가 call 되는 부분을 추적하여 따라갔지만, 아직 그림이 그려지지 않는다.
