# Part 96: Android Architecture - Final Review
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 96.1 Complete Clean Architecture

```
สถาปัตยกรรมระดับ Production:

app/
├── di/                          ← Hilt modules
├── presentation/                ← UI Layer
│   ├── base/
│   │   ├── BaseActivity.java
│   │   ├── BaseFragment.java
│   │   └── BaseViewModel.java
│   ├── home/
│   │   ├── HomeFragment.java
│   │   ├── HomeViewModel.java
│   │   └── HomeUiState.java
│   └── product/
│       ├── ProductDetailFragment.java
│       └── ProductDetailViewModel.java
├── domain/                      ← Business Layer (no Android deps!)
│   ├── model/
│   │   ├── Product.java
│   │   └── Order.java
│   ├── repository/              ← Interfaces only
│   │   ├── ProductRepository.java
│   │   └── OrderRepository.java
│   └── usecase/
│       ├── GetProductsUseCase.java
│       ├── SearchProductsUseCase.java
│       └── PlaceOrderUseCase.java
└── data/                        ← Data Layer
    ├── local/
    │   ├── AppDatabase.java
    │   ├── entity/
    │   │   └── ProductEntity.java
    │   └── dao/
    │       └── ProductDao.java
    ├── remote/
    │   ├── ApiService.java
    │   └── dto/
    │       └── ProductDto.java
    └── repository/
        └── ProductRepositoryImpl.java    ← implements domain.ProductRepository
```

---

## 96.2 Layer Communication

```java
// Domain layer: pure Java, no Android
public interface ProductRepository {
    CompletableFuture<List<Product>> getProducts(String category);
    CompletableFuture<Product>       getProductById(String id);
    CompletableFuture<Void>          saveProduct(Product product);
}

// Data layer: implements domain interface
public class ProductRepositoryImpl implements ProductRepository {
    
    private final ProductDao localDao;
    private final ApiService remoteApi;
    private final ProductMapper mapper;
    
    @Inject
    public ProductRepositoryImpl(ProductDao localDao, ApiService remoteApi, ProductMapper mapper) {
        this.localDao  = localDao;
        this.remoteApi = remoteApi;
        this.mapper    = mapper;
    }
    
    @Override
    public CompletableFuture<List<Product>> getProducts(String category) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                // 1. Try remote
                Response<List<ProductDto>> response = 
                    remoteApi.getProducts(category).execute();
                
                if (response.isSuccessful() && response.body() != null) {
                    List<Product> products = mapper.toDomainList(response.body());
                    
                    // 2. Update local cache
                    List<ProductEntity> entities = mapper.toEntityList(products);
                    localDao.insertAll(entities);
                    
                    return products;
                }
            } catch (Exception networkError) {
                Log.w("Repo", "Network failed, using cache", networkError);
            }
            
            // 3. Fallback to local
            return mapper.toDomainList(localDao.getAll());
        });
    }
}

// Mapper: DTO ↔ Entity ↔ Domain
public class ProductMapper {
    
    // DTO → Domain (from network response)
    public Product toDomain(ProductDto dto) {
        return new Product(
            dto.id, dto.name, dto.description,
            dto.originalPrice, dto.discountPrice,
            dto.imageUrl, dto.category,
            dto.stock, dto.rating, dto.reviewCount);
    }
    
    // Domain → Entity (for Room)
    public ProductEntity toEntity(Product product) {
        ProductEntity entity = new ProductEntity();
        entity.id           = product.getId();
        entity.name         = product.getName();
        entity.originalPrice = product.getOriginalPrice();
        entity.discountPrice = product.getDiscountPrice();
        entity.imageUrl     = product.getImageUrl();
        entity.category     = product.getCategory();
        entity.stock        = product.getStock();
        entity.rating       = product.getRating();
        entity.reviewCount  = product.getReviewCount();
        return entity;
    }
    
    // Entity → Domain (from Room)
    public Product toDomain(ProductEntity entity) {
        return new Product(
            entity.id, entity.name, null,
            entity.originalPrice, entity.discountPrice,
            entity.imageUrl, entity.category,
            entity.stock, entity.rating, entity.reviewCount);
    }
    
    public List<Product>       toDomainList(List<ProductDto> dtos)         { return dtos.stream().map(this::toDomain).collect(Collectors.toList()); }
    public List<Product>       toDomainList(List<ProductEntity> entities)  { return entities.stream().map(this::toDomain).collect(Collectors.toList()); }
    public List<ProductEntity> toEntityList(List<Product> products)        { return products.stream().map(this::toEntity).collect(Collectors.toList()); }
}
```

---

## 96.3 Hilt Module Setup

```java
// NetworkModule.java
@Module
@InstallIn(SingletonComponent.class)
public class NetworkModule {
    
    @Provides @Singleton
    public OkHttpClient provideOkHttpClient(AuthInterceptor authInterceptor) {
        return new OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(new HttpLoggingInterceptor()
                .setLevel(BuildConfig.DEBUG 
                    ? HttpLoggingInterceptor.Level.BODY 
                    : HttpLoggingInterceptor.Level.NONE))
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build();
    }
    
    @Provides @Singleton
    public Retrofit provideRetrofit(OkHttpClient client) {
        return new Retrofit.Builder()
            .baseUrl(BuildConfig.API_BASE_URL)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build();
    }
    
    @Provides @Singleton
    public ApiService provideApiService(Retrofit retrofit) {
        return retrofit.create(ApiService.class);
    }
}

// DatabaseModule.java
@Module
@InstallIn(SingletonComponent.class)
public class DatabaseModule {
    
    @Provides @Singleton
    public AppDatabase provideDatabase(@ApplicationContext Context context) {
        return AppDatabase.create(context);
    }
    
    @Provides
    public ProductDao provideProductDao(AppDatabase database) {
        return database.productDao();
    }
    
    @Provides
    public OrderDao provideOrderDao(AppDatabase database) {
        return database.orderDao();
    }
}

// RepositoryModule.java
@Module
@InstallIn(SingletonComponent.class)
public abstract class RepositoryModule {
    
    @Binds @Singleton
    public abstract ProductRepository bindProductRepository(ProductRepositoryImpl impl);
    
    @Binds @Singleton
    public abstract OrderRepository bindOrderRepository(OrderRepositoryImpl impl);
}
```

---

## 96.4 สรุป Part 96

ในบทนี้คุณได้เรียนรู้:

✅ Complete Clean Architecture folder structure  
✅ Layer communication (Domain ↔ Data ↔ Presentation)  
✅ Repository pattern (interface in domain, impl in data)  
✅ Network-first with local fallback  
✅ Mapper classes (DTO/Entity/Domain)  
✅ Hilt NetworkModule, DatabaseModule, RepositoryModule  
✅ @Binds vs @Provides in Hilt  

---

*[← Part 95: Database Migration](./part-95-database-migration.md) | [Part 97: Kotlin Coroutines Interop →](./part-97-kotlin-coroutines-interop.md)*
