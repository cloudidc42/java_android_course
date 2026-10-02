# Part 43: Custom Views และ Canvas Drawing
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 43.1 Custom View คืออะไร

Custom View คือ View ที่เราสร้างเองโดย extend View class  
ใช้เมื่อ standard widgets ไม่เพียงพอสำหรับ UI ที่ต้องการ

**เมื่อไหรควรสร้าง Custom View:**
- ต้องการ custom drawing (charts, gauges, shapes)
- Combine หลาย views ให้เป็น reusable component
- ต้องการ custom touch behavior

---

## 43.2 Custom View พื้นฐาน - Circle View

```java
// view/CircleView.java
package com.example.myapp.view;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.*;
import android.util.AttributeSet;
import android.view.View;
import com.example.myapp.R;

public class CircleView extends View {
    
    private Paint circlePaint;
    private Paint textPaint;
    private float radius;
    private int circleColor;
    private String label;
    
    // Constructors - required for XML inflation
    public CircleView(Context context) {
        super(context);
        init(null);
    }
    
    public CircleView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(attrs);
    }
    
    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(attrs);
    }
    
    private void init(AttributeSet attrs) {
        // Read custom attributes from XML
        if (attrs != null) {
            TypedArray ta = getContext().obtainStyledAttributes(attrs, R.styleable.CircleView);
            circleColor = ta.getColor(R.styleable.CircleView_circleColor, Color.BLUE);
            label = ta.getString(R.styleable.CircleView_label);
            if (label == null) label = "";
            ta.recycle();
        } else {
            circleColor = Color.BLUE;
            label = "";
        }
        
        // Circle paint
        circlePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        circlePaint.setColor(circleColor);
        circlePaint.setStyle(Paint.Style.FILL);
        
        // Text paint
        textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setColor(Color.WHITE);
        textPaint.setTextSize(40f);
        textPaint.setTextAlign(Paint.Align.CENTER);
    }
    
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // Make view square
        int width  = resolveSize(200, widthMeasureSpec);
        int height = resolveSize(200, heightMeasureSpec);
        int size   = Math.min(width, height);
        setMeasuredDimension(size, size);
        radius = size / 2f;
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        float cx = getWidth() / 2f;
        float cy = getHeight() / 2f;
        
        // Draw circle
        canvas.drawCircle(cx, cy, radius - 4, circlePaint);
        
        // Draw label
        if (!label.isEmpty()) {
            // Center text vertically
            float textY = cy - (textPaint.descent() + textPaint.ascent()) / 2;
            canvas.drawText(label, cx, textY, textPaint);
        }
    }
    
    // Public setters (trigger redraw)
    public void setCircleColor(int color) {
        circlePaint.setColor(color);
        invalidate();  // request redraw
    }
    
    public void setLabel(String text) {
        this.label = text;
        invalidate();
    }
}
```

**attrs.xml:**
```xml
<!-- res/values/attrs.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">
        <attr name="circleColor" format="color"/>
        <attr name="label" format="string"/>
    </declare-styleable>
</resources>
```

**Usage in XML:**
```xml
<com.example.myapp.view.CircleView
    android:layout_width="100dp"
    android:layout_height="100dp"
    app:circleColor="#FF5722"
    app:label="A" />
```

---

## 43.3 Progress Bar แบบ Custom (Arc)

```java
// view/ArcProgressView.java
public class ArcProgressView extends View {
    
    private Paint bgPaint, progressPaint, textPaint;
    private RectF arcRect;
    private float progress = 0f;  // 0 - 100
    private int progressColor = Color.parseColor("#4CAF50");
    private int bgColor = Color.parseColor("#E0E0E0");
    
    public ArcProgressView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }
    
    private void init() {
        bgPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        bgPaint.setStyle(Paint.Style.STROKE);
        bgPaint.setStrokeWidth(20f);
        bgPaint.setColor(bgColor);
        bgPaint.setStrokeCap(Paint.Cap.ROUND);
        
        progressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        progressPaint.setStyle(Paint.Style.STROKE);
        progressPaint.setStrokeWidth(20f);
        progressPaint.setColor(progressColor);
        progressPaint.setStrokeCap(Paint.Cap.ROUND);
        
        textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setColor(Color.BLACK);
        textPaint.setTextSize(48f);
        textPaint.setTextAlign(Paint.Align.CENTER);
        textPaint.setTypeface(Typeface.DEFAULT_BOLD);
    }
    
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        float padding = 30f;
        arcRect = new RectF(padding, padding, w - padding, h - padding);
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        // Background arc (full 270 degrees, starting from -225)
        canvas.drawArc(arcRect, -225, 270, false, bgPaint);
        
        // Progress arc
        float sweep = (progress / 100f) * 270;
        canvas.drawArc(arcRect, -225, sweep, false, progressPaint);
        
        // Percentage text
        float cx = getWidth() / 2f;
        float cy = getHeight() / 2f - (textPaint.descent() + textPaint.ascent()) / 2;
        canvas.drawText((int) progress + "%", cx, cy, textPaint);
    }
    
    public void setProgress(float progress) {
        this.progress = Math.max(0, Math.min(100, progress));
        invalidate();
    }
    
    // Animate progress
    public void animateToProgress(float targetProgress) {
        android.animation.ValueAnimator animator = 
            android.animation.ValueAnimator.ofFloat(this.progress, targetProgress);
        animator.setDuration(1000);
        animator.setInterpolator(new android.view.animation.DecelerateInterpolator());
        animator.addUpdateListener(anim -> {
            setProgress((Float) anim.getAnimatedValue());
        });
        animator.start();
    }
}
```

---

## 43.4 Line Chart Custom View

```java
// view/LineChartView.java
public class LineChartView extends View {
    
    private List<Float> data = new ArrayList<>();
    private Paint linePaint, pointPaint, gridPaint, textPaint;
    private Path linePath;
    private float paddingLeft = 60f, paddingTop = 20f;
    private float paddingRight = 20f, paddingBottom = 40f;
    
    public LineChartView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }
    
    private void init() {
        linePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        linePaint.setColor(Color.parseColor("#2196F3"));
        linePaint.setStrokeWidth(4f);
        linePaint.setStyle(Paint.Style.STROKE);
        
        pointPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        pointPaint.setColor(Color.parseColor("#2196F3"));
        pointPaint.setStyle(Paint.Style.FILL);
        
        gridPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        gridPaint.setColor(Color.parseColor("#E0E0E0"));
        gridPaint.setStrokeWidth(1f);
        gridPaint.setStyle(Paint.Style.STROKE);
        
        textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setColor(Color.GRAY);
        textPaint.setTextSize(28f);
        
        linePath = new Path();
    }
    
    public void setData(List<Float> data) {
        this.data = data;
        invalidate();
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        if (data == null || data.size() < 2) return;
        
        float chartW = getWidth() - paddingLeft - paddingRight;
        float chartH = getHeight() - paddingTop - paddingBottom;
        
        float maxVal = Collections.max(data);
        float minVal = Collections.min(data);
        float range  = maxVal - minVal;
        if (range == 0) range = 1;
        
        // Draw grid lines
        int gridLines = 4;
        for (int i = 0; i <= gridLines; i++) {
            float y = paddingTop + chartH * i / gridLines;
            canvas.drawLine(paddingLeft, y, paddingLeft + chartW, y, gridPaint);
            
            float value = maxVal - (range * i / gridLines);
            canvas.drawText(String.format("%.0f", value), 5, y + 10, textPaint);
        }
        
        // Draw line
        linePath.reset();
        for (int i = 0; i < data.size(); i++) {
            float x = paddingLeft + chartW * i / (data.size() - 1);
            float y = paddingTop + chartH * (1 - (data.get(i) - minVal) / range);
            
            if (i == 0) linePath.moveTo(x, y);
            else linePath.lineTo(x, y);
            
            // Draw point
            canvas.drawCircle(x, y, 8f, pointPaint);
        }
        canvas.drawPath(linePath, linePaint);
    }
}
```

---

## 43.5 Custom ViewGroup

```java
// view/BadgedImageView.java (combines ImageView + badge TextView)
public class BadgedImageView extends FrameLayout {
    
    private ImageView imageView;
    private TextView badgeView;
    
    public BadgedImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
        inflate(context, R.layout.view_badged_image, this);
        
        imageView = findViewById(R.id.imageView);
        badgeView = findViewById(R.id.badge);
    }
    
    public void setImage(int resId) {
        imageView.setImageResource(resId);
    }
    
    public void setBadge(int count) {
        if (count <= 0) {
            badgeView.setVisibility(GONE);
        } else {
            badgeView.setVisibility(VISIBLE);
            badgeView.setText(count > 99 ? "99+" : String.valueOf(count));
        }
    }
}
```

```xml
<!-- res/layout/view_badged_image.xml -->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:scaleType="centerCrop" />
    
    <TextView
        android:id="@+id/badge"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="top|end"
        android:background="@drawable/bg_badge"
        android:textColor="@android:color/white"
        android:textSize="10sp"
        android:paddingHorizontal="4dp"
        android:minWidth="18dp"
        android:gravity="center"
        android:visibility="gone" />
    
</merge>
```

---

## 43.6 Touch Handling

```java
// view/DrawingView.java (finger painting)
public class DrawingView extends View {
    
    private Paint paint;
    private Path currentPath;
    private List<Path> paths = new ArrayList<>();
    private List<Paint> paints = new ArrayList<>();
    private int currentColor = Color.BLACK;
    private float strokeWidth = 8f;
    
    public DrawingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }
    
    private void initPaint() {
        paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        paint.setColor(currentColor);
        paint.setStrokeWidth(strokeWidth);
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeJoin(Paint.Join.ROUND);
        paint.setStrokeCap(Paint.Cap.ROUND);
    }
    
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                currentPath = new Path();
                currentPath.moveTo(x, y);
                paths.add(currentPath);
                // Save copy of current paint
                Paint p = new Paint(paint);
                paints.add(p);
                break;
                
            case MotionEvent.ACTION_MOVE:
                if (currentPath != null) currentPath.lineTo(x, y);
                break;
                
            case MotionEvent.ACTION_UP:
                currentPath = null;
                break;
        }
        
        invalidate();
        return true;
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawColor(Color.WHITE);
        for (int i = 0; i < paths.size(); i++) {
            canvas.drawPath(paths.get(i), paints.get(i));
        }
    }
    
    public void setColor(int color) {
        currentColor = color;
        paint.setColor(color);
    }
    
    public void setStrokeWidth(float width) {
        strokeWidth = width;
        paint.setStrokeWidth(width);
    }
    
    public void undo() {
        if (!paths.isEmpty()) {
            paths.remove(paths.size() - 1);
            paints.remove(paints.size() - 1);
            invalidate();
        }
    }
    
    public void clear() {
        paths.clear();
        paints.clear();
        invalidate();
    }
    
    public Bitmap getBitmap() {
        Bitmap bitmap = Bitmap.createBitmap(getWidth(), getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        draw(canvas);
        return bitmap;
    }
}
```

---

## 43.7 สรุป Part 43

ในบทนี้คุณได้เรียนรู้:

✅ Create Custom View (extend View)  
✅ Custom attributes (attrs.xml, TypedArray)  
✅ onMeasure, onDraw, invalidate()  
✅ Canvas drawing (circles, arcs, lines, text)  
✅ ArcProgressView with animation  
✅ LineChartView with data  
✅ Custom ViewGroup  
✅ Touch event handling (finger drawing)  

---

*[← Part 42: Android Permissions](./part-42-android-permissions.md) | [Part 44: Android Animations →](./part-44-android-animations.md)*
