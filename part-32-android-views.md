# Part 32: Android Views และ Layouts
## หลักสูตร Java & Android Development - ระดับ Beginner Android

---

## 32.1 View Hierarchy

ทุก UI element ใน Android เป็น `View` หรือ `ViewGroup`

```
ViewGroup (layout container)
├── View (widget)
├── View (widget)
└── ViewGroup (nested)
    ├── View
    └── View
```

---

## 32.2 LinearLayout

จัด elements ในแนวตั้ง (vertical) หรือแนวนอน (horizontal)

```xml
<!-- res/layout/activity_linear.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Vertical stack -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ชื่อ-นามสกุล"
        android:textSize="16sp"
        android:textStyle="bold" />

    <EditText
        android:id="@+id/etFullName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="กรอกชื่อ-นามสกุล"
        android:inputType="textPersonName"
        android:layout_marginBottom="16dp" />

    <!-- Horizontal row -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginBottom="16dp">

        <Button
            android:id="@+id/btnSave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="บันทึก"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/btnCancel"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="ยกเลิก" />
    </LinearLayout>

    <!-- Weight distribution -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <View
            android:layout_width="0dp"
            android:layout_height="4dp"
            android:layout_weight="3"
            android:background="#4CAF50" />

        <View
            android:layout_width="0dp"
            android:layout_height="4dp"
            android:layout_weight="1"
            android:background="#F44336" />
    </LinearLayout>

</LinearLayout>
```

---

## 32.3 ConstraintLayout

จัด elements โดยกำหนด constraint กับ parent หรือ sibling views  
เป็น layout ที่แนะนำสำหรับ performance

```xml
<!-- res/layout/activity_constraint.xml -->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <!-- Center in parent -->
    <ImageView
        android:id="@+id/ivLogo"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="32dp" />

    <!-- Below the image -->
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Login"
        android:textSize="28sp"
        android:textStyle="bold"
        app:layout_constraintTop_toBottomOf="@id/ivLogo"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp" />

    <!-- Email input -->
    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilEmail"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="อีเมล"
        app:layout_constraintTop_toBottomOf="@id/tvTitle"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="24dp">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etEmail"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textEmailAddress" />
    </com.google.android.material.textfield.TextInputLayout>

    <!-- Password input -->
    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilPassword"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="รหัสผ่าน"
        app:passwordToggleEnabled="true"
        app:layout_constraintTop_toBottomOf="@id/tilEmail"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="8dp">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etPassword"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword" />
    </com.google.android.material.textfield.TextInputLayout>

    <!-- Login button -->
    <Button
        android:id="@+id/btnLogin"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="เข้าสู่ระบบ"
        app:layout_constraintTop_toBottomOf="@id/tilPassword"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="24dp" />

    <!-- Register link - at bottom -->
    <TextView
        android:id="@+id/tvRegister"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="ยังไม่มีบัญชี? สมัครสมาชิก"
        android:textColor="@color/purple_500"
        android:padding="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginBottom="16dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

---

## 32.4 Common Widgets

```xml
<!-- res/layout/activity_widgets.xml -->
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- TextView -->
        <TextView
            android:id="@+id/tvLabel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="TextView Example"
            android:textSize="18sp"
            android:textColor="#333333"
            android:textStyle="bold|italic"
            android:layout_marginBottom="8dp" />

        <!-- EditText variants -->
        <EditText
            android:id="@+id/etText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="ข้อความทั่วไป"
            android:inputType="text"
            android:layout_marginBottom="8dp" />

        <EditText
            android:id="@+id/etNumber"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="ตัวเลข"
            android:inputType="number"
            android:layout_marginBottom="8dp" />

        <EditText
            android:id="@+id/etMultiline"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="ข้อความหลายบรรทัด"
            android:inputType="textMultiLine"
            android:minLines="3"
            android:maxLines="5"
            android:gravity="top"
            android:layout_marginBottom="8dp" />

        <!-- Buttons -->
        <Button
            android:id="@+id/btnNormal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ปุ่มปกติ"
            android:layout_marginBottom="8dp" />

        <com.google.android.material.button.MaterialButton
            android:id="@+id/btnMaterial"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Material Button"
            android:layout_marginBottom="8dp" />

        <!-- ImageView -->
        <ImageView
            android:id="@+id/ivImage"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:src="@mipmap/ic_launcher"
            android:scaleType="centerCrop"
            android:layout_marginBottom="8dp" />

        <!-- CheckBox -->
        <CheckBox
            android:id="@+id/cbAgree"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ยอมรับเงื่อนไข"
            android:layout_marginBottom="8dp" />

        <!-- RadioGroup -->
        <RadioGroup
            android:id="@+id/rgGender"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="8dp">

            <RadioButton
                android:id="@+id/rbMale"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="ชาย"
                android:checked="true" />

            <RadioButton
                android:id="@+id/rbFemale"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="หญิง" />
        </RadioGroup>

        <!-- Switch -->
        <Switch
            android:id="@+id/swNotification"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="รับการแจ้งเตือน"
            android:layout_marginBottom="8dp" />

        <!-- SeekBar -->
        <SeekBar
            android:id="@+id/sbVolume"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:max="100"
            android:progress="50"
            android:layout_marginBottom="8dp" />

        <!-- Spinner (dropdown) -->
        <Spinner
            android:id="@+id/spinnerCity"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="8dp" />

        <!-- ProgressBar -->
        <ProgressBar
            android:id="@+id/pbLoading"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:visibility="gone" />

        <ProgressBar
            android:id="@+id/pbProgress"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:max="100"
            android:progress="60" />

    </LinearLayout>
</ScrollView>
```

---

## 32.5 Widget Code (Java)

```java
package com.example.myfirstapp;

import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

public class WidgetsActivity extends AppCompatActivity {

    private EditText etName;
    private Button btnSubmit;
    private CheckBox cbNewsletter;
    private RadioGroup rgGender;
    private Switch swPremium;
    private SeekBar sbAge;
    private Spinner spinnerCity;
    private TextView tvResult;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_widgets);

        // Initialize views
        etName = findViewById(R.id.etText);
        btnSubmit = findViewById(R.id.btnNormal);
        cbNewsletter = findViewById(R.id.cbAgree);
        rgGender = findViewById(R.id.rgGender);
        swPremium = findViewById(R.id.swNotification);
        sbAge = findViewById(R.id.sbVolume);
        spinnerCity = findViewById(R.id.spinnerCity);
        tvResult = findViewById(R.id.tvLabel);

        // Setup Spinner
        String[] cities = {"กรุงเทพ", "เชียงใหม่", "ภูเก็ต", "ขอนแก่น", "ชลบุรี"};
        ArrayAdapter<String> adapter = new ArrayAdapter<>(
            this, android.R.layout.simple_spinner_item, cities);
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        spinnerCity.setAdapter(adapter);

        // SeekBar listener
        sbAge.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                // progress changed
            }
            @Override public void onStartTrackingTouch(SeekBar seekBar) {}
            @Override public void onStopTrackingTouch(SeekBar seekBar) {}
        });

        // Submit button
        btnSubmit.setOnClickListener(v -> {
            String name = etName.getText().toString().trim();
            if (name.isEmpty()) {
                Toast.makeText(this, "กรุณาใส่ชื่อ", Toast.LENGTH_SHORT).show();
                return;
            }

            int genderId = rgGender.getCheckedRadioButtonId();
            RadioButton rbGender = findViewById(genderId);
            String gender = rbGender != null ? rbGender.getText().toString() : "ไม่ระบุ";

            String city = spinnerCity.getSelectedItem().toString();
            boolean newsletter = cbNewsletter.isChecked();
            boolean premium = swPremium.isChecked();
            int age = sbAge.getProgress();

            String result = String.format(
                "ชื่อ: %s\nเพศ: %s\nเมือง: %s\nอายุ: %d\nรับข่าวสาร: %s\nPremium: %s",
                name, gender, city, age,
                newsletter ? "ใช่" : "ไม่",
                premium ? "ใช่" : "ไม่");

            tvResult.setText(result);
        });
    }
}
```

---

## 32.6 Dimension Units

```
px  - pixels (ไม่แนะนำ - ต่าง device ต่างขนาด)
dp  - density-independent pixels (แนะนำสำหรับ size/margin)
sp  - scale-independent pixels (แนะนำสำหรับ font size - respects user font settings)
pt  - points (1/72 inch)

แนวทาง:
- Layout sizes: ใช้ dp
- Font sizes: ใช้ sp
- Never use px for layout

ขนาดที่แนะนำ:
- Padding/Margin: 4dp, 8dp, 16dp, 24dp, 32dp
- Button height: 48dp (minimum touch target)
- Icon size: 24dp, 48dp
- Body text: 14sp-16sp
- Title: 20sp-24sp
- Headline: 24sp-32sp
```

---

## 32.7 สรุป Part 32

ในบทนี้คุณได้เรียนรู้:

✅ View hierarchy  
✅ LinearLayout (vertical/horizontal, weight)  
✅ ConstraintLayout  
✅ Common widgets (TextView, EditText, Button, ImageView)  
✅ CheckBox, RadioGroup, Switch  
✅ SeekBar, Spinner, ProgressBar  
✅ ArrayAdapter for Spinner  
✅ Dimension units (dp, sp)  

---

*[← Part 31: Android Intro](./part-31-android-intro.md) | [Part 33: Intents & Navigation →](./part-33-android-intents.md)*
