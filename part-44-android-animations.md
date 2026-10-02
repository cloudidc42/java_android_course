# Part 44: Android Animations
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 44.1 ประเภท Animation ใน Android

```
1. View Animation (เก่า)      - translate, scale, rotate, alpha
2. Property Animation (แนะนำ) - ObjectAnimator, ValueAnimator, AnimatorSet
3. Transition Framework       - scene transitions between Activities/Fragments
4. MotionLayout               - complex coordinated animations
```

---

## 44.2 Property Animation พื้นฐาน

```java
// PropertyAnimationDemo.java
import android.animation.*;
import android.view.animation.*;

public class PropertyAnimationDemo {
    
    // Fade in/out
    public static void fadeIn(View view, long duration) {
        ObjectAnimator animator = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f);
        animator.setDuration(duration);
        animator.setInterpolator(new DecelerateInterpolator());
        animator.start();
    }
    
    public static void fadeOut(View view, long duration, Runnable onEnd) {
        ObjectAnimator animator = ObjectAnimator.ofFloat(view, "alpha", 1f, 0f);
        animator.setDuration(duration);
        animator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                view.setVisibility(View.GONE);
                if (onEnd != null) onEnd.run();
            }
        });
        animator.start();
    }
    
    // Slide in from bottom
    public static void slideInFromBottom(View view) {
        float screenH = view.getContext().getResources().getDisplayMetrics().heightPixels;
        view.setTranslationY(screenH);
        view.setVisibility(View.VISIBLE);
        
        ObjectAnimator animator = ObjectAnimator.ofFloat(view, "translationY", screenH, 0f);
        animator.setDuration(400);
        animator.setInterpolator(new DecelerateInterpolator(2f));
        animator.start();
    }
    
    // Scale bounce
    public static void scaleBounce(View view) {
        AnimatorSet set = new AnimatorSet();
        
        ObjectAnimator scaleX = ObjectAnimator.ofFloat(view, "scaleX", 1f, 1.3f, 1f);
        ObjectAnimator scaleY = ObjectAnimator.ofFloat(view, "scaleY", 1f, 1.3f, 1f);
        
        scaleX.setDuration(300);
        scaleY.setDuration(300);
        scaleX.setInterpolator(new OvershootInterpolator());
        scaleY.setInterpolator(new OvershootInterpolator());
        
        set.playTogether(scaleX, scaleY);
        set.start();
    }
    
    // Shake animation (incorrect password effect)
    public static void shake(View view) {
        float[] positions = {0, -20, 20, -20, 20, -10, 10, 0};
        ObjectAnimator animator = ObjectAnimator.ofFloat(view, "translationX", positions);
        animator.setDuration(600);
        animator.start();
    }
    
    // Rotation
    public static void rotate360(View view) {
        ObjectAnimator animator = ObjectAnimator.ofFloat(view, "rotation", 0f, 360f);
        animator.setDuration(1000);
        animator.setInterpolator(new LinearInterpolator());
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.start();
    }
}
```

---

## 44.3 ValueAnimator - Custom Animations

```java
// Counter animation (number goes from 0 to target)
public static void animateCounter(TextView textView, int from, int to, long duration) {
    ValueAnimator animator = ValueAnimator.ofInt(from, to);
    animator.setDuration(duration);
    animator.setInterpolator(new DecelerateInterpolator());
    animator.addUpdateListener(anim -> {
        textView.setText(String.valueOf((int) anim.getAnimatedValue()));
    });
    animator.start();
}

// Color animation
public static void animateBackgroundColor(View view, int fromColor, int toColor) {
    ValueAnimator animator = ValueAnimator.ofObject(new ArgbEvaluator(), fromColor, toColor);
    animator.setDuration(500);
    animator.addUpdateListener(anim -> {
        view.setBackgroundColor((int) anim.getAnimatedValue());
    });
    animator.start();
}

// Progress bar animation
public static void animateProgress(ProgressBar progressBar, int target) {
    ObjectAnimator animator = ObjectAnimator.ofInt(progressBar, "progress", 0, target);
    animator.setDuration(1000);
    animator.setInterpolator(new DecelerateInterpolator());
    animator.start();
}
```

---

## 44.4 AnimatorSet - Sequence & Together

```java
// Login success animation
public static void playLoginSuccess(ImageView checkIcon, TextView statusText) {
    // Step 1: Scale icon
    AnimatorSet step1 = new AnimatorSet();
    step1.playTogether(
        ObjectAnimator.ofFloat(checkIcon, "scaleX", 0f, 1.2f),
        ObjectAnimator.ofFloat(checkIcon, "scaleY", 0f, 1.2f),
        ObjectAnimator.ofFloat(checkIcon, "alpha", 0f, 1f)
    );
    step1.setDuration(300);
    
    // Step 2: Settle to normal size
    AnimatorSet step2 = new AnimatorSet();
    step2.playTogether(
        ObjectAnimator.ofFloat(checkIcon, "scaleX", 1.2f, 1f),
        ObjectAnimator.ofFloat(checkIcon, "scaleY", 1.2f, 1f)
    );
    step2.setDuration(150);
    
    // Step 3: Fade in text
    ObjectAnimator step3 = ObjectAnimator.ofFloat(statusText, "alpha", 0f, 1f);
    step3.setDuration(300);
    
    // Play in sequence
    AnimatorSet fullSet = new AnimatorSet();
    fullSet.playSequentially(step1, step2, step3);
    fullSet.start();
}
```

---

## 44.5 Transition API (Activity Transitions)

```java
// Launching with shared element transition
// In source Activity:
public void openDetail(View sharedView, String itemId) {
    Intent intent = new Intent(this, DetailActivity.class);
    intent.putExtra("itemId", itemId);
    
    ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(
        this,
        sharedView,
        "sharedImage"  // transition name - must match in target
    );
    startActivity(intent, options.toBundle());
}

// In source layout:
// android:transitionName="sharedImage"

// In DetailActivity:
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_detail);
    
    // Support transitions
    postponeEnterTransition();  // wait for data to load
    
    ImageView imageView = findViewById(R.id.detailImage);
    // Load image, then:
    imageView.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
        @Override
        public boolean onPreDraw() {
            imageView.getViewTreeObserver().removeOnPreDrawListener(this);
            startPostponedEnterTransition();  // now start transition
            return true;
        }
    });
}
```

---

## 44.6 Layout Animations (RecyclerView)

```java
// AnimationUtil.java
public class AnimationUtil {
    
    // Slide + fade in each item from bottom as user scrolls
    public static void setItemAnimation(RecyclerView recyclerView) {
        recyclerView.setItemAnimator(new DefaultItemAnimator());
        
        // Custom layout animation controller
        LayoutAnimationController controller = AnimationUtils.loadLayoutAnimation(
            recyclerView.getContext(), R.anim.layout_animation_fall_down);
        recyclerView.setLayoutAnimation(controller);
    }
}
```

```xml
<!-- res/anim/item_fall_down.xml -->
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300">
    <translate
        android:fromYDelta="-20%"
        android:toYDelta="0%"
        android:interpolator="@android:anim/decelerate_interpolator" />
    <alpha
        android:fromAlpha="0"
        android:toAlpha="1" />
</set>

<!-- res/anim/layout_animation_fall_down.xml -->
<layoutAnimation
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:animation="@anim/item_fall_down"
    android:animationOrder="normal"
    android:delay="15%" />
```

---

## 44.7 Lottie Animations

```groovy
// build.gradle
implementation 'com.airbnb.android:lottie:6.1.0'
```

```xml
<!-- activity_splash.xml -->
<com.airbnb.lottie.LottieAnimationView
    android:id="@+id/lottieView"
    android:layout_width="200dp"
    android:layout_height="200dp"
    app:lottie_fileName="loading.json"
    app:lottie_autoPlay="true"
    app:lottie_loop="true" />
```

```java
// LottieActivity.java
LottieAnimationView lottieView = findViewById(R.id.lottieView);

// Play once, then navigate
lottieView.addAnimatorListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        startActivity(new Intent(SplashActivity.this, MainActivity.class));
        finish();
    }
});
lottieView.playAnimation();

// Load from URL
lottieView.setAnimationFromUrl("https://assets.lottiefiles.com/animation.json");
lottieView.playAnimation();

// Control speed
lottieView.setSpeed(1.5f);

// Progress
lottieView.setProgress(0.5f);  // jump to middle
```

---

## 44.8 Ripple Effect

```xml
<!-- button_ripple.xml (drawable) -->
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?attr/colorControlHighlight">
    <item>
        <shape android:shape="rectangle">
            <corners android:radius="8dp"/>
            <solid android:color="@color/colorPrimary"/>
        </shape>
    </item>
</ripple>

<!-- Usage in Button: -->
<!-- android:background="@drawable/button_ripple" -->
```

---

## 44.9 สรุป Part 44

ในบทนี้คุณได้เรียนรู้:

✅ ObjectAnimator (alpha, translation, scale, rotation)  
✅ ValueAnimator (counter, color animation)  
✅ AnimatorSet (sequence, together)  
✅ Shake, bounce effects  
✅ Shared element transition between Activities  
✅ RecyclerView item animations  
✅ Lottie animation integration  
✅ Ripple effect  

---

*[← Part 43: Custom Views](./part-43-android-custom-views.md) | [Part 45: Content Providers →](./part-45-android-content-providers.md)*
