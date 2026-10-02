# Part 77: Architecture Components Deep Dive
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 77.1 LiveData ขั้นสูง

```java
// MediatorLiveData: observe multiple sources
public class SearchViewModel extends ViewModel {
    
    // Separate live data for different criteria
    private final MutableLiveData<String> searchQuery = new MutableLiveData<>("");
    private final MutableLiveData<String> category = new MutableLiveData<>("all");
    private final MutableLiveData<String> sortOrder = new MutableLiveData<>("name");
    
    // Combine all criteria into one search result
    public final LiveData<List<Product>> searchResults;
    
    public SearchViewModel(ProductRepository repository) {
        MediatorLiveData<List<Product>> mediator = new MediatorLiveData<>();
        
        Observer<Object> triggerSearch = ignored -> {
            String query = searchQuery.getValue();
            String cat   = category.getValue();
            String sort  = sortOrder.getValue();
            
            // Fetch whenever any criteria changes
            repository.search(query, cat, sort)
                .thenAccept(results -> mediator.postValue(results));
        };
        
        mediator.addSource(searchQuery, triggerSearch);
        mediator.addSource(category,    triggerSearch);
        mediator.addSource(sortOrder,   triggerSearch);
        
        searchResults = mediator;
    }
    
    public void setQuery(String q)    { searchQuery.setValue(q); }
    public void setCategory(String c) { category.setValue(c); }
    public void setSort(String s)     { sortOrder.setValue(s); }
}

// Transformations.switchMap: swap LiveData source
public class UserDetailViewModel extends ViewModel {
    
    private final MutableLiveData<Integer> userId = new MutableLiveData<>();
    
    public final LiveData<User> user = Transformations.switchMap(userId,
        id -> userRepository.getUserById(id));  // returns new LiveData for each id
    
    public void loadUser(int id) { userId.setValue(id); }
}

// Custom LiveData with lifecycle awareness
public class NetworkStateLiveData extends LiveData<Boolean> {
    
    private final ConnectivityManager cm;
    
    private final ConnectivityManager.NetworkCallback callback = 
        new ConnectivityManager.NetworkCallback() {
            @Override
            public void onAvailable(Network network) { postValue(true); }
            
            @Override
            public void onLost(Network network) { postValue(false); }
        };
    
    public NetworkStateLiveData(Context context) {
        cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
    }
    
    @Override
    protected void onActive() {
        // Start observing when there's an active observer
        NetworkRequest request = new NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .build();
        cm.registerNetworkCallback(request, callback);
    }
    
    @Override
    protected void onInactive() {
        // Stop when no observers (save battery)
        cm.unregisterNetworkCallback(callback);
    }
}
```

---

## 77.2 ViewModel SavedStateHandle

```java
// Survive process death (low memory kill + restore)
@HiltViewModel
public class ProductDetailViewModel extends ViewModel {
    
    private static final String KEY_PRODUCT_ID = "product_id";
    private static final String KEY_SCROLL_POS = "scroll_pos";
    
    private final SavedStateHandle savedStateHandle;
    private final ProductRepository repository;
    
    @Inject
    public ProductDetailViewModel(SavedStateHandle savedState, ProductRepository repository) {
        this.savedStateHandle = savedState;
        this.repository = repository;
        
        // Auto-load when productId is set (from SavedState or navigation args)
        String productId = savedState.get(KEY_PRODUCT_ID);
        if (productId != null) loadProduct(productId);
    }
    
    public void setProductId(String id) {
        savedStateHandle.set(KEY_PRODUCT_ID, id);
        loadProduct(id);
    }
    
    public void saveScrollPosition(int position) {
        savedStateHandle.set(KEY_SCROLL_POS, position);
    }
    
    public int getScrollPosition() {
        Integer pos = savedStateHandle.get(KEY_SCROLL_POS);
        return pos != null ? pos : 0;
    }
    
    // LiveData backed by SavedStateHandle (auto survives process death)
    public LiveData<String> getProductIdLiveData() {
        return savedStateHandle.getLiveData(KEY_PRODUCT_ID);
    }
    
    private void loadProduct(String id) {
        // ... load from repo
    }
}
```

---

## 77.3 ViewBinding Best Practices

```java
// BaseFragment.java with ViewBinding
public abstract class BaseFragment<VB extends ViewBinding> extends Fragment {
    
    private VB _binding;
    protected VB binding;  // null after onDestroyView
    
    protected abstract VB inflateBinding(LayoutInflater inflater, ViewGroup container);
    
    @Override
    public final View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedState) {
        _binding = inflateBinding(inflater, container);
        binding = _binding;
        return _binding.getRoot();
    }
    
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        _binding = null;
        binding = null;  // prevent memory leaks
    }
}

// Concrete fragment
public class HomeFragment extends BaseFragment<FragmentHomeBinding> {
    
    @Override
    protected FragmentHomeBinding inflateBinding(LayoutInflater inflater, ViewGroup container) {
        return FragmentHomeBinding.inflate(inflater, container, false);
    }
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        binding.recyclerView.setAdapter(adapter);
        binding.swipeRefresh.setOnRefreshListener(this::refresh);
        binding.btnSearch.setOnClickListener(v -> openSearch());
    }
}
```

---

## 77.4 SharedViewModel

```java
// Share ViewModel between fragments in the same Activity
// Activity acts as owner - ViewModel lives as long as Activity

// In each Fragment:
SharedViewModel sharedViewModel = new ViewModelProvider(requireActivity())
    .get(SharedViewModel.class);

// SharedViewModel.java
@HiltViewModel
public class SharedViewModel extends ViewModel {
    
    private final MutableLiveData<Product> selectedProduct = new MutableLiveData<>();
    private final MutableLiveData<Cart> cart = new MutableLiveData<>(new Cart(new ArrayList<>()));
    
    public void selectProduct(Product product) {
        selectedProduct.setValue(product);
    }
    
    public void addToCart(Product product) {
        Cart currentCart = cart.getValue();
        // update cart...
        cart.setValue(updatedCart);
    }
    
    public LiveData<Product> getSelectedProduct() { return selectedProduct; }
    public LiveData<Cart>    getCart()             { return cart; }
}
```

---

## 77.5 สรุป Part 77

ในบทนี้คุณได้เรียนรู้:

✅ MediatorLiveData (combine multiple sources)  
✅ Transformations.switchMap  
✅ Custom LiveData (NetworkStateLiveData)  
✅ Lifecycle awareness (onActive/onInactive)  
✅ SavedStateHandle (survive process death)  
✅ ViewBinding base class pattern  
✅ SharedViewModel (Activity-scoped)  

---

*[← Part 76: Java Patterns](./part-76-java-patterns-world-class.md) | [Part 78: Deep Linking & App Links →](./part-78-android-deep-links.md)*
