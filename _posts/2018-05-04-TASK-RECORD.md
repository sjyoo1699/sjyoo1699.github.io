---
layout: post
title: "180504 <TaskRecord.java>"
date:   2018-05-04 16:06:30 +0900
categories: jekyll update
---

 처음 추측한 대로라면 TaskRecord는 ActivityStack에서 activityRecord로 리스트를 만들고, 이 리스트를 통해 TaskRecord를 생성할 것이라고 추측하였다. 그 이유는 TaskRecord가 attribute로 activityRecord의 리스트를 가지고 있고, activityStack을 가지고 있기 때문이었다. 따라서, 이에 대한 답을 얻기 위해 TaskRecord의 생성자를 살펴보았다.

생성자가 총 3개 있었다. 2개는 protected로 선언된 생성자였고, 1개는 private으로 선언된 생성자였다. protected로 선언된 생성자에서는 list를 아규먼트로 받지 않고, 새로 객체를 할당해주었다. 하지만 private 생성자는 엄청나게 많은 아규먼트가 있었고, 그 중 list를 받아서 초기화 해주는 부분이 있었는데, 따라서 이 생성자를 호출하는 생성메소드를 찾게 되었다. 

그렇게 찾은 메소드가 아래의 메소드이다. 

아래의 메소드를 살펴보면 restoreFromXml 이라는 이름의 메소드인데, Xml파일을 파싱해서 그 파싱한 결과로 TaskRecord 객체를 생성해주는 메소드이다. Xml객체를 파싱해서 정보를 가져오는데, 이 Xml 파일은 추측하건데 안드로이드 어플리케이션이 하나씩 가지고 있는 manifest.xml 인 것 같다.

manifest.xml 파일에는 어플리케이션의 정보가 기술되어 있다. 이 파일은 개발자가 작성할 수도 있지만, 툴이 자체적으로 빌드할 때 구조를 파악해서 작성하는 것으로 보인다. 따라서 이 파일에는 어플리케이션 내의 액티비티들의 구조와 설명이 기술되어 있는 것이다.

이렇게 파싱한 결과들로 TaskRecord를 생성하는데 여기서 흥미로운 점은, 메소드 내부에 반복문 부분에서 activity tag를 만나면 activityRecord를 생성해주는데, 이 또한 activityRecord의 restoreFromXml 함수를 호출해서 생성한다. 이렇게 생성한 activityRecord들을 리스트에 저장한 후에, 이 리스트를 이용하여 TaskRecord 객체를 생성한다.

TaskRecord 객체를 생성하고 나서, TaskRecord 객체에 아규먼트로 넘겨 준 activityRecord들의 리스트들을 하나하나 위에서 생성한 Task와 바인딩 시켜주고 메소드가 종료된다.

이 뜻은 activityRecord가 먼저 만들어지고, TaskRecord가 만들어지는 것이 아니라, 둘이 거의 동시에 만들어지고 바인딩 되는 것이다. 그리고 하나의 어플리케이션이 하나의 Task로 만들어진다는 점이 처음에 추측한 것과 달랐다. 

물론 아직 확실하지는 않고, 의문점이 있기에 TaskRecord.java의 코드를 더 살펴봐야 할 것 같다.



```
static TaskRecord restoreFromXml(XmlPullParser in, ActivityStackSupervisor stackSupervisor)
            throws IOException, XmlPullParserException {
        Intent intent = null;
        Intent affinityIntent = null;
        ArrayList<ActivityRecord> activities = new ArrayList<>();
        ComponentName realActivity = null;
        boolean realActivitySuspended = false;
        ComponentName origActivity = null;
        String affinity = null;
        String rootAffinity = null;
        boolean hasRootAffinity = false;
        boolean rootHasReset = false;
        boolean autoRemoveRecents = false;
        boolean askedCompatMode = false;
        int taskType = ActivityRecord.APPLICATION_ACTIVITY_TYPE;
        int userId = 0;
        boolean userSetupComplete = true;
        int effectiveUid = -1;
        String lastDescription = null;
        long firstActiveTime = -1;
        long lastActiveTime = -1;
        long lastTimeOnTop = 0;
        boolean neverRelinquishIdentity = true;
        int taskId = INVALID_TASK_ID;
        final int outerDepth = in.getDepth();
        TaskDescription taskDescription = new TaskDescription();
        TaskThumbnailInfo thumbnailInfo = new TaskThumbnailInfo();
        int taskAffiliation = INVALID_TASK_ID;
        int taskAffiliationColor = 0;
        int prevTaskId = INVALID_TASK_ID;
        int nextTaskId = INVALID_TASK_ID;
        int callingUid = -1;
        String callingPackage = "";
        int resizeMode = RESIZE_MODE_FORCE_RESIZEABLE;
        boolean supportsPictureInPicture = false;
        boolean privileged = false;
        Rect bounds = null;
        int minWidth = INVALID_MIN_SIZE;
        int minHeight = INVALID_MIN_SIZE;
        int persistTaskVersion = 0;

        for (int attrNdx = in.getAttributeCount() - 1; attrNdx >= 0; --attrNdx) {
            final String attrName = in.getAttributeName(attrNdx);
            final String attrValue = in.getAttributeValue(attrNdx);
            if (TaskPersister.DEBUG) Slog.d(TaskPersister.TAG, "TaskRecord: attribute name=" +
                    attrName + " value=" + attrValue);
            if (ATTR_TASKID.equals(attrName)) {
                if (taskId == INVALID_TASK_ID) taskId = Integer.parseInt(attrValue);
            } else if (ATTR_REALACTIVITY.equals(attrName)) {
                realActivity = ComponentName.unflattenFromString(attrValue);
            } else if (ATTR_REALACTIVITY_SUSPENDED.equals(attrName)) {
                realActivitySuspended = Boolean.valueOf(attrValue);
            } else if (ATTR_ORIGACTIVITY.equals(attrName)) {
                origActivity = ComponentName.unflattenFromString(attrValue);
            } else if (ATTR_AFFINITY.equals(attrName)) {
                affinity = attrValue;
            } else if (ATTR_ROOT_AFFINITY.equals(attrName)) {
                rootAffinity = attrValue;
                hasRootAffinity = true;
            } else if (ATTR_ROOTHASRESET.equals(attrName)) {
                rootHasReset = Boolean.parseBoolean(attrValue);
            } else if (ATTR_AUTOREMOVERECENTS.equals(attrName)) {
                autoRemoveRecents = Boolean.parseBoolean(attrValue);
            } else if (ATTR_ASKEDCOMPATMODE.equals(attrName)) {
                askedCompatMode = Boolean.parseBoolean(attrValue);
            } else if (ATTR_USERID.equals(attrName)) {
                userId = Integer.parseInt(attrValue);
            } else if (ATTR_USER_SETUP_COMPLETE.equals(attrName)) {
                userSetupComplete = Boolean.parseBoolean(attrValue);
            } else if (ATTR_EFFECTIVE_UID.equals(attrName)) {
                effectiveUid = Integer.parseInt(attrValue);
            } else if (ATTR_TASKTYPE.equals(attrName)) {
                taskType = Integer.parseInt(attrValue);
            } else if (ATTR_FIRSTACTIVETIME.equals(attrName)) {
                firstActiveTime = Long.parseLong(attrValue);
            } else if (ATTR_LASTACTIVETIME.equals(attrName)) {
                lastActiveTime = Long.parseLong(attrValue);
            } else if (ATTR_LASTDESCRIPTION.equals(attrName)) {
                lastDescription = attrValue;
            } else if (ATTR_LASTTIMEMOVED.equals(attrName)) {
                lastTimeOnTop = Long.parseLong(attrValue);
            } else if (ATTR_NEVERRELINQUISH.equals(attrName)) {
                neverRelinquishIdentity = Boolean.parseBoolean(attrValue);
            } else if (attrName.startsWith(TaskThumbnailInfo.ATTR_TASK_THUMBNAILINFO_PREFIX)) {
                thumbnailInfo.restoreFromXml(attrName, attrValue);
            } else if (attrName.startsWith(TaskDescription.ATTR_TASKDESCRIPTION_PREFIX)) {
                taskDescription.restoreFromXml(attrName, attrValue);
            } else if (ATTR_TASK_AFFILIATION.equals(attrName)) {
                taskAffiliation = Integer.parseInt(attrValue);
            } else if (ATTR_PREV_AFFILIATION.equals(attrName)) {
                prevTaskId = Integer.parseInt(attrValue);
            } else if (ATTR_NEXT_AFFILIATION.equals(attrName)) {
                nextTaskId = Integer.parseInt(attrValue);
            } else if (ATTR_TASK_AFFILIATION_COLOR.equals(attrName)) {
                taskAffiliationColor = Integer.parseInt(attrValue);
            } else if (ATTR_CALLING_UID.equals(attrName)) {
                callingUid = Integer.parseInt(attrValue);
            } else if (ATTR_CALLING_PACKAGE.equals(attrName)) {
                callingPackage = attrValue;
            } else if (ATTR_RESIZE_MODE.equals(attrName)) {
                resizeMode = Integer.parseInt(attrValue);
            } else if (ATTR_SUPPORTS_PICTURE_IN_PICTURE.equals(attrName)) {
                supportsPictureInPicture = Boolean.parseBoolean(attrValue);
            } else if (ATTR_PRIVILEGED.equals(attrName)) {
                privileged = Boolean.parseBoolean(attrValue);
            } else if (ATTR_NON_FULLSCREEN_BOUNDS.equals(attrName)) {
                bounds = Rect.unflattenFromString(attrValue);
            } else if (ATTR_MIN_WIDTH.equals(attrName)) {
                minWidth = Integer.parseInt(attrValue);
            } else if (ATTR_MIN_HEIGHT.equals(attrName)) {
                minHeight = Integer.parseInt(attrValue);
            } else if (ATTR_PERSIST_TASK_VERSION.equals(attrName)) {
                persistTaskVersion = Integer.parseInt(attrValue);
            } else {
                Slog.w(TAG, "TaskRecord: Unknown attribute=" + attrName);
            }
        }

        int event;
        while (((event = in.next()) != XmlPullParser.END_DOCUMENT) &&
                (event != XmlPullParser.END_TAG || in.getDepth() >= outerDepth)) {
            if (event == XmlPullParser.START_TAG) {
                final String name = in.getName();
                if (TaskPersister.DEBUG) Slog.d(TaskPersister.TAG, "TaskRecord: START_TAG name=" +
                        name);
                if (TAG_AFFINITYINTENT.equals(name)) {
                    affinityIntent = Intent.restoreFromXml(in);
                } else if (TAG_INTENT.equals(name)) {
                    intent = Intent.restoreFromXml(in);
                } else if (TAG_ACTIVITY.equals(name)) {
                    ActivityRecord activity = ActivityRecord.restoreFromXml(in, stackSupervisor);
                    if (TaskPersister.DEBUG) Slog.d(TaskPersister.TAG, "TaskRecord: activity=" +
                            activity);
                    if (activity != null) {
                        activities.add(activity);
                    }
                } else {
                    Slog.e(TAG, "restoreTask: Unexpected name=" + name);
                    XmlUtils.skipCurrentTag(in);
                }
            }
        }
        if (!hasRootAffinity) {
            rootAffinity = affinity;
        } else if ("@".equals(rootAffinity)) {
            rootAffinity = null;
        }
        if (effectiveUid <= 0) {
            Intent checkIntent = intent != null ? intent : affinityIntent;
            effectiveUid = 0;
            if (checkIntent != null) {
                IPackageManager pm = AppGlobals.getPackageManager();
                try {
                    ApplicationInfo ai = pm.getApplicationInfo(
                            checkIntent.getComponent().getPackageName(),
                            PackageManager.MATCH_UNINSTALLED_PACKAGES
                                    | PackageManager.MATCH_DISABLED_COMPONENTS, userId);
                    if (ai != null) {
                        effectiveUid = ai.uid;
                    }
                } catch (RemoteException e) {
                }
            }
            Slog.w(TAG, "Updating task #" + taskId + " for " + checkIntent
                    + ": effectiveUid=" + effectiveUid);
        }

        if (persistTaskVersion < 1) {
            // We need to convert the resize mode of home activities saved before version one if
            // they are marked as RESIZE_MODE_RESIZEABLE to RESIZE_MODE_RESIZEABLE_VIA_SDK_VERSION
            // since we didn't have that differentiation before version 1 and the system didn't
            // resize home activities before then.
            if (taskType == HOME_ACTIVITY_TYPE && resizeMode == RESIZE_MODE_RESIZEABLE) {
                resizeMode = RESIZE_MODE_RESIZEABLE_VIA_SDK_VERSION;
            }
        } else {
            // This activity has previously marked itself explicitly as both resizeable and
            // supporting picture-in-picture.  Since there is no longer a requirement for
            // picture-in-picture activities to be resizeable, we can mark this simply as
            // resizeable and supporting picture-in-picture separately.
            if (resizeMode == RESIZE_MODE_RESIZEABLE_AND_PIPABLE_DEPRECATED) {
                resizeMode = RESIZE_MODE_RESIZEABLE;
                supportsPictureInPicture = true;
            }
        }

        final TaskRecord task = new TaskRecord(stackSupervisor.mService, taskId, intent,
                affinityIntent, affinity, rootAffinity, realActivity, origActivity, rootHasReset,
                autoRemoveRecents, askedCompatMode, taskType, userId, effectiveUid, lastDescription,
                activities, firstActiveTime, lastActiveTime, lastTimeOnTop, neverRelinquishIdentity,
                taskDescription, thumbnailInfo, taskAffiliation, prevTaskId, nextTaskId,
                taskAffiliationColor, callingUid, callingPackage, resizeMode,
                supportsPictureInPicture, privileged, realActivitySuspended, userSetupComplete,
                minWidth, minHeight);
        task.updateOverrideConfiguration(bounds);

        for (int activityNdx = activities.size() - 1; activityNdx >=0; --activityNdx) {
            activities.get(activityNdx).setTask(task);
        }

        if (DEBUG_RECENTS) Slog.d(TAG_RECENTS, "Restored task=" + task);
        return task;
    }
```

