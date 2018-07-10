---
layout: post
title: "180702 ABOUT DUMP FUNCTION IN ACTIVITY RECORD"
date:   2018-07-02 14:00:30 +0900
categories: jekyll update
---


지난 포스팅 중에 Activity Record 에서 dump 함수에서 printWriter를 쓰는 것보다 bufferedWriter를 쓰는 것이 효율적일 것 같다고 언급했다.
오늘은 그에 대해 자세히 알아보려고 한다.

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


먼저 IO 스트림에서 출력 버퍼가 성능에 미치는 영향에 대해서 살펴보겠다.

# IO 스트림에서 출력 버퍼가 성능에 미치는 영향
***

IO 스트림은 파일시스템이나 다른 주변장치들과의 데이터 통신뿐만이 아니라 프로세스간 파이프 같은 데이터 통신 및 네트워크 상의 다른 컴퓨터의 
프로세스 간의 데이터 통신을 표준화된 방법으로 처리할 수 있도록 규정한 방법이다. 이런 IO 스트림은 OS 레벨에서 지원하게 되며, 고급화된 표준
IO 데이터 통신 방법이다.

IO 스트림에 데이터를 출력할 때 자바 언어의 스트림에서 제공하는 출력 스트림에 데이터를 내보내면 내부적으로 OS 차원에서 제공하는 고수준 
프로시져를 호출하고, 다시 OS 커널의 저수준 프로시져를 거치게 된다. 이 때 데이터는 하나의 덩어리로 보내지게 된다. 

이런 이유로, 1바이트의 데이터를 1000번에 걸쳐 보내는 것과, 1000바이트의 데이터를 1번에 보내는 것은 엄청난 속도 차이를 불러오게 된다. 
이것은 언어에 상관없이 OS 차원에서 지원하는 것이므로, 자바만의 특징이 아니라 IO 스트림의 특징이다. 데이터를 스트림을 통하여 출력할 때, 
큰 덩어리로 묶어서 보낼 수 있다면, 작은 단위로 나누어 출력하는 것보다 성능을 향상시킬 수 있다. 만일 데이터의 특성상 묶을 수가 없고, 작은 
단위로 반복해서 보낼 수 밖에 없는 경우에는 출력 버퍼를 사용하는 것이 좋다.

출력 버퍼는 스트림으로 출력하는 데이터를 메모리상에서 임시로 보관하고 있다가 일정량이상 모이면 한번에 출력하는 방법이다.



아래는 PrintWriter와 BufferedWriter, PrintWriter+BufferdWriter(wrapper), FileWriter 네 개의 클래스를 통해 짧은 문자열을 수만번에
걸쳐 저장하는 경우에 성능을 비교한 그래프이다. 

![fig43-1](https://user-images.githubusercontent.com/28890428/42145627-45fc6732-7dfd-11e8-918d-a7eed7c1cd37.jpg)
[출처] http://javacan.tistory.com/entry/43

- x 축은 반복횟수이며 64회 반복당 1씩 증가하며 총 6만4천번을 의미한다.
- y 축은 소요시간은 밀리초, 남은 자유 메모리는 MB이며 처음 힙 메모리의 크기는 64MB이다. 

소요시간을 살펴보면 출력버퍼를 사용하지 않은 경우가 대략 2배 정도의 시간이 더 걸린 것을 알 수 있다.

이처럼 printWriter가 bufferedWriter에 비해 현저히 느린 속도를 보여주는데, 왜 printWriter를 사용하는 것일까?

그 이유는 printWriter가 사용하기 쉽고 간편하기 때문이다. 일반적으로 우리가 자주 사용하는 print, println, printf 등이 printWriter에서
제공하는 함수이다. 

즉 printWriter와 bufferedWriter 두 방법은 각각 장점을 가지고 있다. 
printWriter는 간편함, bufferedWriter는 버퍼를 사용하여 좀 더 효율적인 파일쓰기를 지원해준다는 것이다.

AOSP에 우리 팀이 참여함으로서 달성하고자 하는 목표는 '안드로이드 OS의 성능 개선'이다.
이러한 목표에는 printWriter보다는 bufferedWriter를 활용하는 것이 좋을 것 같다.

***

이제 이러한 dump 함수가 얼마나 빈번하게 호출이 되는 지 알아봐야 하고, 
위에서 IO 스트림에 대해 공부하면서 알게 된 사실로 PrintWriter 클래스를 

```
PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("foo.txt"))); 
```

이런 식으로 만들면 BufferedWriter의 장점과 PrintWriter의 장점을 모두 사용할 수 있다. 
위의 dump 함수의 인자로 받은 PrintWriter가 이렇게 만들어져 있다면 dump 함수에서 성능 개선점은 찾기 힘들 것이다.

그래서 ActivityManager의 코드를 찾아보던 중 이런 코드를 찾았다.

'''
public boolean stopBinderTrackingAndDump(ParcelFileDescriptor fd) throws RemoteException {
        try {
            synchronized (this) {
                mBinderTransactionTrackingEnabled = false;
                // TODO: hijacking SET_ACTIVITY_WATCHER, but should be changed to its own
                // permission (same as profileControl).
                if (checkCallingPermission(android.Manifest.permission.SET_ACTIVITY_WATCHER)
                        != PackageManager.PERMISSION_GRANTED) {
                    throw new SecurityException("Requires permission "
                            + android.Manifest.permission.SET_ACTIVITY_WATCHER);
                }

                if (fd == null) {
                    throw new IllegalArgumentException("null fd");
                }

                PrintWriter pw = new FastPrintWriter(new FileOutputStream(fd.getFileDescriptor()));
                pw.println("Binder transaction traces for all processes.\n");
                for (ProcessRecord process : mLruProcesses) {
                    if (!processSanityChecksLocked(process)) {
                        continue;
                    }

                    pw.println("Traces for process: " + process.processName);
                    pw.flush();
                    try {
                        TransferPipe tp = new TransferPipe();
                        try {
                            process.thread.stopBinderTrackingAndDump(tp.getWriteFd());
                            tp.go(fd.getFileDescriptor());
                        } finally {
                            tp.kill();
                        }
                    } catch (IOException e) {
                        pw.println("Failure while dumping IPC traces from " + process +
                                ".  Exception: " + e);
                        pw.flush();
                    } catch (RemoteException e) {
                        pw.println("Got a RemoteException while dumping IPC traces from " +
                                process + ".  Exception: " + e);
                        pw.flush();
                    }
                }
                fd = null;
                return true;
            }
        } finally {
            if (fd != null) {
                try {
                    fd.close();
                } catch (IOException e) {
                }
            }
        }
    }
'''

위의 코드에서 보면 PrintWriter를 상속받은 FastPrintWriter가 있는 것을 볼 수 있다. 
FastPrintWriter의 경로는 
import com.android.internal.util.FastPrintWriter;
이것인데, 아직 이 파일을 찾아내지는 못하였다.(internal을 못 찾았음)

FastPrintWriter가 어떻게 구현되어있는 지는 아직 저 파일을 찾아내지 못해서 확신할 수 없지만, 
아마 PrintWriter + BufferedWriter를 사용하는 것으로 보여진다. (dump 메소드를 사용하면서 flush()도 호출하는 것을 보아서 더 확실해졌다.)

이렇게 되면 dump 메소드에서 성능 개선점을 찾고자 했던 것은 다시 원점으로 돌아가게 된다.
