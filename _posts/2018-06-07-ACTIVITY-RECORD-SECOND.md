---
layout: post
title: "180607 ACTIVITY RECORD SECOND"
date:   2018-06-07 19:00:30 +0900
categories: jekyll update
---

Activity Record.java의 함수 중에 dump 라는 함수가 있다. 이 함수의 인자로는 PrintWriter 객체와, String으로 prefix를 받는다.
dump 함수는 Acitivity의 정보를 써서 어딘가에 저장해놓는 기능을 하는 함수인데, 이 함수의 호출이 잦다면 PrintWriter가 아닌 
BufferedWriter를 쓰는 게 시간적으로 효율이 좋을 것이라고 생각이 되기에, 이에 대해서 더 알아봐야 할 것 같고, 이 함수가 호출되는
부분을 찾아서 봐야 판단이 될 것 같다. 잦은 호출이 아니더라도, BufferedWriter를 쓰는 게 좋을 것 같은데, 굳이 PrintWriter를 쓴
이유에 대해서 찾아봐야겠다.

```
void dump(PrintWriter pw, String prefix) {
    final long now = SystemClock.uptimeMillis();
    pw.print(prefix); pw.print("packageName="); pw.print(packageName);
            pw.print(" processName="); pw.println(processName);
    pw.print(prefix); pw.print("launchedFromUid="); pw.print(launchedFromUid);
            pw.print(" launchedFromPackage="); pw.print(launchedFromPackage);
            pw.print(" userId="); pw.println(userId);
    pw.print(prefix); pw.print("app="); pw.println(app);
    pw.print(prefix); pw.println(intent.toInsecureStringWithClip());
    pw.print(prefix); pw.print("frontOfTask="); pw.print(frontOfTask);
            pw.print(" task="); pw.println(task);
    pw.print(prefix); pw.print("taskAffinity="); pw.println(taskAffinity);
    pw.print(prefix); pw.print("realActivity=");
            pw.println(realActivity.flattenToShortString());
    if (appInfo != null) {
        pw.print(prefix); pw.print("baseDir="); pw.println(appInfo.sourceDir);
        if (!Objects.equals(appInfo.sourceDir, appInfo.publicSourceDir)) {
            pw.print(prefix); pw.print("resDir="); pw.println(appInfo.publicSourceDir);
        }
        pw.print(prefix); pw.print("dataDir="); pw.println(appInfo.dataDir);
        if (appInfo.splitSourceDirs != null) {
            pw.print(prefix); pw.print("splitDir=");
                    pw.println(Arrays.toString(appInfo.splitSourceDirs));
        }
    }
    pw.print(prefix); pw.print("stateNotNeeded="); pw.print(stateNotNeeded);
            pw.print(" componentSpecified="); pw.print(componentSpecified);
            pw.print(" mActivityType="); pw.println(mActivityType);
    if (rootVoiceInteraction) {
        pw.print(prefix); pw.print("rootVoiceInteraction="); pw.println(rootVoiceInteraction);
    }
    pw.print(prefix); pw.print("compat="); pw.print(compat);
            pw.print(" labelRes=0x"); pw.print(Integer.toHexString(labelRes));
            pw.print(" icon=0x"); pw.print(Integer.toHexString(icon));
            pw.print(" theme=0x"); pw.println(Integer.toHexString(theme));
    pw.println(prefix + "mLastReportedConfigurations:");
    mLastReportedConfiguration.dump(pw, prefix + " ");

    pw.print(prefix); pw.print("CurrentConfiguration="); pw.println(getConfiguration());
    if (!getOverrideConfiguration().equals(EMPTY)) {
        pw.println(prefix + "OverrideConfiguration=" + getOverrideConfiguration());
    }
    if (!mBounds.isEmpty()) {
        pw.println(prefix + "mBounds=" + mBounds);
    }
    if (resultTo != null || resultWho != null) {
        pw.print(prefix); pw.print("resultTo="); pw.print(resultTo);
                pw.print(" resultWho="); pw.print(resultWho);
                pw.print(" resultCode="); pw.println(requestCode);
    }
    if (taskDescription != null) {
        final String iconFilename = taskDescription.getIconFilename();
        if (iconFilename != null || taskDescription.getLabel() != null ||
                taskDescription.getPrimaryColor() != 0) {
            pw.print(prefix); pw.print("taskDescription:");
                    pw.print(" iconFilename="); pw.print(taskDescription.getIconFilename());
                    pw.print(" label=\""); pw.print(taskDescription.getLabel());
                            pw.print("\"");
                    pw.print(" primaryColor=");
                    pw.println(Integer.toHexString(taskDescription.getPrimaryColor()));
                    pw.print(prefix); pw.print("  backgroundColor=");
                    pw.println(Integer.toHexString(taskDescription.getBackgroundColor()));
                    pw.print(prefix); pw.print("  statusBarColor=");
                    pw.println(Integer.toHexString(taskDescription.getStatusBarColor()));
                    pw.print(prefix); pw.print("  navigationBarColor=");
                    pw.println(Integer.toHexString(taskDescription.getNavigationBarColor()));
        }
        if (iconFilename == null && taskDescription.getIcon() != null) {
            pw.print(prefix); pw.println("taskDescription contains Bitmap");
        }
    }
    if (results != null) {
        pw.print(prefix); pw.print("results="); pw.println(results);
    }
    if (pendingResults != null && pendingResults.size() > 0) {
        pw.print(prefix); pw.println("Pending Results:");
        for (WeakReference<PendingIntentRecord> wpir : pendingResults) {
            PendingIntentRecord pir = wpir != null ? wpir.get() : null;
            pw.print(prefix); pw.print("  - ");
            if (pir == null) {
                pw.println("null");
            } else {
                pw.println(pir);
                pir.dump(pw, prefix + "    ");
            }
        }
    }
    if (newIntents != null && newIntents.size() > 0) {
        pw.print(prefix); pw.println("Pending New Intents:");
        for (int i=0; i<newIntents.size(); i++) {
            Intent intent = newIntents.get(i);
            pw.print(prefix); pw.print("  - ");
            if (intent == null) {
                pw.println("null");
            } else {
                pw.println(intent.toShortString(false, true, false, true));
            }
        }
    }
    if (pendingOptions != null) {
        pw.print(prefix); pw.print("pendingOptions="); pw.println(pendingOptions);
    }
    if (appTimeTracker != null) {
        appTimeTracker.dumpWithHeader(pw, prefix, false);
    }
    if (uriPermissions != null) {
        uriPermissions.dump(pw, prefix);
    }
    pw.print(prefix); pw.print("launchFailed="); pw.print(launchFailed);
            pw.print(" launchCount="); pw.print(launchCount);
            pw.print(" lastLaunchTime=");
            if (lastLaunchTime == 0) pw.print("0");
            else TimeUtils.formatDuration(lastLaunchTime, now, pw);
            pw.println();
    pw.print(prefix); pw.print("haveState="); pw.print(haveState);
            pw.print(" icicle="); pw.println(icicle);
    pw.print(prefix); pw.print("state="); pw.print(state);
            pw.print(" stopped="); pw.print(stopped);
            pw.print(" delayedResume="); pw.print(delayedResume);
            pw.print(" finishing="); pw.println(finishing);
    pw.print(prefix); pw.print("keysPaused="); pw.print(keysPaused);
            pw.print(" inHistory="); pw.print(inHistory);
            pw.print(" visible="); pw.print(visible);
            pw.print(" sleeping="); pw.print(sleeping);
            pw.print(" idle="); pw.print(idle);
            pw.print(" mStartingWindowState=");
            pw.println(startingWindowStateToString(mStartingWindowState));
    pw.print(prefix); pw.print("fullscreen="); pw.print(fullscreen);
            pw.print(" noDisplay="); pw.print(noDisplay);
            pw.print(" immersive="); pw.print(immersive);
            pw.print(" launchMode="); pw.println(launchMode);
    pw.print(prefix); pw.print("frozenBeforeDestroy="); pw.print(frozenBeforeDestroy);
            pw.print(" forceNewConfig="); pw.println(forceNewConfig);
    pw.print(prefix); pw.print("mActivityType=");
            pw.println(activityTypeToString(mActivityType));
    if (requestedVrComponent != null) {
        pw.print(prefix);
        pw.print("requestedVrComponent=");
        pw.println(requestedVrComponent);
    }
    if (displayStartTime != 0 || startTime != 0) {
        pw.print(prefix); pw.print("displayStartTime=");
                if (displayStartTime == 0) pw.print("0");
                else TimeUtils.formatDuration(displayStartTime, now, pw);
                pw.print(" startTime=");
                if (startTime == 0) pw.print("0");
                else TimeUtils.formatDuration(startTime, now, pw);
                pw.println();
    }
    final boolean waitingVisible =
            mStackSupervisor.mActivitiesWaitingForVisibleActivity.contains(this);
    if (lastVisibleTime != 0 || waitingVisible || nowVisible) {
        pw.print(prefix); pw.print("waitingVisible="); pw.print(waitingVisible);
                pw.print(" nowVisible="); pw.print(nowVisible);
                pw.print(" lastVisibleTime=");
                if (lastVisibleTime == 0) pw.print("0");
                else TimeUtils.formatDuration(lastVisibleTime, now, pw);
                pw.println();
    }
    if (mDeferHidingClient) {
        pw.println(prefix + "mDeferHidingClient=" + mDeferHidingClient);
    }
    if (deferRelaunchUntilPaused || configChangeFlags != 0) {
        pw.print(prefix); pw.print("deferRelaunchUntilPaused="); pw.print(deferRelaunchUntilPaused);
                pw.print(" configChangeFlags=");
                pw.println(Integer.toHexString(configChangeFlags));
    }
    if (connections != null) {
        pw.print(prefix); pw.print("connections="); pw.println(connections);
    }
    if (info != null) {
        pw.println(prefix + "resizeMode=" + ActivityInfo.resizeModeToString(info.resizeMode));
        pw.println(prefix + "mLastReportedMultiWindowMode=" + mLastReportedMultiWindowMode
                + " mLastReportedPictureInPictureMode=" + mLastReportedPictureInPictureMode);
        if (info.supportsPictureInPicture()) {
            pw.println(prefix + "supportsPictureInPicture=" + info.supportsPictureInPicture());
            pw.println(prefix + "supportsEnterPipOnTaskSwitch: "
                    + supportsEnterPipOnTaskSwitch);
        }
        if (info.maxAspectRatio != 0) {
            pw.println(prefix + "maxAspectRatio=" + info.maxAspectRatio);
        }
    }
}
```


