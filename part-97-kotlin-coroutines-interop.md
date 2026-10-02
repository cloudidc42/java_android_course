# Part 97: Kotlin Coroutines Interop for Java Developers
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 97.1 ทำไม Java Developer ต้องรู้ Coroutines

```
Android ecosystem ปัจจุบัน (2024):
- Kotlin Coroutines = มาตรฐานใหม่สำหรับ async ใน Android
- Library ใหม่หลายตัวใช้ suspend fun / Flow
- Google Architecture Components (Room, Retrofit) มี Coroutines support

Java Developer ควรรู้:
1. วิธีเรียก suspend function จาก Java code
2. วิธีเรียก Kotlin Flow จาก Java/LiveData
3. วิธี migrate Java CompletableFuture → Kotlin suspend
```

---

## 97.2 Call Kotlin Suspend Function from Java

```kotlin
// Kotlin side: suspend function
// ProductRepository.kt
interface ProductRepository {
    suspend fun getProducts(category: String?): List<Product>
    suspend fun getProductById(id: String): Product?
}

// ProductRepositoryImpl.kt
class ProductRepositoryImpl @Inject constructor(
    private val dao: ProductDao,
    private val api: ApiService
) : ProductRepository {
    
    override suspend fun getProducts(category: String?): List<Product> {
        return withContext(Dispatchers.IO) {
            api.getProducts(category).map { it.toDomain() }
        }
    }
}
```

```java
// Java side: call suspend function
// Using kotlinx.coroutines.future.FutureKt (bridges suspend → CompletableFuture)
import kotlinx.coroutines.future.FutureKt;
import kotlinx.coroutines.GlobalScope;

// In Java ViewModel (calling Kotlin suspend)
public class ProductViewModelJava extends ViewModel {
    
    private final ProductRepository repository;  // Kotlin interface
    
    public ProductViewModelJava(ProductRepository repository) {
        this.repository = repository;
    }
    
    public CompletableFuture<List<Product>> getProducts(String category) {
        // Convert suspend function to CompletableFuture
        return FutureKt.future(
            GlobalScope.INSTANCE,
            EmptyCoroutineContext.INSTANCE,
            CoroutineStart.DEFAULT,
            (scope, continuation) -> repository.getProducts(category, continuation)
        );
    }
}

// Better: Use CoroutineScope tied to ViewModel lifecycle
// In Kotlin ViewModel that Java code calls:
@HiltViewModel
class ProductViewModel @Inject constructor(
    private val repository: ProductRepository,
    private val errorHandler: ErrorHandler
) : ViewModel() {
    
    // Java-callable: returns LiveData
    val products: LiveData<List<Product>> = liveData {
        try {
            emit(repository.getProducts(null))
        } catch (e: Exception) {
            // handle error
        }
    }
}
```

---

## 97.3 Kotlin Flow in Java

```kotlin
// Kotlin: emit Flow
@Dao
interface ProductDao {
    @Query("SELECT * FROM products WHERE category = :category")
    fun getProductsFlow(category: String): Flow<List<ProductEntity>>
}
```

```java
// Java: convert Flow to LiveData via asLiveData()
// OR use RxJava bridge
import kotlinx.coroutines.flow.Flow;
import androidx.lifecycle.LiveData;
import androidx.lifecycle.LiveDataKt;  // asLiveData extension

// Wrap in ViewModel (Kotlin, callable from Java Fragment)
@HiltViewModel
class ProductViewModel @Inject constructor(
    private val dao: ProductDao
) : ViewModel() {
    
    // Java Fragment can observe this LiveData normally
    val productsLiveData: LiveData<List<ProductEntity>> =
        dao.getProductsFlow("electronics").asLiveData()
}
```

---

## 97.4 Practical Migration Path

```
Migration strategy for Java → Kotlin with Coroutines:

1. Start with data layer (Repository):
   - Add Kotlin files alongside Java
   - Create KotlinProductRepository.kt implementing same interface
   - Gradually replace Java repository

2. Add CoroutineScope to ViewModel:
   - Keep Java ViewModel
   - Add Kotlin extension file for launch helpers
   - Use viewModelScope.launch { ... }

3. Convert UI last (Fragment/Activity):
   - Keep Java Fragment
   - Observe Kotlin ViewModel's LiveData the same way
   - Java Fragment doesn't need to know about Coroutines

Example: Java Fragment observes Kotlin ViewModel
```

```java
// Java Fragment (unchanged, still works!)
public class ProductFragment extends Fragment {
    
    private ProductViewModel viewModel;  // Now Kotlin ViewModel
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        viewModel = new ViewModelProvider(this).get(ProductViewModel.class);
        
        // Observes LiveData from Kotlin ViewModel (no change needed!)
        viewModel.getProducts().observe(getViewLifecycleOwner(), products -> {
            adapter.submitList(products);
        });
    }
}
```

---

## 97.5 RxJava ↔ Coroutines Bridge

```kotlin
// Convert RxJava Observable → Flow
import kotlinx.coroutines.rx3.asFlow

val productFlow: Flow<List<Product>> = 
    rxProductObservable.asFlow()

// Convert Flow → RxJava Observable
import kotlinx.coroutines.rx3.asObservable

val productObservable: Observable<List<Product>> =
    productFlow.asObservable()

// Convert suspend → Single
import kotlinx.coroutines.rx3.rxSingle

val productSingle: Single<Product> = rxSingle {
    repository.getProductById("123")
}
```

---

## 97.6 สรุป Part 97

ในบทนี้คุณได้เรียนรู้:

✅ ทำไม Java dev ต้องรู้ Coroutines  
✅ Call Kotlin suspend function from Java (FutureKt)  
✅ Kotlin Flow → LiveData (asLiveData)  
✅ Java Fragment observes Kotlin ViewModel (unchanged!)  
✅ Migration strategy: data layer first  
✅ RxJava ↔ Coroutines bridge  
✅ Practical interoperability patterns  

---

*[← Part 96: Architecture Final](./part-96-architecture-final.md) | [Part 98: Production Checklist →](./part-98-production-checklist.md)*
