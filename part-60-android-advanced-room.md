# Part 60: Advanced Room Database
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 60.1 Room Database Migration

```java
// เมื่อเปลี่ยน schema ต้อง migrate หรือ fallback ถึงจะไม่ crash

// AppDatabase.java
@Database(
    entities = {User.class, Note.class, Tag.class},
    version = 3,   // เพิ่มขึ้นเมื่อ schema เปลี่ยน
    exportSchema = true
)
public abstract class AppDatabase extends RoomDatabase {
    
    public abstract UserDao userDao();
    public abstract NoteDao noteDao();
    public abstract TagDao tagDao();
    
    private static volatile AppDatabase INSTANCE;
    
    public static AppDatabase getInstance(Context context) {
        if (INSTANCE == null) {
            synchronized (AppDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(
                        context.getApplicationContext(),
                        AppDatabase.class,
                        "myapp.db"
                    )
                    .addMigrations(MIGRATION_1_2, MIGRATION_2_3)
                    .build();
                }
            }
        }
        return INSTANCE;
    }
    
    // Migration version 1 → 2: เพิ่ม column email
    static final Migration MIGRATION_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            database.execSQL(
                "ALTER TABLE users ADD COLUMN email TEXT NOT NULL DEFAULT ''");
        }
    };
    
    // Migration version 2 → 3: สร้าง table tags
    static final Migration MIGRATION_2_3 = new Migration(2, 3) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            database.execSQL(
                "CREATE TABLE tags (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL," +
                "name TEXT NOT NULL," +
                "color TEXT NOT NULL DEFAULT '#000000'" +
                ")");
            database.execSQL(
                "CREATE TABLE note_tags (" +
                "note_id INTEGER NOT NULL," +
                "tag_id INTEGER NOT NULL," +
                "PRIMARY KEY(note_id, tag_id)," +
                "FOREIGN KEY(note_id) REFERENCES notes(id) ON DELETE CASCADE," +
                "FOREIGN KEY(tag_id) REFERENCES tags(id) ON DELETE CASCADE" +
                ")");
        }
    };
}
```

---

## 60.2 Relations (One-to-Many, Many-to-Many)

```java
// One-to-Many: User has many Notes
@Entity(tableName = "notes")
public class NoteEntity {
    @PrimaryKey(autoGenerate = true)
    public int id;
    
    @ColumnInfo(name = "user_id")
    public int userId;  // Foreign key
    
    public String title;
    public String content;
    
    @ColumnInfo(name = "created_at")
    public long createdAt;
}

// UserWithNotes - one-to-many relation
public class UserWithNotes {
    @Embedded
    public UserEntity user;
    
    @Relation(
        parentColumn = "id",
        entityColumn = "user_id"
    )
    public List<NoteEntity> notes;
}

// UserDao
@Dao
public interface UserDao {
    @Transaction
    @Query("SELECT * FROM users WHERE id = :userId")
    LiveData<UserWithNotes> getUserWithNotes(int userId);
    
    @Transaction
    @Query("SELECT * FROM users")
    LiveData<List<UserWithNotes>> getAllUsersWithNotes();
}

// Many-to-Many: Note has many Tags, Tag has many Notes
@Entity(tableName = "tags")
public class TagEntity {
    @PrimaryKey(autoGenerate = true)
    public int id;
    public String name;
    public String color;
}

// Junction table
@Entity(
    tableName = "note_tags",
    primaryKeys = {"note_id", "tag_id"},
    foreignKeys = {
        @ForeignKey(entity = NoteEntity.class, parentColumns = "id",
            childColumns = "note_id", onDelete = ForeignKey.CASCADE),
        @ForeignKey(entity = TagEntity.class, parentColumns = "id",
            childColumns = "tag_id", onDelete = ForeignKey.CASCADE)
    }
)
public class NoteTagCrossRef {
    @ColumnInfo(name = "note_id") public int noteId;
    @ColumnInfo(name = "tag_id")  public int tagId;
}

// NoteWithTags
public class NoteWithTags {
    @Embedded
    public NoteEntity note;
    
    @Relation(
        parentColumn = "id",
        entity = TagEntity.class,
        entityColumn = "id",
        associateBy = @Junction(
            value = NoteTagCrossRef.class,
            parentColumn = "note_id",
            entityColumn = "tag_id"
        )
    )
    public List<TagEntity> tags;
}

// NoteDao with relations
@Dao
public interface NoteDao {
    @Transaction
    @Query("SELECT * FROM notes")
    LiveData<List<NoteWithTags>> getNotesWithTags();
    
    @Insert
    long insertNote(NoteEntity note);
    
    @Insert
    void insertNoteTagCrossRef(NoteTagCrossRef crossRef);
    
    @Transaction
    default void insertNoteWithTags(NoteEntity note, List<TagEntity> tags) {
        long noteId = insertNote(note);
        for (TagEntity tag : tags) {
            insertNoteTagCrossRef(new NoteTagCrossRef((int) noteId, tag.id));
        }
    }
}
```

---

## 60.3 Full-Text Search (FTS)

```java
// FTS4 entity for fast text search
@Fts4(contentEntity = NoteEntity.class)
@Entity(tableName = "notes_fts")
public class NoteFts {
    @ColumnInfo(name = "rowid")
    public int rowId;
    
    public String title;
    public String content;
}

// NoteDao with FTS
@Dao
public interface NoteDao {
    // Full-text search using FTS
    @Query("SELECT * FROM notes WHERE rowid IN " +
           "(SELECT rowid FROM notes_fts WHERE notes_fts MATCH :query)")
    LiveData<List<NoteEntity>> searchNotes(String query);
    
    // Snippet of matched text
    @Query("SELECT snippet(notes_fts, -1, '<b>', '</b>', '...', 5) AS snippet " +
           "FROM notes_fts WHERE notes_fts MATCH :query")
    List<String> getSearchSnippets(String query);
}

// Usage: search with wildcard
noteDao.searchNotes("android*");  // matches "android", "android studio", etc.
noteDao.searchNotes("\"java programming\"");  // exact phrase
```

---

## 60.4 Room with Coroutines (RxJava alternative)

```java
// Room supports Flowable (RxJava) for reactive queries
// build.gradle: implementation "androidx.room:room-rxjava3:2.6.1"

@Dao
public interface NoteDao {
    
    // Flowable = stream that emits on DB changes
    @Query("SELECT * FROM notes ORDER BY created_at DESC")
    Flowable<List<NoteEntity>> getAllNotesReactive();
    
    // Single = one-time async query
    @Query("SELECT * FROM notes WHERE id = :id")
    Single<NoteEntity> getNoteById(int id);
    
    // Completable = async insert/update/delete
    @Insert
    Completable insertAsync(NoteEntity note);
    
    @Delete
    Completable deleteAsync(NoteEntity note);
}

// Usage in ViewModel
public class NoteViewModel extends ViewModel {
    
    private final NoteDao noteDao;
    private final CompositeDisposable disposable = new CompositeDisposable();
    
    private final MutableLiveData<List<Note>> notesLiveData = new MutableLiveData<>();
    
    public void loadNotes() {
        disposable.add(
            noteDao.getAllNotesReactive()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(entities -> {
                    List<Note> notes = NoteMapper.toDomainList(entities);
                    notesLiveData.setValue(notes);
                }, throwable -> Log.e("ViewModel", "Error", throwable))
        );
    }
    
    @Override
    protected void onCleared() {
        disposable.clear();
    }
}
```

---

## 60.5 Database Backup & Export

```java
// DatabaseBackupHelper.java
public class DatabaseBackupHelper {
    
    private final Context context;
    private static final String DB_NAME = "myapp.db";
    
    public DatabaseBackupHelper(Context context) {
        this.context = context;
    }
    
    // Export database to Downloads folder
    public boolean exportDatabase() {
        try {
            // Close DB first
            AppDatabase.getInstance(context).close();
            
            File dbFile = context.getDatabasePath(DB_NAME);
            
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                // Android 10+ use MediaStore
                ContentValues values = new ContentValues();
                values.put(MediaStore.Downloads.DISPLAY_NAME, "backup_" + System.currentTimeMillis() + ".db");
                values.put(MediaStore.Downloads.MIME_TYPE, "application/octet-stream");
                
                Uri uri = context.getContentResolver()
                    .insert(MediaStore.Downloads.EXTERNAL_CONTENT_URI, values);
                if (uri == null) return false;
                
                try (OutputStream out = context.getContentResolver().openOutputStream(uri);
                     InputStream in = new FileInputStream(dbFile)) {
                    byte[] buffer = new byte[8192];
                    int read;
                    while ((read = in.read(buffer)) != -1) out.write(buffer, 0, read);
                }
            } else {
                File exportDir = Environment.getExternalStoragePublicDirectory(
                    Environment.DIRECTORY_DOWNLOADS);
                File exportFile = new File(exportDir, "backup_" + System.currentTimeMillis() + ".db");
                copyFile(dbFile, exportFile);
            }
            return true;
        } catch (Exception e) {
            Log.e("Backup", "Export failed", e);
            return false;
        }
    }
    
    private void copyFile(File src, File dst) throws IOException {
        try (FileChannel in = new FileInputStream(src).getChannel();
             FileChannel out = new FileOutputStream(dst).getChannel()) {
            out.transferFrom(in, 0, in.size());
        }
    }
}
```

---

## 60.6 สรุป Part 60

ในบทนี้คุณได้เรียนรู้:

✅ Room Database Migration (addMigrations, ALTER TABLE)  
✅ One-to-Many relation (@Relation)  
✅ Many-to-Many relation (Junction table, @Junction)  
✅ Full-Text Search with FTS4  
✅ Room with RxJava3 (Flowable, Single, Completable)  
✅ Database backup/export  

---

*[← Part 59: i18n](./part-59-android-i18n.md) | [Part 61: Multi-Module Architecture →](./part-61-android-multi-module.md)*
