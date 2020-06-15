
MotionLayout keeps track of any nested scrollable view in its mScrollTarget field.
This causes a memory leak when that view has a different lifecycle than the MotionLayout.
For example: a NestedScrollView is inside a Fragment that gets removed from the MotionLayout.

This happens with MotionLayout 2.0.0-beta5, 2.0.0-beta6 and *2.0.0-beta7*. It did not happen in 2.0.0-beta4.

To reproduce:
- Start the app
- Click on the button to go to the second fragment
- Scroll through the images
- Press back to close the second fragment
- Press Home

Leak Canary detects the following leak:

```
hprof: heap dump "/data/user/0/be.norio.sandbox.motionlayoutleak/files/leakcanary/2020-06-15_10-21-02_146.hprof" starting...
hprof: heap dump completed (15MB) in 632.111ms objects 238748 objects with stack traces 0
Analysis in progress, working on: PARSING_HEAP_DUMP
Analysis in progress, working on: EXTRACTING_METADATA
Analysis in progress, working on: FINDING_RETAINED_OBJECTS
Analysis in progress, working on: FINDING_PATHS_TO_RETAINED_OBJECTS
Analysis in progress, working on: FINDING_DOMINATORS
Found 2 retained objects
Analysis in progress, working on: COMPUTING_NATIVE_RETAINED_SIZE
Analysis in progress, working on: COMPUTING_RETAINED_SIZE
Analysis in progress, working on: BUILDING_LEAK_TRACES
Found 2 paths to retained objects, down to 1 after removing duplicated paths
Analysis in progress, working on: REPORTING_HEAP_ANALYSIS
====================================
HEAP ANALYSIS RESULT
====================================
1 APPLICATION LEAKS
References underlined with "~~~" are likely causes.
Learn more at https://squ.re/leaks.
2478 bytes retained by leaking objects
Signature: 1ac98bd3de32512986d7573ad8bd69aaaf51c7c8
┬───
│ GC Root: System class
│
├─ android.view.inputmethod.InputMethodManager class
│    Leaking: NO (InputMethodManager↓ is not leaking and a class is never leaking)
│    ↓ static InputMethodManager.sInstance
├─ android.view.inputmethod.InputMethodManager instance
│    Leaking: NO (DecorView↓ is not leaking and InputMethodManager is a singleton)
│    ↓ InputMethodManager.mNextServedView
├─ com.android.internal.policy.DecorView instance
│    Leaking: NO (LinearLayout↓ is not leaking and View attached)
│    mContext instance of com.android.internal.policy.DecorContext, wrapping activity be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
│    Parent android.view.ViewRootImpl not a android.view.View
│    View#mParent is set
│    View#mAttachInfo is not null (view attached)
│    View.mWindowAttachCount = 1
│    ↓ DecorView.mContentRoot
├─ android.widget.LinearLayout instance
│    Leaking: NO (MainActivity↓ is not leaking and View attached)
│    mContext instance of be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
│    View.parent com.android.internal.policy.DecorView attached as well
│    View#mParent is set
│    View#mAttachInfo is not null (view attached)
│    View.mWindowAttachCount = 1
│    ↓ LinearLayout.mContext
├─ be.norio.sandbox.motionlayoutleak.MainActivity instance
│    Leaking: NO (ContentFrameLayout↓ is not leaking and Activity#mDestroyed is false)
│    ↓ MainActivity.mDelegate
├─ androidx.appcompat.app.AppCompatDelegateImpl instance
│    Leaking: NO (ContentFrameLayout↓ is not leaking)
│    ↓ AppCompatDelegateImpl.mActionBar
├─ androidx.appcompat.app.WindowDecorActionBar instance
│    Leaking: NO (ContentFrameLayout↓ is not leaking)
│    ↓ WindowDecorActionBar.mContentView
├─ androidx.appcompat.widget.ContentFrameLayout instance
│    Leaking: NO (MotionLayout↓ is not leaking and View attached)
│    mContext instance of androidx.appcompat.view.ContextThemeWrapper, wrapping activity be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
│    View.parent androidx.appcompat.widget.ActionBarOverlayLayout attached as well
│    View#mParent is set
│    View#mAttachInfo is not null (view attached)
│    View.mID = R.id.null
│    View.mWindowAttachCount = 1
│    ↓ ContentFrameLayout.mChildren
├─ android.view.View[] array
│    Leaking: NO (MotionLayout↓ is not leaking)
│    ↓ View[].[0]
├─ androidx.constraintlayout.motion.widget.MotionLayout instance
│    Leaking: NO (View attached)
│    mContext instance of be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
│    View.parent androidx.appcompat.widget.ContentFrameLayout attached as well
│    View#mParent is set
│    View#mAttachInfo is not null (view attached)
│    View.mWindowAttachCount = 1
│    ↓ MotionLayout.mScrollTarget
│                   ~~~~~~~~~~~~~
├─ androidx.core.widget.NestedScrollView instance
│    Leaking: YES (View detached and has parent)
│    mContext instance of be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
│    View#mParent is set
│    View#mAttachInfo is null (view detached)
│    View.mWindowAttachCount = 1
│    ↓ NestedScrollView.mParent
╰→ android.widget.FrameLayout instance
​     Leaking: YES (ObjectWatcher was watching this because be.norio.sandbox.motionlayoutleak.FragmentTwo received Fragment#onDestroyView() callback (references to its views should be cleared to prevent leaks))
​     key = c6cfcd15-f899-46ea-8079-220c8f3ad68f
​     watchDurationMillis = 8219
​     retainedDurationMillis = 3181
​     mContext instance of be.norio.sandbox.motionlayoutleak.MainActivity with mDestroyed = false
​     View#mParent is null
​     View#mAttachInfo is null (view detached)
​     View.mWindowAttachCount = 1
====================================
0 LIBRARY LEAKS
A Library Leak is a leak caused by a known bug in 3rd party code that you do not have control over.
See https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/#4-categorizing-leaks
====================================
METADATA
Please include this in bug reports and Stack Overflow questions.
Build.VERSION.SDK_INT: 29
Build.MANUFACTURER: Google
LeakCanary version: 2.3
App process name: be.norio.sandbox.motionlayoutleak
Analysis duration: 5423 ms
Heap dump file path: /data/user/0/be.norio.sandbox.motionlayoutleak/files/leakcanary/2020-06-15_10-21-02_146.hprof
Heap dump timestamp: 1592209268241
====================================

```