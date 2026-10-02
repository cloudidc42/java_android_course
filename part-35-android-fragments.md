# Part 35: Fragments
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 35.1 Fragment คืออะไร

Fragment คือ "ส่วนย่อยของ UI" ที่สามารถ reuse ได้ใน Activity หลายๆ ตัว  
เหมาะสำหรับ tablet layout (หลาย pane) และ navigation

```
Activity (host)
├── Fragment A (list)
└── Fragment B (detail)
```

---

## 35.2 สร้าง Fragment

```java
// HomeFragment.java
package com.example.myapp.fragment;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class HomeFragment extends Fragment {

    private static final String ARG_TITLE = "title";

    // Factory method (preferred over constructor with args)
    public static HomeFragment newInstance(String title) {
        HomeFragment fragment = new HomeFragment();
        Bundle args = new Bundle();
        args.putString(ARG_TITLE, title);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Retrieve arguments
        if (getArguments() != null) {
            String title = getArguments().getString(ARG_TITLE);
        }
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        // Inflate the fragment layout
        return inflater.inflate(R.layout.fragment_home, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        // Initialize views here (view is ready)
        TextView tvWelcome = view.findViewById(R.id.tvWelcome);
        tvWelcome.setText("ยินดีต้อนรับสู่หน้าหลัก");
    }
}
```

```xml
<!-- res/layout/fragment_home.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvWelcome"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textStyle="bold" />

</LinearLayout>
```

---

## 35.3 Fragment Transactions

```java
// MainActivity.java - จัดการ Fragments
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Load initial fragment (only if fresh start, not rotation)
        if (savedInstanceState == null) {
            loadFragment(new HomeFragment(), "home");
        }
    }

    private void loadFragment(Fragment fragment, String tag) {
        getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.fragmentContainer, fragment, tag)
            .commit();
    }

    private void addFragment(Fragment fragment, String tag) {
        getSupportFragmentManager()
            .beginTransaction()
            .add(R.id.fragmentContainer, fragment, tag)
            .addToBackStack(tag)  // allow back press to pop
            .commit();
    }

    // Navigate to detail with animation
    private void navigateToDetail(int itemId) {
        DetailFragment detail = DetailFragment.newInstance(itemId);
        getSupportFragmentManager()
            .beginTransaction()
            .setCustomAnimations(
                R.anim.slide_in_right,   // enter
                R.anim.slide_out_left,   // exit
                R.anim.slide_in_left,    // pop enter
                R.anim.slide_out_right)  // pop exit
            .replace(R.id.fragmentContainer, detail, "detail")
            .addToBackStack("detail")
            .commit();
    }

    // Find fragment by tag
    private Fragment findFragment(String tag) {
        return getSupportFragmentManager().findFragmentByTag(tag);
    }
}
```

```xml
<!-- res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragmentContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

---

## 35.4 Fragment-Activity Communication

```java
// Pattern: Interface callback

// ListFragment.java
public class ListFragment extends Fragment {

    // Interface for communication
    public interface OnItemSelectedListener {
        void onItemSelected(int itemId, String itemName);
    }

    private OnItemSelectedListener listener;

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        // Activity must implement the interface
        if (context instanceof OnItemSelectedListener) {
            listener = (OnItemSelectedListener) context;
        } else {
            throw new RuntimeException(context + " must implement OnItemSelectedListener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        listener = null;
    }

    private void onItemClick(int id, String name) {
        if (listener != null) {
            listener.onItemSelected(id, name);
        }
    }
}

// MainActivity.java - implements the interface
public class MainActivity extends AppCompatActivity
        implements ListFragment.OnItemSelectedListener {

    @Override
    public void onItemSelected(int itemId, String itemName) {
        // Handle selection (e.g., load detail fragment)
        DetailFragment detail = DetailFragment.newInstance(itemId);
        getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.detailContainer, detail)
            .commit();
    }
}
```

---

## 35.5 Bottom Navigation with Fragments

```java
// MainActivity.java with BottomNavigationView
public class MainActivity extends AppCompatActivity {

    private BottomNavigationView bottomNav;
    private Fragment homeFragment, searchFragment, profileFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bottom_nav);

        bottomNav = findViewById(R.id.bottomNavigation);

        // Initialize fragments
        homeFragment    = new HomeFragment();
        searchFragment  = new SearchFragment();
        profileFragment = new ProfileFragment();

        // Load default fragment
        loadFragment(homeFragment);

        bottomNav.setOnItemSelectedListener(item -> {
            int id = item.getItemId();
            if (id == R.id.nav_home) {
                loadFragment(homeFragment);
                return true;
            } else if (id == R.id.nav_search) {
                loadFragment(searchFragment);
                return true;
            } else if (id == R.id.nav_profile) {
                loadFragment(profileFragment);
                return true;
            }
            return false;
        });
    }

    private void loadFragment(Fragment fragment) {
        getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.fragmentContainer, fragment)
            .commit();
    }
}
```

```xml
<!-- res/layout/activity_bottom_nav.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNavigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/white"
        app:menu="@menu/bottom_nav_menu" />

</LinearLayout>
```

```xml
<!-- res/menu/bottom_nav_menu.xml -->
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/nav_home"
        android:icon="@drawable/ic_home"
        android:title="หน้าหลัก" />
    <item
        android:id="@+id/nav_search"
        android:icon="@drawable/ic_search"
        android:title="ค้นหา" />
    <item
        android:id="@+id/nav_profile"
        android:icon="@drawable/ic_person"
        android:title="โปรไฟล์" />
</menu>
```

---

## 35.6 Fragment Lifecycle

```java
// Fragment Lifecycle (เรียงตามลำดับ)

onAttach()         // Fragment attached to Activity
onCreate()         // Fragment created (no UI yet)
onCreateView()     // Inflate fragment layout
onViewCreated()    // View ready - setup listeners here
onStart()          // Fragment visible
onResume()         // Fragment interactive
--- (user uses fragment) ---
onPause()          // Fragment loses focus
onStop()           // Fragment not visible
onDestroyView()    // View destroyed (Fragment still exists)
onDestroy()        // Fragment destroyed
onDetach()         // Fragment detached from Activity

// IMPORTANT: Do NOT reference views after onDestroyView()
// Use ViewBinding with null-check in onDestroyView()
```

---

## 35.7 ViewBinding in Fragments

```java
// FragmentWithBinding.java
import com.example.myapp.databinding.FragmentExampleBinding;

public class ExampleFragment extends Fragment {

    private FragmentExampleBinding binding;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        binding = FragmentExampleBinding.inflate(inflater, container, false);
        return binding.getRoot();
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        // Access views directly through binding
        binding.tvTitle.setText("Hello Fragment!");
        binding.btnAction.setOnClickListener(v -> {
            binding.tvResult.setText("Clicked!");
        });
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;  // IMPORTANT: prevent memory leak
    }
}
```

```groovy
// Enable ViewBinding in build.gradle
android {
    buildFeatures {
        viewBinding true
    }
}
```

---

## 35.8 สรุป Part 35

ในบทนี้คุณได้เรียนรู้:

✅ Fragment พื้นฐาน (onCreateView, onViewCreated)  
✅ Fragment arguments (newInstance pattern)  
✅ Fragment transactions (add, replace, addToBackStack)  
✅ Fragment-Activity communication (interface callback)  
✅ Bottom Navigation with Fragments  
✅ Fragment lifecycle  
✅ ViewBinding in Fragments  

---

*[← Part 34: RecyclerView](./part-34-android-recyclerview.md) | [Part 36: Android Room Database →](./part-36-android-room.md)*
