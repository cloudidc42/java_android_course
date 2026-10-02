# Part 95: Database Migration Strategies
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 95.1 Room Migration Best Practices

```java
// AppDatabase.java - versioned migrations
@Database(
    entities = {
        UserEntity.class,
        ProductEntity.class,
        CartItemEntity.class,
        OrderEntity.class,
        OrderItemEntity.class,
        ReviewEntity.class
    },
    version = 6,
    exportSchema = true  // saves schema JSON for testing
)
public abstract class AppDatabase extends RoomDatabase {
    
    // V1 → V2: Add email column to users
    public static final Migration MIGRATION_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL(
                "ALTER TABLE users ADD COLUMN email TEXT NOT NULL DEFAULT ''");
        }
    };
    
    // V2 → V3: Add products table
    public static final Migration MIGRATION_2_3 = new Migration(2, 3) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL(
                "CREATE TABLE IF NOT EXISTS products (" +
                "id TEXT NOT NULL PRIMARY KEY, " +
                "name TEXT NOT NULL, " +
                "price REAL NOT NULL DEFAULT 0.0, " +
                "category TEXT NOT NULL DEFAULT '', " +
                "image_url TEXT, " +
                "in_stock INTEGER NOT NULL DEFAULT 1)");
        }
    };
    
    // V3 → V4: Add orders + order_items tables (multi-statement migration)
    public static final Migration MIGRATION_3_4 = new Migration(3, 4) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL(
                "CREATE TABLE IF NOT EXISTS orders (" +
                "id TEXT NOT NULL PRIMARY KEY, " +
                "user_id TEXT NOT NULL, " +
                "total REAL NOT NULL, " +
                "status TEXT NOT NULL DEFAULT 'pending', " +
                "created_at INTEGER NOT NULL)");
            
            database.execSQL(
                "CREATE TABLE IF NOT EXISTS order_items (" +
                "id TEXT NOT NULL PRIMARY KEY, " +
                "order_id TEXT NOT NULL, " +
                "product_id TEXT NOT NULL, " +
                "quantity INTEGER NOT NULL, " +
                "price REAL NOT NULL, " +
                "FOREIGN KEY(order_id) REFERENCES orders(id) ON DELETE CASCADE)");
            
            // Add index for foreign key
            database.execSQL(
                "CREATE INDEX idx_order_items_order_id ON order_items(order_id)");
        }
    };
    
    // V4 → V5: Rename column (SQLite doesn't support RENAME COLUMN before 3.25)
    public static final Migration MIGRATION_4_5 = new Migration(4, 5) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            // SQLite workaround for renaming column:
            // 1. Create new table with correct schema
            // 2. Copy data
            // 3. Drop old table
            // 4. Rename new table
            
            database.execSQL(
                "CREATE TABLE products_new (" +
                "id TEXT NOT NULL PRIMARY KEY, " +
                "name TEXT NOT NULL, " +
                "discount_price REAL NOT NULL DEFAULT 0.0, " +  // renamed from 'price'
                "original_price REAL NOT NULL DEFAULT 0.0, " +  // new column
                "category TEXT NOT NULL DEFAULT '', " +
                "image_url TEXT, " +
                "in_stock INTEGER NOT NULL DEFAULT 1)");
            
            // Copy existing data (price → discount_price, same value for original_price)
            database.execSQL(
                "INSERT INTO products_new " +
                "(id, name, discount_price, original_price, category, image_url, in_stock) " +
                "SELECT id, name, price, price, category, image_url, in_stock FROM products");
            
            database.execSQL("DROP TABLE products");
            database.execSQL("ALTER TABLE products_new RENAME TO products");
        }
    };
    
    // V5 → V6: Add reviews table + add rating column to products
    public static final Migration MIGRATION_5_6 = new Migration(5, 6) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL(
                "ALTER TABLE products ADD COLUMN rating REAL NOT NULL DEFAULT 0.0");
            database.execSQL(
                "ALTER TABLE products ADD COLUMN review_count INTEGER NOT NULL DEFAULT 0");
            
            database.execSQL(
                "CREATE TABLE IF NOT EXISTS reviews (" +
                "id TEXT NOT NULL PRIMARY KEY, " +
                "product_id TEXT NOT NULL, " +
                "user_id TEXT NOT NULL, " +
                "rating REAL NOT NULL, " +
                "comment TEXT NOT NULL DEFAULT '', " +
                "created_at INTEGER NOT NULL)");
            
            database.execSQL(
                "CREATE INDEX idx_reviews_product_id ON reviews(product_id)");
        }
    };
    
    // Register all migrations
    public static AppDatabase create(Context context) {
        return Room.databaseBuilder(context, AppDatabase.class, "shopapp.db")
            .addMigrations(
                MIGRATION_1_2,
                MIGRATION_2_3,
                MIGRATION_3_4,
                MIGRATION_4_5,
                MIGRATION_5_6)
            // Optional: skip migration (DEV only! destroys all data)
            // .fallbackToDestructiveMigrationFrom(1, 2)
            .build();
    }
    
    // DAOs
    public abstract UserDao    userDao();
    public abstract ProductDao productDao();
    public abstract OrderDao   orderDao();
    public abstract ReviewDao  reviewDao();
}
```

---

## 95.2 Test Migrations

```java
// MigrationTest.java
@RunWith(AndroidJUnit4.class)
public class MigrationTest {
    
    private static final String TEST_DB = "test-migration-db";
    
    @Rule
    public MigrationTestHelper helper = new MigrationTestHelper(
        InstrumentationRegistry.getInstrumentation(),
        AppDatabase.class.getCanonicalName(),
        new FrameworkSQLiteOpenHelperFactory());
    
    @Test
    public void migrate1To2() throws Exception {
        // Create version 1 database
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);
        
        // Insert data with V1 schema
        db.execSQL("INSERT INTO users (id, name) VALUES ('1', 'Alice')");
        db.close();
        
        // Apply migration
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2);
        
        // Verify: Alice still exists, email column has default value
        Cursor cursor = db.query("SELECT email FROM users WHERE id = '1'");
        assertTrue(cursor.moveToFirst());
        assertEquals("", cursor.getString(0));
        cursor.close();
    }
    
    @Test
    public void migrate4To5ProductsRenamed() throws Exception {
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 4);
        
        // Insert product with old 'price' column
        db.execSQL("INSERT INTO products (id, name, price, category, in_stock) " +
            "VALUES ('p1', 'Widget', 99.99, 'tools', 1)");
        db.close();
        
        db = helper.runMigrationsAndValidate(TEST_DB, 5, true, MIGRATION_4_5);
        
        // Verify: discount_price = 99.99, original_price = 99.99
        Cursor cursor = db.query(
            "SELECT discount_price, original_price FROM products WHERE id = 'p1'");
        assertTrue(cursor.moveToFirst());
        assertEquals(99.99, cursor.getDouble(0), 0.001);
        assertEquals(99.99, cursor.getDouble(1), 0.001);
        cursor.close();
    }
    
    @Test
    public void migrateAll() throws Exception {
        // Test full migration path from V1 to V6
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);
        db.close();
        
        helper.runMigrationsAndValidate(TEST_DB, 6, true,
            MIGRATION_1_2, MIGRATION_2_3, MIGRATION_3_4,
            MIGRATION_4_5, MIGRATION_5_6);
    }
}
```

---

## 95.3 สรุป Part 95

ในบทนี้คุณได้เรียนรู้:

✅ Room migration versioning  
✅ exportSchema = true (for testing)  
✅ ALTER TABLE ADD COLUMN  
✅ CREATE TABLE in migration  
✅ Rename column workaround (create/copy/drop/rename)  
✅ Add index in migration  
✅ fallbackToDestructiveMigrationFrom (dev only)  
✅ MigrationTestHelper for unit testing migrations  
✅ Test full migration path  

---

*[← Part 94: Error Handling](./part-94-error-handling.md) | [Part 96: Android App Architecture - Final Review →](./part-96-architecture-final.md)*
