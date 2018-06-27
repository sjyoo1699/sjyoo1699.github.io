---
layout: post
title: "180626 ACTIVITY RECORD THIRD"
date:   2018-06-26 19:00:30 +0900
categories: jekyll update
---

ActivityRecord 클래스는 TaskRecord와 같이 ConfigurationContainer 클래스를 상속받아 확장하고 있고, TaskRecord와는 다르게 
AppWindowContainerListener를 implements하고 있다. (TaskRecord는 TaskWindowContainerListener를 implements) Activity는 
사용자와 상호작용하는 하나의 화면을 나타내는 객체이기 때문에 ActivityRecord.java 클래스에는 실제 화면 출력과 관련된 함수들이
많고, 그에 따라 AppWindowContainerListener를 implements하는 것으로 보인다. 

ActivityRecord의 함수들을 보던 중 오버라이드한 함수를 보게 되었고 ActivityRecord가 leaf node인 것을 확인하였다.
이 함수의 parent를 따라 쭉 올라가면서, AcitivityRecord부터 연결된 구조를 확인할 수 있었다.

아래의 코드들은 순서대로 parent가 null이 될 때까지 연결된 구조를 찾은 것이다.


```

    //ActivityRecord.java
    
    @Override
    protected int getChildCount() {
        // {@link ActivityRecord} is a leaf node and has no children.
        return 0;
    }

    @Override
    protected ConfigurationContainer getChildAt(int index) {
        return null;
    }

    @Override
    protected ConfigurationContainer getParent() {
        return getTask();
    }
    
    //TaskRecord.java
    
    @Override
    protected int getChildCount() {
        return mActivities.size();
    }

    @Override
    protected ConfigurationContainer getChildAt(int index) {
        return mActivities.get(index);
    }

    @Override
    protected ConfigurationContainer getParent() {
        return mStack;
    }
    
    //ActivityStack.java
    
    @Override
    protected int getChildCount() {
        return mTaskHistory.size();
    }

    @Override
    protected ConfigurationContainer getChildAt(int index) {
        return mTaskHistory.get(index);
    }

    @Override
    protected ConfigurationContainer getParent() {
        return getDisplay();
    }
    
    ActivityStackSupervisor.ActivityDisplay getDisplay() {
        return mStackSupervisor.getActivityDisplay(mDisplayId);
    }

    //ActivityStackSupervisor.java
    
    // TODO: Move to its own file.
    /** Exactly one of these classes per Display in the system. Capable of holding zero or more
     * attached {@link ActivityStack}s */
    class ActivityDisplay extends ConfigurationContainer {
        /** Actual Display this object tracks. */
        int mDisplayId;
        Display mDisplay;

        /** All of the stacks on this display. Order matters, topmost stack is in front of all other
         * stacks, bottommost behind. Accessed directly by ActivityManager package classes */
        final ArrayList<ActivityStack> mStacks = new ArrayList<>();

        /** Array of all UIDs that are present on the display. */
        private IntArray mDisplayAccessUIDs = new IntArray();

        /** All tokens used to put activities on this stack to sleep (including mOffToken) */
        final ArrayList<SleepTokenImpl> mAllSleepTokens = new ArrayList<>();
        /** The token acquired by ActivityStackSupervisor to put stacks on the display to sleep */
        SleepToken mOffToken;

        private boolean mSleeping;

        @VisibleForTesting
        ActivityDisplay() {
            mActivityDisplays.put(mDisplayId, this);
        }

        // After instantiation, check that mDisplay is not null before using this. The alternative
        // is for this to throw an exception if mDisplayManager.getDisplay() returns null.
        ActivityDisplay(int displayId) {
            final Display display = mDisplayManager.getDisplay(displayId);
            if (display == null) {
                return;
            }
            init(display);
        }

        void init(Display display) {
            mDisplay = display;
            mDisplayId = display.getDisplayId();
        }

        void attachStack(ActivityStack stack, int position) {
            if (DEBUG_STACK) Slog.v(TAG_STACK, "attachStack: attaching " + stack
                    + " to displayId=" + mDisplayId + " position=" + position);
            mStacks.add(position, stack);
            mService.updateSleepIfNeededLocked();
        }

        void detachStack(ActivityStack stack) {
            if (DEBUG_STACK) Slog.v(TAG_STACK, "detachStack: detaching " + stack
                    + " from displayId=" + mDisplayId);
            mStacks.remove(stack);
            mService.updateSleepIfNeededLocked();
        }

        @Override
        public String toString() {
            return "ActivityDisplay={" + mDisplayId + " numStacks=" + mStacks.size() + "}";
        }

        @Override
        protected int getChildCount() {
            return mStacks.size();
        }

        @Override
        protected ConfigurationContainer getChildAt(int index) {
            return mStacks.get(index);
        }

        @Override
        protected ConfigurationContainer getParent() {
            return ActivityStackSupervisor.this;
        }

        boolean isPrivate() {
            return (mDisplay.getFlags() & FLAG_PRIVATE) != 0;
        }

        boolean isUidPresent(int uid) {
            for (ActivityStack stack : mStacks) {
                if (stack.isUidPresent(uid)) {
                    return true;
                }
            }
            return false;
        }

        /** Update and get all UIDs that are present on the display and have access to it. */
        private IntArray getPresentUIDs() {
            mDisplayAccessUIDs.clear();
            for (ActivityStack stack : mStacks) {
                stack.getPresentUIDs(mDisplayAccessUIDs);
            }
            return mDisplayAccessUIDs;
        }

        boolean shouldDestroyContentOnRemove() {
            return mDisplay.getRemoveMode() == REMOVE_MODE_DESTROY_CONTENT;
        }

        boolean shouldSleep() {
            return (mStacks.isEmpty() || !mAllSleepTokens.isEmpty())
                    && (mService.mRunningVoice == null);
        }

        boolean isSleeping() {
            return mSleeping;
        }

        void setIsSleeping(boolean asleep) {
            mSleeping = asleep;
        }
    }
    
    
    ///////////////////////
    @Override
    protected int getChildCount() {
        return mActivityDisplays.size();
    }

    @Override
    protected ActivityDisplay getChildAt(int index) {
        return mActivityDisplays.valueAt(index);
    }

    @Override
    protected ConfigurationContainer getParent() {
        return null;
    }
    
```

