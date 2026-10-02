# Part 63: MotionLayout - Complex Animations
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 63.1 MotionLayout คืออะไร

MotionLayout เป็น subclass ของ ConstraintLayout ที่จัดการ animation ระหว่าง 2 states

```
ConstraintLayout → ConstraintSet (layout state)
MotionLayout    → MotionScene (animation between states)

ข้อดี:
- ไม่ต้องเขียน Java animation code
- Declarative XML
- รองรับ gesture (swipe to animate)
- Preview ใน Android Studio
- ซับซ้อนมากกว่า Transition/Animator
```

---

## 63.2 Basic MotionLayout

```xml
<!-- res/layout/activity_motion.xml -->
<androidx.constraintlayout.motion.widget.MotionLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/motionLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/motion_scene">
    
    <!-- Views to animate -->
    <ImageView
        android:id="@+id/ivProfile"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:src="@drawable/ic_person" />
    
    <TextView
        android:id="@+id/tvName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="John Doe"
        android:textSize="18sp" />
    
    <View
        android:id="@+id/headerBackground"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:background="@color/primary" />

</androidx.constraintlayout.motion.widget.MotionLayout>

<!-- res/xml/motion_scene.xml -->
<MotionScene
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:motion="http://schemas.android.com/apk/res-auto">
    
    <!-- State 1: Expanded (start) -->
    <ConstraintSet android:id="@+id/start">
        <Constraint android:id="@+id/ivProfile">
            <Layout
                android:layout_width="100dp"
                android:layout_height="100dp"
                motion:layout_constraintTop_toTopOf="parent"
                motion:layout_constraintStart_toStartOf="parent"
                android:layout_marginTop="16dp"
                android:layout_marginStart="16dp" />
            <Transform android:scaleX="1" android:scaleY="1" android:alpha="1" />
        </Constraint>
        
        <Constraint android:id="@+id/tvName">
            <Layout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                motion:layout_constraintTop_toBottomOf="@id/ivProfile"
                motion:layout_constraintStart_toStartOf="@id/ivProfile"
                android:layout_marginTop="8dp" />
            <Transform android:translationX="0" android:alpha="1" />
        </Constraint>
        
        <Constraint android:id="@+id/headerBackground">
            <Layout
                android:layout_width="match_parent"
                android:layout_height="200dp"
                motion:layout_constraintTop_toTopOf="parent" />
        </Constraint>
    </ConstraintSet>
    
    <!-- State 2: Collapsed (end) - scrolled up -->
    <ConstraintSet android:id="@+id/end">
        <Constraint android:id="@+id/ivProfile">
            <Layout
                android:layout_width="40dp"
                android:layout_height="40dp"
                motion:layout_constraintTop_toTopOf="parent"
                motion:layout_constraintStart_toStartOf="parent"
                android:layout_marginTop="8dp"
                android:layout_marginStart="8dp" />
            <Transform android:scaleX="0.4" android:scaleY="0.4" android:alpha="0.8" />
        </Constraint>
        
        <Constraint android:id="@+id/tvName">
            <Layout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                motion:layout_constraintTop_toTopOf="parent"
                motion:layout_constraintStart_toEndOf="@id/ivProfile"
                android:layout_marginTop="12dp"
                android:layout_marginStart="8dp" />
            <Transform android:translationX="0" android:alpha="1" />
        </Constraint>
        
        <Constraint android:id="@+id/headerBackground">
            <Layout
                android:layout_width="match_parent"
                android:layout_height="60dp"
                motion:layout_constraintTop_toTopOf="parent" />
        </Constraint>
    </ConstraintSet>
    
    <!-- Transition -->
    <Transition
        motion:constraintSetStart="@id/start"
        motion:constraintSetEnd="@id/end"
        motion:duration="300"
        motion:interpolator="easeInOut">
        
        <!-- Trigger by scrolling -->
        <OnSwipe
            motion:dragDirection="dragUp"
            motion:touchAnchorId="@id/motionLayout"
            motion:touchAnchorSide="top" />
    </Transition>
    
</MotionScene>
```

---

## 63.3 KeyFrames - Custom Animation Path

```xml
<!-- In Transition block -->
<KeyFrameSet>
    <!-- KeyPosition: change position at specific time -->
    <KeyPosition
        motion:motionTarget="@id/ivProfile"
        motion:framePosition="50"          <!-- 50% of animation -->
        motion:keyPositionType="parentRelative"
        motion:percentX="0.5"
        motion:percentY="0.2" />
    
    <!-- KeyAttribute: change attribute at specific time -->
    <KeyAttribute
        motion:motionTarget="@id/tvName"
        motion:framePosition="30"
        android:alpha="0.5"
        android:scaleX="0.8"
        android:scaleY="0.8" />
    
    <!-- KeyCycle: oscillation animation -->
    <KeyCycle
        motion:motionTarget="@id/ivProfile"
        motion:framePosition="0"
        android:translationY="0"
        motion:waveShape="sin"
        motion:wavePeriod="1"
        motion:waveOffset="0" />
</KeyFrameSet>
```

---

## 63.4 MotionLayout with ScrollView

```java
// Sync MotionLayout with NestedScrollView
public class MotionScrollActivity extends AppCompatActivity {
    
    private MotionLayout motionLayout;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_motion_scroll);
        
        motionLayout = findViewById(R.id.motionLayout);
        NestedScrollView scrollView = findViewById(R.id.nestedScrollView);
        
        scrollView.setOnScrollChangeListener(
            (NestedScrollView.OnScrollChangeListener) (v, scrollX, scrollY, oldX, oldY) -> {
                int headerHeight = 200;  // header height in pixels
                float progress = Math.min(1f, (float) scrollY / headerHeight);
                motionLayout.setProgress(progress);
            });
    }
}

// Programmatic control
motionLayout.setProgress(0f);           // start state
motionLayout.setProgress(1f);           // end state
motionLayout.transitionToEnd();         // animate to end
motionLayout.transitionToStart();       // animate to start

motionLayout.addTransitionListener(new MotionLayout.TransitionListener() {
    @Override
    public void onTransitionStarted(MotionLayout layout, int startId, int endId) {}
    
    @Override
    public void onTransitionChange(MotionLayout layout, int startId, int endId, float progress) {
        // progress: 0.0 - 1.0
    }
    
    @Override
    public void onTransitionCompleted(MotionLayout layout, int currentId) {
        if (currentId == R.id.end) {
            // Animation reached end state
        }
    }
    
    @Override
    public void onTransitionTrigger(MotionLayout layout, int triggerId, boolean positive, float progress) {}
});
```

---

## 63.5 Multi-State MotionLayout

```xml
<!-- MotionScene with 3 states -->
<MotionScene ...>
    
    <ConstraintSet android:id="@+id/stateNormal" />
    <ConstraintSet android:id="@+id/stateExpanded" />
    <ConstraintSet android:id="@+id/stateCollapsed" />
    
    <!-- Transition Normal → Expanded -->
    <Transition
        android:id="@+id/transitionExpand"
        motion:constraintSetStart="@id/stateNormal"
        motion:constraintSetEnd="@id/stateExpanded" />
    
    <!-- Transition Normal → Collapsed -->
    <Transition
        android:id="@+id/transitionCollapse"
        motion:constraintSetStart="@id/stateNormal"
        motion:constraintSetEnd="@id/stateCollapsed" />

</MotionScene>
```

```java
// Transition between states programmatically
motionLayout.transitionToState(R.id.stateExpanded);
motionLayout.transitionToState(R.id.stateCollapsed, 500);  // with duration
```

---

## 63.6 สรุป Part 63

ในบทนี้คุณได้เรียนรู้:

✅ MotionLayout vs Animator  
✅ MotionScene XML structure  
✅ ConstraintSet (start/end states)  
✅ Transition (duration, interpolator)  
✅ OnSwipe gesture trigger  
✅ KeyFrames (KeyPosition, KeyAttribute, KeyCycle)  
✅ Sync MotionLayout with scroll position  
✅ Multi-state MotionLayout  

---

*[← Part 62: App Widgets](./part-62-android-widgets.md) | [Part 64: DataStore →](./part-64-android-datastore.md)*
