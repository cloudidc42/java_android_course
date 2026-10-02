# Part 58: Accessibility (การเข้าถึง) สำหรับ Android
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 58.1 Accessibility คืออะไรและทำไมสำคัญ

Accessibility คือการทำให้แอปใช้งานได้โดยผู้ทุพพลภาพ:
- ผู้พิการทางสายตา → TalkBack (screen reader)
- ผู้พิการทางการได้ยิน → closed captions
- ผู้พิการทางการเคลื่อนไหว → Switch Access, voice control

**ผลดี:**
- ครอบคลุมผู้ใช้มากขึ้น (15% ของประชากรโลกมีความพิการ)
- Play Store ให้คะแนนสูงขึ้น
- กฎหมายบางประเทศบังคับ

---

## 58.2 Content Descriptions

```xml
<!-- BAD: ImageButton without description -->
<ImageButton
    android:id="@+id/btnFavorite"
    android:src="@drawable/ic_heart" />

<!-- GOOD: With contentDescription -->
<ImageButton
    android:id="@+id/btnFavorite"
    android:src="@drawable/ic_heart"
    android:contentDescription="เพิ่มในรายการโปรด" />

<!-- Decorative image (no description needed) -->
<ImageView
    android:src="@drawable/decorative_bg"
    android:importantForAccessibility="no" />

<!-- Dynamic content description in code -->
```

```java
// Dynamic contentDescription
button.setContentDescription(isLiked ?
    "เอาออกจากรายการโปรด" : "เพิ่มในรายการโปรด");

// Announce changes to TalkBack
imageView.announceForAccessibility("โหลดรูปภาพเสร็จแล้ว");

// Send accessibility event
view.sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_FOCUSED);
```

---

## 58.3 Semantic Structure

```xml
<!-- Group related items with contentDescription -->
<LinearLayout
    android:orientation="horizontal"
    android:contentDescription="ราคา 250 บาท ลด 20%"
    android:focusable="true">
    
    <TextView
        android:text="250 บาท"
        android:importantForAccessibility="no" />
    <TextView
        android:text="-20%"
        android:importantForAccessibility="no" />
</LinearLayout>

<!-- Heading for sections -->
<TextView
    android:text="รายการสินค้า"
    android:accessibilityHeading="true" />

<!-- Live region - announce changes automatically -->
<TextView
    android:id="@+id/tvStatus"
    android:accessibilityLiveRegion="polite" />
    <!-- "polite" = announce when idle, "assertive" = announce immediately -->
```

```java
// Accessibility node info for custom views
@Override
public void onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info) {
    super.onInitializeAccessibilityNodeInfo(info);
    
    // Mark as button
    info.setClassName(Button.class.getName());
    
    // State
    info.setChecked(isChecked);
    info.setCheckable(true);
    
    // Custom action
    info.addAction(new AccessibilityNodeInfo.AccessibilityAction(
        AccessibilityNodeInfo.ACTION_CLICK, "สลับสถานะ"));
}
```

---

## 58.4 Touch Target Size

```xml
<!-- Minimum touch target: 48dp x 48dp -->
<!-- BAD: 24dp icon button -->
<ImageButton
    android:layout_width="24dp"
    android:layout_height="24dp"
    android:src="@drawable/ic_close" />

<!-- GOOD: 48dp minimum -->
<ImageButton
    android:layout_width="48dp"
    android:layout_height="48dp"
    android:padding="12dp"
    android:src="@drawable/ic_close" />

<!-- Or use padding to expand touch area -->
<ImageButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:padding="12dp"
    android:src="@drawable/ic_close" />
```

---

## 58.5 Color Contrast

```xml
<!-- WCAG AA: minimum contrast ratio 4.5:1 for normal text, 3:1 for large text -->

<!-- BAD: Low contrast -->
<TextView
    android:textColor="#CCCCCC"
    android:background="#FFFFFF" />
<!-- Contrast ratio: 1.6:1 - FAIL -->

<!-- GOOD: High contrast -->
<TextView
    android:textColor="#212121"
    android:background="#FFFFFF" />
<!-- Contrast ratio: 16.1:1 - PASS -->

<!-- Support dark mode with proper contrast in both -->
```

```java
// Check contrast at runtime (for dynamic colors)
public static boolean isContrastSufficient(int foreground, int background) {
    double fgLuminance = getRelativeLuminance(foreground);
    double bgLuminance = getRelativeLuminance(background);
    
    double lighter = Math.max(fgLuminance, bgLuminance);
    double darker  = Math.min(fgLuminance, bgLuminance);
    
    double ratio = (lighter + 0.05) / (darker + 0.05);
    return ratio >= 4.5;  // WCAG AA
}

private static double getRelativeLuminance(int color) {
    double r = linearize(Color.red(color) / 255.0);
    double g = linearize(Color.green(color) / 255.0);
    double b = linearize(Color.blue(color) / 255.0);
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

private static double linearize(double c) {
    return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
}
```

---

## 58.6 Keyboard and Switch Access

```xml
<!-- Make custom views focusable -->
<com.example.CustomView
    android:focusable="true"
    android:focusableInTouchMode="true"
    android:nextFocusDown="@id/nextView" />

<!-- Define focus order -->
<EditText
    android:id="@+id/etName"
    android:nextFocusForward="@id/etEmail" />

<EditText
    android:id="@+id/etEmail"
    android:nextFocusForward="@id/etPhone" />

<EditText
    android:id="@+id/etPhone"
    android:nextFocusForward="@id/btnSubmit" />
```

```java
// Handle keyboard navigation in custom view
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_ENTER || keyCode == KeyEvent.KEYCODE_SPACE) {
        performClick();
        return true;
    }
    return super.onKeyDown(keyCode, event);
}

@Override
public boolean performClick() {
    super.performClick();
    // Handle click action
    return true;
}
```

---

## 58.7 Testing Accessibility

```java
// Espresso Accessibility Testing
// build.gradle: androidTestImplementation 'androidx.test.espresso:espresso-accessibility:3.5.1'

@Test
public void testAccessibility() {
    // Enable TalkBack checks
    AccessibilityChecks.enable().setRunChecksFromRootView(true);
    
    onView(withId(R.id.btnSave)).perform(click());
    // Will fail if button has no content description, too small, etc.
}

// Manual testing checklist:
// 1. Enable TalkBack (Settings → Accessibility → TalkBack)
// 2. Navigate using swipe gestures
// 3. Verify all elements are announced correctly
// 4. Test with font size increased
// 5. Test with display size increased
// 6. Test in landscape mode
// 7. Test with color correction (protanopia, deuteranopia)
```

---

## 58.8 Text Scaling Support

```java
// Always use sp (not px/dp) for text sizes - respects user's font size preference

// Check current font scale
float fontScale = getResources().getConfiguration().fontScale;
// 1.0 = default, 1.3 = large, 1.5 = huge

// Use Autosizing TextView
// In XML:
// app:autoSizeTextType="uniform"
// app:autoSizeMinTextSize="12sp"
// app:autoSizeMaxTextSize="20sp"

// Avoid fixed heights for text containers - use wrap_content
// BAD: fixed height
// <TextView android:layout_height="40dp" />

// GOOD: dynamic height
// <TextView android:layout_height="wrap_content" android:minHeight="48dp" />
```

---

## 58.9 สรุป Part 58

ในบทนี้คุณได้เรียนรู้:

✅ Accessibility importance  
✅ contentDescription สำหรับ images, buttons  
✅ importantForAccessibility  
✅ AccessibilityLiveRegion  
✅ Minimum touch target size (48dp)  
✅ Color contrast ratio (WCAG AA)  
✅ Keyboard and Switch Access navigation  
✅ Accessibility testing with Espresso  
✅ Text scaling (sp units, AutoSizing)  

---

*[← Part 57: Advanced RecyclerView](./part-57-android-advanced-recyclerview.md) | [Part 59: Internationalization →](./part-59-android-i18n.md)*
