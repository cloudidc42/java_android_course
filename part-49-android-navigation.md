# Part 49: Jetpack Navigation Component
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 49.1 Navigation Component คืออะไร

Navigation Component จาก Jetpack จัดการ navigation ระหว่าง Fragments แทน code ที่ซับซ้อน

**ข้อดี:**
- Visual navigation graph
- Back stack จัดการอัตโนมัติ
- Safe Args (type-safe arguments)
- Deep Links support
- Transition animations built-in

---

## 49.2 Setup

```groovy
// build.gradle
def nav_version = "2.7.5"
implementation "androidx.navigation:navigation-fragment:$nav_version"
implementation "androidx.navigation:navigation-ui:$nav_version"

// Safe Args plugin
// project-level build.gradle:
// classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"

// app-level build.gradle:
// id 'androidx.navigation.safeargs'
```

---

## 49.3 Navigation Graph

```xml
<!-- res/navigation/nav_graph.xml -->
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <!-- Home Fragment -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        android:label="หน้าหลัก"
        tools:layout="@layout/fragment_home">
        
        <!-- Action: navigate to detail -->
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment"
            app:enterAnim="@anim/slide_in_right"
            app:exitAnim="@anim/slide_out_left"
            app:popEnterAnim="@anim/slide_in_left"
            app:popExitAnim="@anim/slide_out_right" />
        
        <!-- Action: navigate to search -->
        <action
            android:id="@+id/action_home_to_search"
            app:destination="@id/searchFragment" />
    </fragment>

    <!-- Detail Fragment -->
    <fragment
        android:id="@+id/detailFragment"
        android:name="com.example.app.DetailFragment"
        android:label="รายละเอียด"
        tools:layout="@layout/fragment_detail">
        
        <!-- Argument (type-safe) -->
        <argument
            android:name="itemId"
            app:argType="integer" />
        <argument
            android:name="itemTitle"
            app:argType="string"
            android:defaultValue="" />
    </fragment>

    <!-- Search Fragment -->
    <fragment
        android:id="@+id/searchFragment"
        android:name="com.example.app.SearchFragment"
        android:label="ค้นหา" />

    <!-- Profile Fragment -->
    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.app.ProfileFragment"
        android:label="โปรไฟล์">
        
        <!-- Deep Link -->
        <deepLink app:uri="myapp://profile/{userId}" />
    </fragment>

</navigation>
```

---

## 49.4 MainActivity with NavController

```java
// MainActivity.java
public class MainActivity extends AppCompatActivity {
    
    private NavController navController;
    private AppBarConfiguration appBarConfiguration;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // NavController from NavHostFragment
        NavHostFragment navHostFragment = (NavHostFragment)
            getSupportFragmentManager().findFragmentById(R.id.navHostFragment);
        navController = navHostFragment.getNavController();
        
        // Setup ActionBar
        appBarConfiguration = new AppBarConfiguration.Builder(
            R.id.homeFragment,      // top-level destinations (no back button)
            R.id.searchFragment,
            R.id.profileFragment
        ).build();
        
        NavigationUI.setupActionBarWithNavController(this, navController, appBarConfiguration);
        
        // Setup BottomNavigationView
        BottomNavigationView bottomNav = findViewById(R.id.bottomNav);
        NavigationUI.setupWithNavController(bottomNav, navController);
        
        // Listen for destination changes
        navController.addOnDestinationChangedListener((controller, destination, arguments) -> {
            String label = destination.getLabel() != null ? destination.getLabel().toString() : "";
            Log.d("Nav", "Navigated to: " + label);
        });
    }
    
    @Override
    public boolean onSupportNavigateUp() {
        return NavigationUI.navigateUp(navController, appBarConfiguration)
            || super.onSupportNavigateUp();
    }
}
```

```xml
<!-- activity_main.xml -->
<androidx.coordinatorlayout.widget.CoordinatorLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize" />

    <!-- NavHostFragment hosts all fragments -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/navHostFragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"
        app:layout_constraintTop_toBottomOf="@id/toolbar"
        app:layout_constraintBottom_toTopOf="@id/bottomNav" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNav"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        app:menu="@menu/bottom_nav_menu" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

---

## 49.5 Navigate Between Fragments

```java
// HomeFragment.java
public class HomeFragment extends Fragment {
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        // Get NavController
        NavController navController = NavHostFragment.findNavController(this);
        
        // Navigate with action ID (simple)
        view.findViewById(R.id.btnDetail).setOnClickListener(v -> {
            navController.navigate(R.id.action_home_to_detail);
        });
        
        // Navigate with arguments (type-safe with Safe Args)
        view.findViewById(R.id.btnDetailWithArgs).setOnClickListener(v -> {
            // Generated by Safe Args plugin
            HomeFragmentDirections.ActionHomeToDetail action =
                HomeFragmentDirections.actionHomeToDetail(42, "สินค้า A");
            navController.navigate(action);
        });
        
        // Navigate with Bundle (without Safe Args)
        view.findViewById(R.id.btnBundleNav).setOnClickListener(v -> {
            Bundle args = new Bundle();
            args.putInt("itemId", 42);
            args.putString("itemTitle", "สินค้า A");
            navController.navigate(R.id.detailFragment, args);
        });
        
        // Navigate with NavOptions (pop back stack)
        view.findViewById(R.id.btnLogin).setOnClickListener(v -> {
            NavOptions options = new NavOptions.Builder()
                .setPopUpTo(R.id.loginFragment, true)  // remove login from stack
                .build();
            navController.navigate(R.id.homeFragment, null, options);
        });
    }
}

// DetailFragment.java
public class DetailFragment extends Fragment {
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        // Receive Safe Args
        DetailFragmentArgs args = DetailFragmentArgs.fromBundle(requireArguments());
        int itemId = args.getItemId();
        String title = args.getItemTitle();
        
        // Or with Bundle:
        // int itemId = requireArguments().getInt("itemId");
        
        ((TextView) view.findViewById(R.id.tvTitle)).setText(title);
        
        // Navigate back
        view.findViewById(R.id.btnBack).setOnClickListener(v -> {
            NavHostFragment.findNavController(this).navigateUp();
        });
        
        // Navigate back with result
        view.findViewById(R.id.btnDone).setOnClickListener(v -> {
            NavController navController = NavHostFragment.findNavController(this);
            navController.getPreviousBackStackEntry()
                .getSavedStateHandle()
                .set("result", "success");
            navController.navigateUp();
        });
    }
}

// HomeFragment.java - receive result
NavController navController = NavHostFragment.findNavController(this);
navController.getCurrentBackStackEntry()
    .getSavedStateHandle()
    .getLiveData("result")
    .observe(getViewLifecycleOwner(), result -> {
        if (result != null) {
            Toast.makeText(requireContext(), "Result: " + result, Toast.LENGTH_SHORT).show();
        }
    });
```

---

## 49.6 Deep Links

```java
// Handle deep link in Activity
// AndroidManifest.xml - already set via <deepLink> in nav_graph

// Manually create deep link intent
Intent deepLinkIntent = new NavDeepLinkBuilder(context)
    .setGraph(R.navigation.nav_graph)
    .setDestination(R.id.profileFragment)
    .setArguments(new ProfileFragmentArgs.Builder("user123").build().toBundle())
    .createTaskStackBuilder()
    .editIntentAt(0);

// Use in notification
PendingIntent pendingIntent = new NavDeepLinkBuilder(context)
    .setGraph(R.navigation.nav_graph)
    .setDestination(R.id.profileFragment)
    .createPendingIntent();
```

---

## 49.7 สรุป Part 49

ในบทนี้คุณได้เรียนรู้:

✅ Navigation Component setup  
✅ Navigation Graph (XML)  
✅ NavHostFragment & NavController  
✅ BottomNavigationView + NavigationUI  
✅ Navigate with actions and Safe Args  
✅ Pass arguments between fragments  
✅ Navigate back and back with result  
✅ Deep Links  
✅ Enter/exit animations on navigation  

---

*[← Part 48: Firebase](./part-48-android-firebase.md) | [Part 50: Hilt Dependency Injection →](./part-50-android-hilt.md)*
