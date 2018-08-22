---
layout: post
title: "180819 ACTIVITYTHREAD"
date:   2018-08-19 14:00:30 +0900
categories: jekyll update
---

지난 포스팅에서 ActivityThread.java 내의 performLaunchActivity 함수에 대해서 알아보았었는데,

이번에는 다른 생명주기에 관한 함수을 알아보고자 한다.

아래는 performPauseActivity 함수이다.

***

```
    /**
     * Pause the activity.
     * @return Saved instance state for pre-Honeycomb apps if it was saved, {@code null} otherwise.
     */
    private Bundle performPauseActivity(ActivityClientRecord r, boolean finished, String reason,
            PendingTransactionActions pendingActions) {
        if (r.paused) {
            if (r.activity.mFinished) {
                // If we are finishing, we won't call onResume() in certain cases.
                // So here we likewise don't want to call onPause() if the activity
                // isn't resumed.
                return null;
            }
            RuntimeException e = new RuntimeException(
                    "Performing pause of activity that is not resumed: "
                    + r.intent.getComponent().toShortString());
            Slog.e(TAG, e.getMessage(), e);
        }
        if (finished) {
            r.activity.mFinished = true;
        }
        // Pre-Honeycomb apps always save their state before pausing
        final boolean shouldSaveState = !r.activity.mFinished && r.isPreHoneycomb();
        if (shouldSaveState) {
            callActivityOnSaveInstanceState(r);
        }
        performPauseActivityIfNeeded(r, reason);
        // Notify any outstanding on paused listeners
        ArrayList<OnActivityPausedListener> listeners;
        synchronized (mOnPauseListeners) {
            listeners = mOnPauseListeners.remove(r.activity);
        }
        int size = (listeners != null ? listeners.size() : 0);
        for (int i = 0; i < size; i++) {
            listeners.get(i).onPaused(r.activity);
        }
        final Bundle oldState = pendingActions != null ? pendingActions.getOldState() : null;
        if (oldState != null) {
            // We need to keep around the original state, in case we need to be created again.
            // But we only do this for pre-Honeycomb apps, which always save their state when
            // pausing, so we can not have them save their state when restarting from a paused
            // state. For HC and later, we want to (and can) let the state be saved as the
            // normal part of stopping the activity.
            if (r.isPreHoneycomb()) {
                r.state = oldState;
            }
        }
        return shouldSaveState ? r.state : null;
    }
```

