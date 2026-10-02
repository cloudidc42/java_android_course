# Part 86: Jetpack Compose Interoperability
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 86.1 ทำไมต้องรู้ Compose Interop

```
Java/View-based project ปัจจุบัน:
- เพิ่ม Compose screen ใหม่โดยไม่ต้อง rewrite ทั้งหมด
- ใช้ Compose component ใน View layout
- ใช้ View component ใน Compose screen

บทนี้เน้น: Compose เรียก View / View เรียก Compose
(เพื่อ migration path และ reuse)
```

---

## 86.2 ComposeView ใน XML Layout

```xml
<!-- fragment_home.xml -->
<LinearLayout ...>
    
    <TextView android:id="@+id/tvTitle" ... />
    
    <!-- Compose UI embedded in View layout -->
    <androidx.compose.ui.platform.ComposeView
        android:id="@+id/composeProductGrid"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
    
    <Button android:id="@+id/btnCheckout" ... />
    
</LinearLayout>
```

```java
// HomeFragment.java (Java + Compose interop)
public class HomeFragment extends Fragment {
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        ComposeView composeView = view.findViewById(R.id.composeProductGrid);
        
        // Set Compose content from Java
        composeView.setContent(
            // Use Kotlin Compose - requires a Kotlin file or lambda
            // Call a Kotlin composable entry point:
            new ComposeContent()
        );
        
        // Or use setViewCompositionStrategy for lifecycle
        composeView.setViewCompositionStrategy(
            ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed);
    }
}
```

```kotlin
// ProductGridContent.kt (Kotlin Compose side)
@Composable
fun ProductGrid(products: List<Product>, onProductClick: (Product) -> Unit) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(2),
        contentPadding = PaddingValues(8.dp)
    ) {
        items(products) { product ->
            ProductCard(product = product, onClick = { onProductClick(product) })
        }
    }
}

// Entry point callable from Java via ComposeView
class ProductGridComposeContent(
    private val products: List<Product>,
    private val onProductClick: (Product) -> Unit
) : AbstractComposeView(context) {
    @Composable
    override fun Content() {
        MaterialTheme {
            ProductGrid(products = products, onProductClick = onProductClick)
        }
    }
}
```

---

## 86.3 AndroidView ใน Compose (View ใน Compose)

```kotlin
// CustomViewInCompose.kt
@Composable
fun LegacyChartView(data: List<Float>) {
    // Embed any View inside Compose
    AndroidView(
        factory = { context ->
            // Create the old View
            MyCustomChartView(context).apply {
                layoutParams = ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.MATCH_PARENT
                )
            }
        },
        update = { view ->
            // Update when recomposition happens
            view.setData(data)
            view.invalidate()
        },
        modifier = Modifier
            .fillMaxWidth()
            .height(200.dp)
    )
}

// Embed RecyclerView (if needed)
@Composable
fun LegacyRecyclerView(items: List<String>) {
    AndroidView(
        factory = { context ->
            RecyclerView(context).apply {
                layoutManager = LinearLayoutManager(context)
                adapter = SimpleStringAdapter(items)
            }
        },
        update = { recyclerView ->
            (recyclerView.adapter as SimpleStringAdapter).updateData(items)
        }
    )
}
```

---

## 86.4 Share ViewModel between View and Compose

```kotlin
// ViewModel works in both
@HiltViewModel
class ProductViewModel @Inject constructor(
    private val repository: ProductRepository
) : ViewModel() {
    
    val products = MutableLiveData<List<Product>>()
    
    fun loadProducts() {
        viewModelScope.launch {
            products.postValue(repository.getProducts())
        }
    }
}

// In Compose screen (Kotlin)
@AndroidEntryPoint
@Composable
fun ProductScreen() {
    val viewModel: ProductViewModel = hiltViewModel()
    val products by viewModel.products.observeAsState(emptyList())
    
    LaunchedEffect(Unit) { viewModel.loadProducts() }
    
    LazyColumn {
        items(products) { product ->
            Text(product.getName())
        }
    }
}

// In Java Fragment (View-based)
public class LegacyProductFragment extends Fragment {
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        ProductViewModel viewModel = new ViewModelProvider(this).get(ProductViewModel.class);
        viewModel.getProducts().observe(getViewLifecycleOwner(), products -> {
            adapter.submitList(products);
        });
        viewModel.loadProducts();
    }
}
```

---

## 86.5 สรุป Part 86

ในบทนี้คุณได้เรียนรู้:

✅ ComposeView in XML layout  
✅ ViewCompositionStrategy lifecycle  
✅ AndroidView - embed View in Compose  
✅ AndroidView factory + update pattern  
✅ Share ViewModel between View and Compose  
✅ observeAsState in Compose  
✅ Migration path: View → Compose  

---

*[← Part 85: App Shortcuts](./part-85-android-shortcuts.md) | [Part 87: Advanced OkHttp & gRPC →](./part-87-advanced-networking.md)*
