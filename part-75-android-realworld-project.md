# Part 75: Real-World Project - E-Commerce App
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 75.1 โครงสร้างโปรเจกต์

```
ShopApp/
├── app/
│   └── src/main/java/com/shopapp/
│       ├── di/               ← Hilt modules
│       ├── presentation/
│       │   ├── auth/         ← Login, Register screens
│       │   ├── home/         ← Product list
│       │   ├── product/      ← Product detail
│       │   ├── cart/         ← Shopping cart
│       │   ├── checkout/     ← Order flow
│       │   └── profile/      ← User profile, orders
│       ├── domain/
│       │   ├── model/        ← Product, Cart, Order, User
│       │   ├── repository/   ← Interfaces
│       │   └── usecase/      ← Business logic
│       └── data/
│           ├── local/        ← Room entities, DAOs
│           ├── remote/       ← Retrofit, DTOs
│           └── repository/   ← Implementations
└── gradle/
    └── libs.versions.toml
```

---

## 75.2 Domain Models

```java
// domain/model/Product.java
public class Product {
    private final String id;
    private final String name;
    private final String description;
    private final double originalPrice;
    private final double discountPrice;
    private final String imageUrl;
    private final String category;
    private final int stock;
    private final double rating;
    private final int reviewCount;
    
    public Product(String id, String name, String description,
            double originalPrice, double discountPrice,
            String imageUrl, String category,
            int stock, double rating, int reviewCount) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.originalPrice = originalPrice;
        this.discountPrice = discountPrice;
        this.imageUrl = imageUrl;
        this.category = category;
        this.stock = stock;
        this.rating = rating;
        this.reviewCount = reviewCount;
    }
    
    public boolean hasDiscount() {
        return discountPrice < originalPrice;
    }
    
    public int getDiscountPercent() {
        if (!hasDiscount()) return 0;
        return (int) ((originalPrice - discountPrice) / originalPrice * 100);
    }
    
    public boolean isInStock() { return stock > 0; }
    
    // Getters
    public String getId()          { return id; }
    public String getName()        { return name; }
    public String getDescription() { return description; }
    public double getOriginalPrice(){ return originalPrice; }
    public double getDiscountPrice(){ return discountPrice; }
    public String getImageUrl()    { return imageUrl; }
    public String getCategory()    { return category; }
    public int    getStock()       { return stock; }
    public double getRating()      { return rating; }
    public int    getReviewCount() { return reviewCount; }
}

// domain/model/CartItem.java
public class CartItem {
    private final String id;
    private final Product product;
    private int quantity;
    
    public CartItem(Product product, int quantity) {
        this.id = UUID.randomUUID().toString();
        this.product = product;
        this.quantity = quantity;
    }
    
    public double getSubtotal() {
        return product.getDiscountPrice() * quantity;
    }
    
    public void setQuantity(int q) { this.quantity = Math.max(1, q); }
    
    public String getId()      { return id; }
    public Product getProduct(){ return product; }
    public int getQuantity()   { return quantity; }
}

// domain/model/Cart.java
public class Cart {
    private final List<CartItem> items;
    
    public Cart(List<CartItem> items) {
        this.items = new ArrayList<>(items);
    }
    
    public double getTotalPrice() {
        return items.stream().mapToDouble(CartItem::getSubtotal).sum();
    }
    
    public int getTotalItems() {
        return items.stream().mapToInt(CartItem::getQuantity).sum();
    }
    
    public boolean isEmpty() { return items.isEmpty(); }
    
    public List<CartItem> getItems() { return Collections.unmodifiableList(items); }
}
```

---

## 75.3 Product List ViewModel

```java
// presentation/home/HomeViewModel.java
@HiltViewModel
public class HomeViewModel extends ViewModel {
    
    private final GetProductsUseCase getProductsUseCase;
    private final SearchProductsUseCase searchProductsUseCase;
    private final AddToCartUseCase addToCartUseCase;
    
    private final MutableLiveData<HomeUiState> uiState = 
        new MutableLiveData<>(HomeUiState.loading());
    private final MutableLiveData<String> toastMessage = new MutableLiveData<>();
    private final MutableLiveData<Cart> cart = new MutableLiveData<>();
    
    private String currentCategory = null;
    private String currentQuery    = null;
    
    @Inject
    public HomeViewModel(GetProductsUseCase getProductsUseCase,
            SearchProductsUseCase searchProductsUseCase,
            AddToCartUseCase addToCartUseCase) {
        this.getProductsUseCase = getProductsUseCase;
        this.searchProductsUseCase = searchProductsUseCase;
        this.addToCartUseCase = addToCartUseCase;
        loadProducts(null);
    }
    
    public void loadProducts(String category) {
        this.currentCategory = category;
        uiState.setValue(HomeUiState.loading());
        
        getProductsUseCase.execute(category)
            .thenAccept(result -> {
                if (result.isSuccess()) {
                    uiState.postValue(HomeUiState.success(result.getData()));
                } else {
                    uiState.postValue(HomeUiState.error(result.getError()));
                }
            });
    }
    
    public void search(String query) {
        if (query.length() < 2) {
            loadProducts(currentCategory);
            return;
        }
        
        currentQuery = query;
        searchProductsUseCase.execute(query)
            .thenAccept(result -> {
                if (result.isSuccess()) {
                    uiState.postValue(HomeUiState.success(result.getData()));
                }
            });
    }
    
    public void addToCart(Product product) {
        addToCartUseCase.execute(product, 1)
            .thenAccept(updatedCart -> {
                cart.postValue(updatedCart);
                toastMessage.postValue("เพิ่ม " + product.getName() + " ลงตะกร้าแล้ว");
            });
    }
    
    public LiveData<HomeUiState> getUiState()    { return uiState; }
    public LiveData<String>     getToastMessage() { return toastMessage; }
    public LiveData<Cart>       getCart()         { return cart; }
    
    // HomeUiState
    public static class HomeUiState {
        enum Type { LOADING, SUCCESS, ERROR }
        final Type         type;
        final List<Product> products;
        final String       errorMessage;
        
        private HomeUiState(Type type, List<Product> products, String error) {
            this.type = type;
            this.products = products;
            this.errorMessage = error;
        }
        
        static HomeUiState loading()               { return new HomeUiState(Type.LOADING, null, null); }
        static HomeUiState success(List<Product> p) { return new HomeUiState(Type.SUCCESS, p, null); }
        static HomeUiState error(String msg)        { return new HomeUiState(Type.ERROR, null, msg); }
    }
}
```

---

## 75.4 Product List Fragment

```java
// presentation/home/HomeFragment.java
@AndroidEntryPoint
public class HomeFragment extends Fragment {
    
    private HomeViewModel viewModel;
    private ProductAdapter adapter;
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        viewModel = new ViewModelProvider(this).get(HomeViewModel.class);
        
        RecyclerView recyclerView = view.findViewById(R.id.recyclerProducts);
        SwipeRefreshLayout swipeRefresh = view.findViewById(R.id.swipeRefresh);
        ProgressBar progressBar = view.findViewById(R.id.progressBar);
        TextView tvError = view.findViewById(R.id.tvError);
        SearchView searchView = view.findViewById(R.id.searchView);
        
        // Setup RecyclerView
        GridLayoutManager layoutManager = new GridLayoutManager(requireContext(), 2);
        recyclerView.setLayoutManager(layoutManager);
        
        adapter = new ProductAdapter(product -> {
            // Navigate to detail
            Bundle args = new Bundle();
            args.putString("product_id", product.getId());
            Navigation.findNavController(view)
                .navigate(R.id.action_home_to_productDetail, args);
        }, product -> {
            // Add to cart
            viewModel.addToCart(product);
        });
        recyclerView.setAdapter(adapter);
        
        // Observe UI state
        viewModel.getUiState().observe(getViewLifecycleOwner(), state -> {
            progressBar.setVisibility(state.type == HomeUiState.Type.LOADING ? 
                View.VISIBLE : View.GONE);
            tvError.setVisibility(state.type == HomeUiState.Type.ERROR ? 
                View.VISIBLE : View.GONE);
            
            if (state.type == HomeUiState.Type.SUCCESS && state.products != null) {
                adapter.submitList(state.products);
                swipeRefresh.setRefreshing(false);
            } else if (state.type == HomeUiState.Type.ERROR) {
                tvError.setText(state.errorMessage);
                swipeRefresh.setRefreshing(false);
            }
        });
        
        // Observe toast
        viewModel.getToastMessage().observe(getViewLifecycleOwner(), msg -> {
            if (msg != null) {
                Toast.makeText(requireContext(), msg, Toast.LENGTH_SHORT).show();
            }
        });
        
        // Search
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                viewModel.search(query);
                return true;
            }
            
            @Override
            public boolean onQueryTextChange(String newText) {
                viewModel.search(newText);
                return true;
            }
        });
        
        // Refresh
        swipeRefresh.setOnRefreshListener(() -> viewModel.loadProducts(null));
    }
}
```

---

## 75.5 สรุป Part 75

ในบทนี้คุณได้เรียนรู้:

✅ Real-world project structure (E-Commerce)  
✅ Domain models (Product, CartItem, Cart)  
✅ Business logic in domain model (hasDiscount, getDiscountPercent)  
✅ HomeViewModel with UiState  
✅ Search functionality integration  
✅ Add to cart flow  
✅ HomeFragment with RecyclerView, Search, SwipeRefresh  
✅ Toast messages via LiveData  

---

*[← Part 74: Advanced Testing](./part-74-android-advanced-testing.md) | [Part 76: Advanced Java Patterns →](./part-76-java-patterns-world-class.md)*
