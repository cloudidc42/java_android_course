# Part 33: Intents และ Navigation
## หลักสูตร Java & Android Development - ระดับ Beginner Android

---

## 33.1 Intent คืออะไร

Intent คือ "ข้อความ" สำหรับสื่อสารระหว่าง components ใน Android

**ประเภทของ Intent:**
- **Explicit Intent**: ระบุ Activity ปลายทางชัดเจน
- **Implicit Intent**: ระบุ action ที่ต้องการ (OS เลือก app)

---

## 33.2 Explicit Intent - เปิด Activity

```java
// MainActivity.java
package com.example.myapp;

import android.content.Intent;
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button btnOpenDetail = findViewById(R.id.btnOpenDetail);
        btnOpenDetail.setOnClickListener(v -> {
            // Simple intent to open another activity
            Intent intent = new Intent(this, DetailActivity.class);
            startActivity(intent);
        });

        Button btnOpenWithData = findViewById(R.id.btnOpenWithData);
        btnOpenWithData.setOnClickListener(v -> {
            // Intent with extra data
            Intent intent = new Intent(this, ProfileActivity.class);
            intent.putExtra("userId", 42);
            intent.putExtra("userName", "Alice");
            intent.putExtra("isAdmin", true);
            startActivity(intent);
        });
    }
}
```

```java
// DetailActivity.java
public class DetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);

        // Enable back button in toolbar
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle("รายละเอียด");
        }
    }

    @Override
    public boolean onSupportNavigateUp() {
        onBackPressed();
        return true;
    }
}
```

```java
// ProfileActivity.java - รับข้อมูลจาก Intent
public class ProfileActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_profile);

        // รับข้อมูลจาก Intent
        Intent intent = getIntent();
        int userId = intent.getIntExtra("userId", -1);
        String userName = intent.getStringExtra("userName");
        boolean isAdmin = intent.getBooleanExtra("isAdmin", false);

        TextView tvProfile = findViewById(R.id.tvProfile);
        tvProfile.setText(String.format(
            "User ID: %d\nชื่อ: %s\nAdmin: %s",
            userId, userName, isAdmin ? "ใช่" : "ไม่ใช่"
        ));
    }
}
```

---

## 33.3 startActivityForResult (รับผลลัพธ์)

```java
// MainActivity.java - เรียก Activity และรับผลลัพธ์กลับ
public class MainActivity extends AppCompatActivity {

    private ActivityResultLauncher<Intent> editLauncher;
    private TextView tvName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvName = findViewById(R.id.tvName);

        // Register result handler (modern way)
        editLauncher = registerForActivityResult(
            new ActivityResultContracts.StartActivityForResult(),
            result -> {
                if (result.getResultCode() == RESULT_OK && result.getData() != null) {
                    String newName = result.getData().getStringExtra("updatedName");
                    tvName.setText(newName);
                    Toast.makeText(this, "อัปเดตแล้ว!", Toast.LENGTH_SHORT).show();
                }
            }
        );

        Button btnEdit = findViewById(R.id.btnEdit);
        btnEdit.setOnClickListener(v -> {
            Intent intent = new Intent(this, EditActivity.class);
            intent.putExtra("currentName", tvName.getText().toString());
            editLauncher.launch(intent);
        });
    }
}
```

```java
// EditActivity.java - ส่งผลลัพธ์กลับ
public class EditActivity extends AppCompatActivity {

    private EditText etName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_edit);

        etName = findViewById(R.id.etName);

        // Pre-fill current value
        String currentName = getIntent().getStringExtra("currentName");
        etName.setText(currentName);

        Button btnSave = findViewById(R.id.btnSave);
        btnSave.setOnClickListener(v -> {
            String newName = etName.getText().toString().trim();
            if (!newName.isEmpty()) {
                Intent result = new Intent();
                result.putExtra("updatedName", newName);
                setResult(RESULT_OK, result);
                finish();  // close this activity and return
            } else {
                Toast.makeText(this, "กรุณาใส่ชื่อ", Toast.LENGTH_SHORT).show();
            }
        });

        Button btnCancel = findViewById(R.id.btnCancel);
        btnCancel.setOnClickListener(v -> {
            setResult(RESULT_CANCELED);
            finish();
        });
    }
}
```

---

## 33.4 Implicit Intent

```java
public class ImplicitIntentDemo extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demo);

        // Open URL in browser
        Button btnBrowser = findViewById(R.id.btnBrowser);
        btnBrowser.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_VIEW,
                Uri.parse("https://www.google.com"));
            startActivity(intent);
        });

        // Make phone call
        Button btnCall = findViewById(R.id.btnCall);
        btnCall.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_DIAL,
                Uri.parse("tel:+6612345678"));
            startActivity(intent);
        });

        // Send email
        Button btnEmail = findViewById(R.id.btnEmail);
        btnEmail.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_SENDTO);
            intent.setData(Uri.parse("mailto:test@example.com"));
            intent.putExtra(Intent.EXTRA_SUBJECT, "หัวเรื่อง");
            intent.putExtra(Intent.EXTRA_TEXT, "เนื้อหาอีเมล");
            if (intent.resolveActivity(getPackageManager()) != null) {
                startActivity(intent);
            }
        });

        // Share text
        Button btnShare = findViewById(R.id.btnShare);
        btnShare.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_SEND);
            intent.setType("text/plain");
            intent.putExtra(Intent.EXTRA_TEXT, "แชร์ข้อความนี้!");
            startActivity(Intent.createChooser(intent, "แชร์ผ่าน..."));
        });

        // Open map
        Button btnMap = findViewById(R.id.btnMap);
        btnMap.setOnClickListener(v -> {
            Uri mapUri = Uri.parse("geo:13.7563,100.5018?q=กรุงเทพ");
            Intent intent = new Intent(Intent.ACTION_VIEW, mapUri);
            intent.setPackage("com.google.android.apps.maps");
            startActivity(intent);
        });

        // Pick image from gallery
        Button btnGallery = findViewById(R.id.btnGallery);
        btnGallery.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_PICK);
            intent.setType("image/*");
            pickImageLauncher.launch(intent);
        });
    }

    private final ActivityResultLauncher<Intent> pickImageLauncher =
        registerForActivityResult(
            new ActivityResultContracts.StartActivityForResult(),
            result -> {
                if (result.getResultCode() == RESULT_OK && result.getData() != null) {
                    Uri imageUri = result.getData().getData();
                    ImageView ivSelected = findViewById(R.id.ivSelected);
                    ivSelected.setImageURI(imageUri);
                }
            }
        );
}
```

---

## 33.5 Pass Complex Data with Serializable/Parcelable

```java
// Model class - implement Serializable (simple) or Parcelable (faster)
import android.os.Parcel;
import android.os.Parcelable;

public class Product implements Parcelable {
    private int id;
    private String name;
    private double price;
    private String category;

    public Product(int id, String name, double price, String category) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.category = category;
    }

    // Parcelable implementation
    protected Product(Parcel in) {
        id = in.readInt();
        name = in.readString();
        price = in.readDouble();
        category = in.readString();
    }

    public static final Creator<Product> CREATOR = new Creator<Product>() {
        @Override
        public Product createFromParcel(Parcel in) { return new Product(in); }
        @Override
        public Product[] newArray(int size) { return new Product[size]; }
    };

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(id);
        dest.writeString(name);
        dest.writeDouble(price);
        dest.writeString(category);
    }

    @Override
    public int describeContents() { return 0; }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getCategory() { return category; }
}

// Send
Intent intent = new Intent(this, ProductDetailActivity.class);
Product product = new Product(1, "iPhone 15", 35000.0, "Electronics");
intent.putExtra("product", product);
startActivity(intent);

// Receive
Product received = getIntent().getParcelableExtra("product");
```

---

## 33.6 AndroidManifest.xml - เพิ่ม Activities

```xml
<manifest ...>
    <application ...>
        
        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        
        <!-- ต้องประกาศทุก Activity ใน Manifest -->
        <activity android:name=".DetailActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity"/>
        
        <activity android:name=".ProfileActivity"
            android:exported="false"/>
        
        <activity android:name=".EditActivity"
            android:exported="false"/>
        
    </application>
</manifest>
```

---

## 33.7 สรุป Part 33

ในบทนี้คุณได้เรียนรู้:

✅ Explicit Intent (เปิด Activity ชัดเจน)  
✅ Intent extras (ส่ง int, String, boolean)  
✅ ActivityResultLauncher (รับผลลัพธ์)  
✅ Implicit Intent (browser, phone, email, share)  
✅ Parcelable (ส่ง object ระหว่าง Activity)  
✅ Back navigation  
✅ Manifest activity registration  

---

*[← Part 32: Android Views](./part-32-android-views.md) | [Part 34: Android RecyclerView →](./part-34-android-recyclerview.md)*
