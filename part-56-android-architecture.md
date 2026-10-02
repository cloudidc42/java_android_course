# Part 56: Android Architecture Patterns ขั้นสูง
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 56.1 Clean Architecture

```
Presentation Layer    → UI (Activity, Fragment, ViewModel)
Domain Layer          → Business Logic (UseCase, Entity)
Data Layer            → Repository, DataSource (Room, Retrofit)

Dependency Rule: outer layers depend on inner layers, NEVER reverse
```

```
app/
├── presentation/
│   ├── ui/
│   │   ├── MainActivity.java
│   │   └── HomeFragment.java
│   └── viewmodel/
│       └── HomeViewModel.java
│
├── domain/
│   ├── model/
│   │   └── User.java           (domain entity, no Android imports)
│   ├── repository/
│   │   └── UserRepository.java  (interface only)
│   └── usecase/
│       ├── GetUsersUseCase.java
│       ├── CreateUserUseCase.java
│       └── DeleteUserUseCase.java
│
└── data/
    ├── repository/
    │   └── UserRepositoryImpl.java  (implements domain interface)
    ├── local/
    │   ├── dao/UserDao.java
    │   └── entity/UserEntity.java   (Room entity)
    ├── remote/
    │   ├── api/UserApiService.java
    │   └── dto/UserDto.java         (API response model)
    └── mapper/
        └── UserMapper.java          (entity/dto → domain model)
```

---

## 56.2 Domain Layer

```java
// domain/model/User.java (pure Java, no Android)
public class User {
    private final int id;
    private final String name;
    private final String email;
    private final String role;
    private final long createdAt;
    
    public User(int id, String name, String email, String role, long createdAt) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.role = role;
        this.createdAt = createdAt;
    }
    
    public boolean isAdmin() {
        return "admin".equals(role);
    }
    
    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public String getRole() { return role; }
    public long getCreatedAt() { return createdAt; }
}

// domain/repository/UserRepository.java (interface)
public interface UserRepository {
    LiveData<List<User>> getAllUsers();
    LiveData<User> getUserById(int id);
    CompletableFuture<User> createUser(String name, String email, String role);
    CompletableFuture<Void> deleteUser(int userId);
    CompletableFuture<List<User>> searchUsers(String query);
}

// domain/usecase/GetUsersUseCase.java
public class GetUsersUseCase {
    
    private final UserRepository repository;
    
    @Inject
    public GetUsersUseCase(UserRepository repository) {
        this.repository = repository;
    }
    
    public LiveData<List<User>> execute() {
        return repository.getAllUsers();
    }
}

// domain/usecase/CreateUserUseCase.java
public class CreateUserUseCase {
    
    private final UserRepository repository;
    
    @Inject
    public CreateUserUseCase(UserRepository repository) {
        this.repository = repository;
    }
    
    // Business logic validation here
    public CompletableFuture<Result<User>> execute(String name, String email, String role) {
        if (name == null || name.trim().isEmpty()) {
            return CompletableFuture.completedFuture(Result.failure("Name is required"));
        }
        if (!isValidEmail(email)) {
            return CompletableFuture.completedFuture(Result.failure("Invalid email"));
        }
        
        return repository.createUser(name.trim(), email.trim(), role)
            .thenApply(Result::success)
            .exceptionally(e -> Result.failure(e.getMessage()));
    }
    
    private boolean isValidEmail(String email) {
        return email != null && android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches();
    }
}
```

---

## 56.3 Data Layer

```java
// data/local/entity/UserEntity.java (Room entity)
@Entity(tableName = "users")
public class UserEntity {
    @PrimaryKey(autoGenerate = true)
    public int id;
    
    @ColumnInfo(name = "name")
    public String name;
    
    @ColumnInfo(name = "email")
    public String email;
    
    @ColumnInfo(name = "role")
    public String role;
    
    @ColumnInfo(name = "created_at")
    public long createdAt;
}

// data/remote/dto/UserDto.java (API DTO)
public class UserDto {
    @SerializedName("id")       public int id;
    @SerializedName("name")     public String name;
    @SerializedName("email")    public String email;
    @SerializedName("role")     public String role;
    @SerializedName("created_at") public long createdAt;
}

// data/mapper/UserMapper.java
public class UserMapper {
    
    // Entity → Domain
    public static User toDomain(UserEntity entity) {
        return new User(entity.id, entity.name, entity.email,
            entity.role, entity.createdAt);
    }
    
    // Domain → Entity
    public static UserEntity toEntity(User user) {
        UserEntity entity = new UserEntity();
        entity.id = user.getId();
        entity.name = user.getName();
        entity.email = user.getEmail();
        entity.role = user.getRole();
        entity.createdAt = user.getCreatedAt();
        return entity;
    }
    
    // DTO → Domain
    public static User fromDto(UserDto dto) {
        return new User(dto.id, dto.name, dto.email, dto.role, dto.createdAt);
    }
    
    // List conversion
    public static List<User> toDomainList(List<UserEntity> entities) {
        List<User> users = new ArrayList<>();
        for (UserEntity e : entities) users.add(toDomain(e));
        return users;
    }
}

// data/repository/UserRepositoryImpl.java
public class UserRepositoryImpl implements UserRepository {
    
    private final UserDao userDao;
    private final UserApiService apiService;
    private final ExecutorService executor;
    
    @Inject
    public UserRepositoryImpl(UserDao userDao, UserApiService apiService) {
        this.userDao = userDao;
        this.apiService = apiService;
        this.executor = Executors.newFixedThreadPool(4);
    }
    
    @Override
    public LiveData<List<User>> getAllUsers() {
        // Transform LiveData<List<UserEntity>> to LiveData<List<User>>
        return Transformations.map(userDao.getAllUsers(), UserMapper::toDomainList);
    }
    
    @Override
    public LiveData<User> getUserById(int id) {
        return Transformations.map(userDao.getUserById(id), UserMapper::toDomain);
    }
    
    @Override
    public CompletableFuture<User> createUser(String name, String email, String role) {
        return CompletableFuture.supplyAsync(() -> {
            UserEntity entity = new UserEntity();
            entity.name = name;
            entity.email = email;
            entity.role = role;
            entity.createdAt = System.currentTimeMillis();
            
            long id = userDao.insert(entity);
            entity.id = (int) id;
            return UserMapper.toDomain(entity);
        }, executor);
    }
    
    @Override
    public CompletableFuture<Void> deleteUser(int userId) {
        return CompletableFuture.runAsync(() -> {
            userDao.deleteById(userId);
        }, executor);
    }
    
    @Override
    public CompletableFuture<List<User>> searchUsers(String query) {
        return CompletableFuture.supplyAsync(() -> {
            // Try remote first, fallback to local
            try {
                List<UserDto> dtos = apiService.searchUsers(query).execute().body();
                if (dtos != null) {
                    return dtos.stream().map(UserMapper::fromDto).collect(Collectors.toList());
                }
            } catch (IOException e) {
                Log.w("UserRepo", "Remote search failed, using local");
            }
            
            List<UserEntity> entities = userDao.searchSync("%" + query + "%");
            return UserMapper.toDomainList(entities);
        }, executor);
    }
}
```

---

## 56.4 Presentation Layer

```java
// presentation/viewmodel/HomeViewModel.java
@HiltViewModel
public class HomeViewModel extends ViewModel {
    
    private final GetUsersUseCase getUsersUseCase;
    private final CreateUserUseCase createUserUseCase;
    private final DeleteUserUseCase deleteUserUseCase;
    
    private final MutableLiveData<UiState> uiState = new MutableLiveData<>(UiState.idle());
    
    @Inject
    public HomeViewModel(
            GetUsersUseCase getUsersUseCase,
            CreateUserUseCase createUserUseCase,
            DeleteUserUseCase deleteUserUseCase) {
        this.getUsersUseCase = getUsersUseCase;
        this.createUserUseCase = createUserUseCase;
        this.deleteUserUseCase = deleteUserUseCase;
    }
    
    public LiveData<List<User>> getUsers() {
        return getUsersUseCase.execute();
    }
    
    public LiveData<UiState> getUiState() { return uiState; }
    
    public void createUser(String name, String email) {
        uiState.setValue(UiState.loading());
        
        createUserUseCase.execute(name, email, "user")
            .thenAcceptAsync(result -> {
                if (result.isSuccess()) {
                    uiState.postValue(UiState.success("สร้างผู้ใช้สำเร็จ"));
                } else {
                    uiState.postValue(UiState.error(result.getError()));
                }
            });
    }
    
    // UiState sealed pattern (Java)
    public static class UiState {
        enum Type { IDLE, LOADING, SUCCESS, ERROR }
        
        public final Type type;
        public final String message;
        
        private UiState(Type type, String message) {
            this.type = type;
            this.message = message;
        }
        
        public static UiState idle()               { return new UiState(Type.IDLE, null); }
        public static UiState loading()            { return new UiState(Type.LOADING, null); }
        public static UiState success(String msg)  { return new UiState(Type.SUCCESS, msg); }
        public static UiState error(String msg)    { return new UiState(Type.ERROR, msg); }
        
        public boolean isLoading() { return type == Type.LOADING; }
    }
}
```

---

## 56.5 Result Wrapper

```java
// domain/util/Result.java
public class Result<T> {
    
    private final T data;
    private final String error;
    private final boolean success;
    
    private Result(T data, String error, boolean success) {
        this.data = data;
        this.error = error;
        this.success = success;
    }
    
    public static <T> Result<T> success(T data) {
        return new Result<>(data, null, true);
    }
    
    public static <T> Result<T> failure(String error) {
        return new Result<>(null, error, false);
    }
    
    public boolean isSuccess() { return success; }
    public T getData() { return data; }
    public String getError() { return error; }
    
    // Functional transform
    public <R> Result<R> map(java.util.function.Function<T, R> transform) {
        if (isSuccess()) return Result.success(transform.apply(data));
        return Result.failure(error);
    }
}
```

---

## 56.6 สรุป Part 56

ในบทนี้คุณได้เรียนรู้:

✅ Clean Architecture (3 layers)  
✅ Dependency rule (outer → inner)  
✅ Domain Layer (entities, interfaces, use cases)  
✅ Data Layer (entities, DTOs, mappers, repository impl)  
✅ Presentation Layer (ViewModel with use cases)  
✅ UiState pattern  
✅ Result wrapper  
✅ LiveData transformation  

---

*[← Part 55: Play Store](./part-55-android-publish.md) | [Part 57: Advanced RecyclerView →](./part-57-android-advanced-recyclerview.md)*
