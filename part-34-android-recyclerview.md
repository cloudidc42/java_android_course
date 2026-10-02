# Part 34: RecyclerView
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 34.1 RecyclerView คืออะไร

RecyclerView คือ widget สำหรับแสดงข้อมูลเป็น list/grid  
ดีกว่า ListView เพราะ recycles views (performance ดีกว่า)

**Components:**
- `RecyclerView` - container widget ใน layout
- `Adapter` - เชื่อมข้อมูลกับ Views
- `ViewHolder` - ถือ references ของ View  
- `LayoutManager` - จัดวาง items (linear, grid, staggered)

---

## 34.2 Dependencies

```groovy
// build.gradle (app)
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.3.1'
    implementation 'com.google.android.material:material:1.9.0'
    // Image loading
    implementation 'com.github.bumptech.glide:glide:4.15.1'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.15.1'
}
```

---

## 34.3 Model Class

```java
// model/Student.java
package com.example.myapp.model;

public class Student {
    private int id;
    private String name;
    private String major;
    private double gpa;
    private String avatarUrl;

    public Student(int id, String name, String major, double gpa) {
        this.id = id;
        this.name = name;
        this.major = major;
        this.gpa = gpa;
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getMajor() { return major; }
    public double getGpa() { return gpa; }
    public String getAvatarUrl() { return avatarUrl; }
    public void setAvatarUrl(String url) { avatarUrl = url; }
}
```

---

## 34.4 Item Layout

```xml
<!-- res/layout/item_student.xml -->
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp">

        <!-- Avatar -->
        <ImageView
            android:id="@+id/ivAvatar"
            android:layout_width="56dp"
            android:layout_height="56dp"
            android:src="@mipmap/ic_launcher"
            android:scaleType="centerCrop"
            android:layout_marginEnd="12dp" />

        <!-- Info -->
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">

            <TextView
                android:id="@+id/tvName"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#212121" />

            <TextView
                android:id="@+id/tvMajor"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="13sp"
                android:textColor="#757575"
                android:layout_marginTop="2dp" />

            <TextView
                android:id="@+id/tvGpa"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="13sp"
                android:textColor="#4CAF50"
                android:layout_marginTop="2dp" />
        </LinearLayout>

        <!-- GPA badge -->
        <TextView
            android:id="@+id/tvGpaBadge"
            android:layout_width="44dp"
            android:layout_height="44dp"
            android:gravity="center"
            android:textSize="14sp"
            android:textStyle="bold"
            android:textColor="@android:color/white"
            android:background="@drawable/circle_badge" />

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

```xml
<!-- res/drawable/circle_badge.xml -->
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#2196F3"/>
    <size android:width="44dp" android:height="44dp"/>
</shape>
```

---

## 34.5 Adapter

```java
// adapter/StudentAdapter.java
package com.example.myapp.adapter;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.bumptech.glide.Glide;
import com.example.myapp.R;
import com.example.myapp.model.Student;
import java.util.ArrayList;
import java.util.List;

public class StudentAdapter extends RecyclerView.Adapter<StudentAdapter.ViewHolder> {

    // Interface for click events
    public interface OnItemClickListener {
        void onItemClick(Student student, int position);
        void onItemLongClick(Student student, int position);
    }

    private List<Student> students;
    private OnItemClickListener listener;
    private Context context;

    public StudentAdapter(Context context, List<Student> students) {
        this.context = context;
        this.students = new ArrayList<>(students);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }

    // Create new view (called by LayoutManager)
    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
            .inflate(R.layout.item_student, parent, false);
        return new ViewHolder(view);
    }

    // Replace contents of a view (called by LayoutManager)
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Student student = students.get(position);
        holder.bind(student);

        // Click events
        holder.itemView.setOnClickListener(v -> {
            if (listener != null) listener.onItemClick(student, position);
        });
        holder.itemView.setOnLongClickListener(v -> {
            if (listener != null) listener.onItemLongClick(student, position);
            return true;
        });
    }

    @Override
    public int getItemCount() { return students.size(); }

    // --- Data update methods ---

    public void updateList(List<Student> newList) {
        students.clear();
        students.addAll(newList);
        notifyDataSetChanged();  // refresh all items
    }

    public void addItem(Student student) {
        students.add(student);
        notifyItemInserted(students.size() - 1);
    }

    public void removeItem(int position) {
        students.remove(position);
        notifyItemRemoved(position);
        notifyItemRangeChanged(position, students.size());
    }

    public void updateItem(int position, Student student) {
        students.set(position, student);
        notifyItemChanged(position);
    }

    public List<Student> getStudents() { return new ArrayList<>(students); }

    // --- ViewHolder ---
    public static class ViewHolder extends RecyclerView.ViewHolder {
        ImageView ivAvatar;
        TextView tvName, tvMajor, tvGpa, tvGpaBadge;

        ViewHolder(@NonNull View itemView) {
            super(itemView);
            ivAvatar   = itemView.findViewById(R.id.ivAvatar);
            tvName     = itemView.findViewById(R.id.tvName);
            tvMajor    = itemView.findViewById(R.id.tvMajor);
            tvGpa      = itemView.findViewById(R.id.tvGpa);
            tvGpaBadge = itemView.findViewById(R.id.tvGpaBadge);
        }

        void bind(Student student) {
            tvName.setText(student.getName());
            tvMajor.setText(student.getMajor());
            tvGpa.setText(String.format("GPA: %.2f", student.getGpa()));
            tvGpaBadge.setText(String.format("%.1f", student.getGpa()));

            // Load image with Glide
            if (student.getAvatarUrl() != null) {
                Glide.with(itemView.getContext())
                    .load(student.getAvatarUrl())
                    .circleCrop()
                    .placeholder(R.mipmap.ic_launcher)
                    .into(ivAvatar);
            }
        }
    }
}
```

---

## 34.6 Activity with RecyclerView

```java
// StudentListActivity.java
package com.example.myapp;

import android.content.Intent;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import androidx.recyclerview.widget.DividerItemDecoration;
import com.example.myapp.adapter.StudentAdapter;
import com.example.myapp.model.Student;
import java.util.*;

public class StudentListActivity extends AppCompatActivity {

    private RecyclerView recyclerView;
    private StudentAdapter adapter;
    private EditText etSearch;
    private List<Student> allStudents = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_student_list);

        recyclerView = findViewById(R.id.recyclerView);
        etSearch = findViewById(R.id.etSearch);

        // Setup RecyclerView
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(layoutManager);
        recyclerView.addItemDecoration(
            new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));

        // Create adapter
        adapter = new StudentAdapter(this, new ArrayList<>());
        recyclerView.setAdapter(adapter);

        // Setup click listener
        adapter.setOnItemClickListener(new StudentAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(Student student, int position) {
                // Open detail
                Intent intent = new Intent(StudentListActivity.this, StudentDetailActivity.class);
                intent.putExtra("studentId", student.getId());
                intent.putExtra("studentName", student.getName());
                startActivity(intent);
            }

            @Override
            public void onItemLongClick(Student student, int position) {
                // Show delete option
                new androidx.appcompat.app.AlertDialog.Builder(StudentListActivity.this)
                    .setTitle("ลบนักเรียน")
                    .setMessage("ต้องการลบ " + student.getName() + " หรือไม่?")
                    .setPositiveButton("ลบ", (dialog, which) -> {
                        adapter.removeItem(position);
                        Toast.makeText(StudentListActivity.this,
                            "ลบ " + student.getName() + " แล้ว", Toast.LENGTH_SHORT).show();
                    })
                    .setNegativeButton("ยกเลิก", null)
                    .show();
            }
        });

        // Load data
        loadStudents();

        // Search
        etSearch.addTextChangedListener(new TextWatcher() {
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                filterStudents(s.toString());
            }
            @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
            @Override public void afterTextChanged(Editable s) {}
        });
    }

    private void loadStudents() {
        // Sample data
        allStudents = Arrays.asList(
            new Student(1, "สมชาย ใจดี", "วิทยาการคอมพิวเตอร์", 3.85),
            new Student(2, "สมหญิง มานะ", "วิศวกรรมซอฟต์แวร์", 3.72),
            new Student(3, "วิชัย สุขใจ", "เทคโนโลยีสารสนเทศ", 3.60),
            new Student(4, "มาลี รักเรียน", "วิทยาการคอมพิวเตอร์", 3.91),
            new Student(5, "ประยุทธ ขยัน", "วิศวกรรมคอมพิวเตอร์", 3.45),
            new Student(6, "อนุชา เก่งกล้า", "ระบบสารสนเทศ", 3.78),
            new Student(7, "สุภาพร ใฝ่รู้", "วิทยาการคอมพิวเตอร์", 3.88),
            new Student(8, "รัชนี พากเพียร", "เทคโนโลยีสารสนเทศ", 3.55)
        );
        adapter.updateList(allStudents);
    }

    private void filterStudents(String query) {
        if (query.isEmpty()) {
            adapter.updateList(allStudents);
            return;
        }
        List<Student> filtered = new ArrayList<>();
        for (Student s : allStudents) {
            if (s.getName().toLowerCase().contains(query.toLowerCase()) ||
                s.getMajor().toLowerCase().contains(query.toLowerCase())) {
                filtered.add(s);
            }
        }
        adapter.updateList(filtered);
    }
}
```

---

## 34.7 Activity Layout

```xml
<!-- res/layout/activity_student_list.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <EditText
        android:id="@+id/etSearch"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="ค้นหานักเรียน..."
        android:drawableStart="@drawable/ic_search"
        android:padding="12dp"
        android:background="@android:color/white"
        android:elevation="4dp" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="4dp"
        android:clipToPadding="false" />

</LinearLayout>
```

---

## 34.8 Grid Layout

```java
// Grid layout (2 columns)
GridLayoutManager gridLayoutManager = new GridLayoutManager(this, 2);
recyclerView.setLayoutManager(gridLayoutManager);

// Staggered grid (Pinterest-like)
StaggeredGridLayoutManager staggeredManager =
    new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL);
recyclerView.setLayoutManager(staggeredManager);

// Horizontal list
LinearLayoutManager horizontalManager =
    new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false);
recyclerView.setLayoutManager(horizontalManager);
```

---

## 34.9 DiffUtil (Better Updates)

```java
// ใช้ DiffUtil แทน notifyDataSetChanged() สำหรับ performance ที่ดีกว่า
import androidx.recyclerview.widget.DiffUtil;
import androidx.recyclerview.widget.ListAdapter;

public class StudentListAdapter extends ListAdapter<Student, StudentListAdapter.ViewHolder> {

    public StudentListAdapter() {
        super(DIFF_CALLBACK);
    }

    private static final DiffUtil.ItemCallback<Student> DIFF_CALLBACK =
        new DiffUtil.ItemCallback<Student>() {
            @Override
            public boolean areItemsTheSame(@NonNull Student oldItem, @NonNull Student newItem) {
                return oldItem.getId() == newItem.getId();
            }
            @Override
            public boolean areContentsTheSame(@NonNull Student oldItem, @NonNull Student newItem) {
                return oldItem.getName().equals(newItem.getName()) &&
                       oldItem.getGpa() == newItem.getGpa();
            }
        };

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
            .inflate(R.layout.item_student, parent, false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        holder.bind(getItem(position));
    }

    // ViewHolder class
    static class ViewHolder extends RecyclerView.ViewHolder {
        TextView tvName, tvMajor;

        ViewHolder(@NonNull View itemView) {
            super(itemView);
            tvName = itemView.findViewById(R.id.tvName);
            tvMajor = itemView.findViewById(R.id.tvMajor);
        }

        void bind(Student student) {
            tvName.setText(student.getName());
            tvMajor.setText(student.getMajor());
        }
    }
}

// Usage: adapter.submitList(newList);  // DiffUtil calculates changes automatically
```

---

## 34.10 สรุป Part 34

ในบทนี้คุณได้เรียนรู้:

✅ RecyclerView components  
✅ ViewHolder pattern  
✅ Adapter (onCreateViewHolder, onBindViewHolder)  
✅ LayoutManager (Linear, Grid, Staggered)  
✅ Click/LongClick listeners  
✅ Search/filter  
✅ Add/remove items  
✅ DiffUtil (efficient updates)  
✅ Glide (image loading)  

---

*[← Part 33: Intents & Navigation](./part-33-android-intents.md) | [Part 35: Android Fragments →](./part-35-android-fragments.md)*
