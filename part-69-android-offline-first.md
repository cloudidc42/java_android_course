# Part 69: Offline-First Architecture
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 69.1 Offline-First คืออะไร

```
Traditional (Online-first):
App → Network (if fail → show error)

Offline-First:
App → Local DB (always) ↔ Network (sync in background)

ผู้ใช้:
- ใช้งานได้เสมอ ไม่มี "no connection" errors
- ข้อมูลแสดงทันที ไม่รอ network
- เมื่อกลับมา online → sync อัตโนมัติ
```

---

## 69.2 Single Source of Truth Pattern

```java
// Repository ใช้ Room เป็น single source of truth
// Network data → save to Room → Room → UI

// UserRepository.java
public class UserRepositoryImpl implements UserRepository {
    
    private final UserDao userDao;
    private final UserApiService apiService;
    private final NetworkConnectivityObserver connectivityObserver;
    
    @Inject
    public UserRepositoryImpl(UserDao userDao, UserApiService apiService,
            NetworkConnectivityObserver connectivityObserver) {
        this.userDao = userDao;
        this.apiService = apiService;
        this.connectivityObserver = connectivityObserver;
    }
    
    @Override
    public LiveData<List<User>> getUsers() {
        // Always return from Room (fast, offline)
        LiveData<List<UserEntity>> localData = userDao.getAllUsers();
        
        // Trigger network refresh
        refreshUsersFromNetwork();
        
        return Transformations.map(localData, UserMapper::toDomainList);
    }
    
    private void refreshUsersFromNetwork() {
        CompletableFuture.runAsync(() -> {
            try {
                Response<List<UserDto>> response = apiService.getUsers().execute();
                if (response.isSuccessful() && response.body() != null) {
                    List<UserEntity> entities = response.body().stream()
                        .map(UserMapper::toEntity)
                        .collect(Collectors.toList());
                    // Replace all local data
                    userDao.deleteAll();
                    userDao.insertAll(entities);
                }
            } catch (Exception e) {
                // Network failed, keep local data - no crash
                Log.d("Repo", "Using cached data: " + e.getMessage());
            }
        });
    }
}
```

---

## 69.3 Sync Queue (Pending Operations)

```java
// PendingOperation.java - track unsynced changes
@Entity(tableName = "pending_operations")
public class PendingOperation {
    
    @PrimaryKey(autoGenerate = true)
    public int id;
    
    @ColumnInfo(name = "operation_type")
    public String operationType;  // CREATE, UPDATE, DELETE
    
    @ColumnInfo(name = "entity_type")
    public String entityType;     // "user", "note", "task"
    
    @ColumnInfo(name = "entity_id")
    public String entityId;
    
    @ColumnInfo(name = "payload")
    public String payload;         // JSON of the entity
    
    @ColumnInfo(name = "created_at")
    public long createdAt;
    
    @ColumnInfo(name = "retry_count")
    public int retryCount = 0;
}

// PendingOperationDao.java
@Dao
public interface PendingOperationDao {
    @Query("SELECT * FROM pending_operations ORDER BY created_at ASC")
    List<PendingOperation> getAllPending();
    
    @Insert
    void insert(PendingOperation op);
    
    @Delete
    void delete(PendingOperation op);
    
    @Query("UPDATE pending_operations SET retry_count = retry_count + 1 WHERE id = :id")
    void incrementRetryCount(int id);
    
    @Query("DELETE FROM pending_operations WHERE retry_count > 5")
    void deleteFailedOperations();
}

// SyncManager.java
public class SyncManager {
    
    private final PendingOperationDao pendingDao;
    private final UserApiService apiService;
    private final Gson gson;
    
    @Inject
    public SyncManager(PendingOperationDao pendingDao, UserApiService apiService) {
        this.pendingDao = pendingDao;
        this.apiService = apiService;
        this.gson = new Gson();
    }
    
    // Call when network is available
    public void syncPendingOperations() {
        List<PendingOperation> pending = pendingDao.getAllPending();
        
        for (PendingOperation op : pending) {
            try {
                syncOperation(op);
                pendingDao.delete(op);  // Success → remove from queue
            } catch (Exception e) {
                pendingDao.incrementRetryCount(op.id);
                if (op.retryCount >= 5) {
                    // Failed too many times → log and remove
                    Log.e("Sync", "Giving up on operation: " + op.id);
                    pendingDao.delete(op);
                }
            }
        }
    }
    
    private void syncOperation(PendingOperation op) throws Exception {
        if ("user".equals(op.entityType)) {
            switch (op.operationType) {
                case "CREATE":
                    UserDto dto = gson.fromJson(op.payload, UserDto.class);
                    apiService.createUser(dto).execute();
                    break;
                case "UPDATE":
                    UserDto updateDto = gson.fromJson(op.payload, UserDto.class);
                    apiService.updateUser(op.entityId, updateDto).execute();
                    break;
                case "DELETE":
                    apiService.deleteUser(op.entityId).execute();
                    break;
            }
        }
        // Add more entity types as needed
    }
    
    // Queue an operation (when offline or as backup)
    public void queueOperation(String type, String entityType, String entityId, Object payload) {
        PendingOperation op = new PendingOperation();
        op.operationType = type;
        op.entityType = entityType;
        op.entityId = entityId;
        op.payload = gson.toJson(payload);
        op.createdAt = System.currentTimeMillis();
        pendingDao.insert(op);
    }
}
```

---

## 69.4 Network Connectivity Observer

```java
// NetworkConnectivityObserver.java
public class NetworkConnectivityObserver {
    
    private final ConnectivityManager connectivityManager;
    private final MutableLiveData<Boolean> isConnected = new MutableLiveData<>();
    
    @Inject
    public NetworkConnectivityObserver(@ApplicationContext Context context) {
        connectivityManager = (ConnectivityManager)
            context.getSystemService(Context.CONNECTIVITY_SERVICE);
        
        // Register network callback
        NetworkRequest request = new NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .build();
        
        connectivityManager.registerNetworkCallback(request,
            new ConnectivityManager.NetworkCallback() {
                @Override
                public void onAvailable(Network network) {
                    isConnected.postValue(true);
                }
                
                @Override
                public void onLost(Network network) {
                    // Check if any other network available
                    isConnected.postValue(isCurrentlyConnected());
                }
            });
        
        // Initial state
        isConnected.setValue(isCurrentlyConnected());
    }
    
    public boolean isCurrentlyConnected() {
        Network network = connectivityManager.getActiveNetwork();
        if (network == null) return false;
        NetworkCapabilities caps = connectivityManager.getNetworkCapabilities(network);
        return caps != null && caps.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET);
    }
    
    public LiveData<Boolean> observeConnection() { return isConnected; }
}

// Auto-sync when connection restored
// In Application or MainActivity
connectivityObserver.observeConnection().observe(this, isOnline -> {
    if (Boolean.TRUE.equals(isOnline)) {
        syncManager.syncPendingOperations();
    }
});
```

---

## 69.5 SyncWorker (periodic background sync)

```java
// SyncWorker.java
public class SyncWorker extends Worker {
    
    private final SyncManager syncManager;
    
    public SyncWorker(Context context, WorkerParameters params) {
        super(context, params);
        // Manual injection (or use HiltWorker)
        syncManager = ((MyApplication) context.getApplicationContext())
            .getSyncManager();
    }
    
    @Override
    public Result doWork() {
        try {
            syncManager.syncPendingOperations();
            return Result.success();
        } catch (Exception e) {
            return Result.retry();
        }
    }
    
    // Schedule periodic sync
    public static void schedulePeriodicSync(Context context) {
        PeriodicWorkRequest syncWork = new PeriodicWorkRequest.Builder(
            SyncWorker.class, 1, TimeUnit.HOURS)
            .setConstraints(new Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build())
            .build();
        
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(
            "periodic_sync",
            ExistingPeriodicWorkPolicy.KEEP,
            syncWork);
    }
}
```

---

## 69.6 สรุป Part 69

ในบทนี้คุณได้เรียนรู้:

✅ Offline-First concept  
✅ Single Source of Truth (Room → UI)  
✅ Background network refresh  
✅ Pending operations queue (sync queue)  
✅ SyncManager (syncPendingOperations)  
✅ NetworkConnectivityObserver (LiveData)  
✅ Auto-sync on reconnect  
✅ SyncWorker (periodic WorkManager)  

---

*[← Part 68: Advanced Notifications](./part-68-android-advanced-notifications.md) | [Part 70: Java Advanced Concurrency →](./part-70-java-advanced-concurrency.md)*
