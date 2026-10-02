# Part 36: Room Database
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 36.1 Room คืออะไร

Room คือ SQLite wrapper library ของ Android Jetpack  
ทำให้เขียน database code ง่าย และ type-safe

**Components:**
- `@Entity` - ตาราง database (class)
- `@Dao` - Data Access Object (interface)
- `@Database` - database definition (abstract class)

---

## 36.2 Dependencies

```groovy
// build.gradle (app)
dependencies {
    def room_version = "2.6.0"
    
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    
    // Optional: RxJava / LiveData / Coroutines
    implementation "androidx.room:room-rxjava3:$room_version"
    
    // ViewModel + LiveData
    implementation "androidx.lifecycle:lifecycle-viewmodel:2.6.2"
    implementation "androidx.lifecycle:lifecycle-livedata:2.6.2"
}
```

---

## 36.3 Entity

```java
// model/Note.java
package com.example.myapp.model;

import androidx.room.*;
import java.util.Date;

@Entity(tableName = "notes")
public class Note {
    
    @PrimaryKey(autoGenerate = true)
    private int id;
    
    @ColumnInfo(name = "title")
    private String title;
    
    @ColumnInfo(name = "content")
    private String content;
    
    @ColumnInfo(name = "created_at")
    private long createdAt;
    
    @ColumnInfo(name = "is_favorite")
    private boolean isFavorite;
    
    // Constructor
    public Note(String title, String content) {
        this.title = title;
        this.content = content;
        this.createdAt = System.currentTimeMillis();
        this.isFavorite = false;
    }
    
    // Getters and Setters (required by Room)
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public long getCreatedAt() { return createdAt; }
    public void setCreatedAt(long createdAt) { this.createdAt = createdAt; }
    
    public boolean isFavorite() { return isFavorite; }
    public void setFavorite(boolean favorite) { isFavorite = favorite; }
}
```

---

## 36.4 DAO

```java
// dao/NoteDao.java
package com.example.myapp.dao;

import androidx.lifecycle.LiveData;
import androidx.room.*;
import com.example.myapp.model.Note;
import java.util.List;

@Dao
public interface NoteDao {
    
    // INSERT
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    long insert(Note note);
    
    @Insert
    void insertAll(List<Note> notes);
    
    // UPDATE
    @Update
    void update(Note note);
    
    // DELETE
    @Delete
    void delete(Note note);
    
    @Query("DELETE FROM notes WHERE id = :id")
    void deleteById(int id);
    
    @Query("DELETE FROM notes")
    void deleteAll();
    
    // SELECT - returns LiveData (auto-updates UI)
    @Query("SELECT * FROM notes ORDER BY created_at DESC")
    LiveData<List<Note>> getAllNotes();
    
    @Query("SELECT * FROM notes WHERE id = :id")
    LiveData<Note> getNoteById(int id);
    
    @Query("SELECT * FROM notes WHERE title LIKE :search OR content LIKE :search ORDER BY created_at DESC")
    LiveData<List<Note>> searchNotes(String search);
    
    @Query("SELECT * FROM notes WHERE is_favorite = 1 ORDER BY created_at DESC")
    LiveData<List<Note>> getFavoriteNotes();
    
    @Query("SELECT COUNT(*) FROM notes")
    int getCount();
    
    // Synchronous (use only on background thread)
    @Query("SELECT * FROM notes ORDER BY created_at DESC")
    List<Note> getAllNotesSync();
}
```

---

## 36.5 Database

```java
// database/NoteDatabase.java
package com.example.myapp.database;

import android.content.Context;
import androidx.room.*;
import androidx.sqlite.db.SupportSQLiteDatabase;
import com.example.myapp.dao.NoteDao;
import com.example.myapp.model.Note;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Database(entities = {Note.class}, version = 1, exportSchema = false)
public abstract class NoteDatabase extends RoomDatabase {
    
    public abstract NoteDao noteDao();
    
    private static volatile NoteDatabase INSTANCE;
    
    // Thread pool for database operations
    public static final ExecutorService databaseWriteExecutor =
        Executors.newFixedThreadPool(4);
    
    // Singleton pattern
    public static NoteDatabase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (NoteDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(
                        context.getApplicationContext(),
                        NoteDatabase.class,
                        "note_database")
                        .fallbackToDestructiveMigration()  // for dev only
                        .addCallback(sRoomDatabaseCallback)
                        .build();
                }
            }
        }
        return INSTANCE;
    }
    
    // Prepopulate database
    private static final RoomDatabase.Callback sRoomDatabaseCallback =
        new RoomDatabase.Callback() {
            @Override
            public void onCreate(@NonNull SupportSQLiteDatabase db) {
                super.onCreate(db);
                
                databaseWriteExecutor.execute(() -> {
                    NoteDao dao = INSTANCE.noteDao();
                    dao.deleteAll();
                    
                    dao.insert(new Note("ยินดีต้อนรับ", "นี่คือโน้ตแรกของคุณ!"));
                    dao.insert(new Note("Android Development", "เรียนรู้ Room Database"));
                    dao.insert(new Note("Java Tips", "ใช้ ExecutorService สำหรับ background thread"));
                });
            }
        };
}
```

---

## 36.6 Repository

```java
// repository/NoteRepository.java
package com.example.myapp.repository;

import android.app.Application;
import androidx.lifecycle.LiveData;
import com.example.myapp.dao.NoteDao;
import com.example.myapp.database.NoteDatabase;
import com.example.myapp.model.Note;
import java.util.List;
import java.util.concurrent.ExecutorService;

public class NoteRepository {
    
    private NoteDao noteDao;
    private LiveData<List<Note>> allNotes;
    private ExecutorService executor;
    
    public NoteRepository(Application application) {
        NoteDatabase db = NoteDatabase.getDatabase(application);
        noteDao = db.noteDao();
        allNotes = noteDao.getAllNotes();
        executor = NoteDatabase.databaseWriteExecutor;
    }
    
    // LiveData (observed in UI thread)
    public LiveData<List<Note>> getAllNotes() { return allNotes; }
    
    public LiveData<List<Note>> searchNotes(String query) {
        return noteDao.searchNotes("%" + query + "%");
    }
    
    public LiveData<List<Note>> getFavoriteNotes() {
        return noteDao.getFavoriteNotes();
    }
    
    // Write operations (must be on background thread)
    public void insert(Note note) {
        executor.execute(() -> noteDao.insert(note));
    }
    
    public void update(Note note) {
        executor.execute(() -> noteDao.update(note));
    }
    
    public void delete(Note note) {
        executor.execute(() -> noteDao.delete(note));
    }
    
    public void deleteById(int id) {
        executor.execute(() -> noteDao.deleteById(id));
    }
}
```

---

## 36.7 ViewModel

```java
// viewmodel/NoteViewModel.java
package com.example.myapp.viewmodel;

import android.app.Application;
import androidx.lifecycle.*;
import com.example.myapp.model.Note;
import com.example.myapp.repository.NoteRepository;
import java.util.List;

public class NoteViewModel extends AndroidViewModel {
    
    private NoteRepository repository;
    private LiveData<List<Note>> allNotes;
    
    public NoteViewModel(Application application) {
        super(application);
        repository = new NoteRepository(application);
        allNotes = repository.getAllNotes();
    }
    
    public LiveData<List<Note>> getAllNotes() { return allNotes; }
    
    public LiveData<List<Note>> searchNotes(String query) {
        return repository.searchNotes(query);
    }
    
    public void insert(Note note) { repository.insert(note); }
    public void update(Note note) { repository.update(note); }
    public void delete(Note note) { repository.delete(note); }
}
```

---

## 36.8 Activity using Room + ViewModel

```java
// NoteListActivity.java
package com.example.myapp;

import android.content.Intent;
import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.example.myapp.adapter.NoteAdapter;
import com.example.myapp.model.Note;
import com.example.myapp.viewmodel.NoteViewModel;
import com.google.android.material.floatingactionbutton.FloatingActionButton;

public class NoteListActivity extends AppCompatActivity {

    private NoteViewModel viewModel;
    private NoteAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_note_list);

        RecyclerView recyclerView = findViewById(R.id.recyclerView);
        FloatingActionButton fab = findViewById(R.id.fab);

        // Setup RecyclerView
        adapter = new NoteAdapter();
        recyclerView.setAdapter(adapter);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        // Get ViewModel
        viewModel = new ViewModelProvider(this).get(NoteViewModel.class);

        // Observe LiveData - auto updates when data changes
        viewModel.getAllNotes().observe(this, notes -> {
            adapter.submitList(notes);
        });

        // Click: open detail
        adapter.setOnNoteClickListener(note -> {
            Intent intent = new Intent(this, NoteDetailActivity.class);
            intent.putExtra("noteId", note.getId());
            startActivity(intent);
        });

        // Long click: delete
        adapter.setOnNoteLongClickListener(note -> {
            new androidx.appcompat.app.AlertDialog.Builder(this)
                .setTitle("ลบโน้ต")
                .setMessage("ต้องการลบ \"" + note.getTitle() + "\" หรือไม่?")
                .setPositiveButton("ลบ", (d, w) -> viewModel.delete(note))
                .setNegativeButton("ยกเลิก", null)
                .show();
        });

        // FAB: create new note
        fab.setOnClickListener(v -> {
            startActivity(new Intent(this, AddNoteActivity.class));
        });
    }
}
```

```java
// AddNoteActivity.java
public class AddNoteActivity extends AppCompatActivity {

    private EditText etTitle, etContent;
    private NoteViewModel viewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_note);

        etTitle   = findViewById(R.id.etTitle);
        etContent = findViewById(R.id.etContent);

        viewModel = new ViewModelProvider(this).get(NoteViewModel.class);

        Button btnSave = findViewById(R.id.btnSave);
        btnSave.setOnClickListener(v -> {
            String title   = etTitle.getText().toString().trim();
            String content = etContent.getText().toString().trim();

            if (title.isEmpty()) {
                etTitle.setError("กรุณาใส่หัวข้อ");
                return;
            }

            Note note = new Note(title, content);
            viewModel.insert(note);

            Toast.makeText(this, "บันทึกแล้ว", Toast.LENGTH_SHORT).show();
            finish();
        });
    }
}
```

---

## 36.9 Database Migration

```java
// เพิ่ม column ใหม่ใน version 2
@Database(entities = {Note.class}, version = 2, exportSchema = false)
public abstract class NoteDatabase extends RoomDatabase {
    
    static final Migration MIGRATION_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            // Add new column
            database.execSQL(
                "ALTER TABLE notes ADD COLUMN priority INTEGER NOT NULL DEFAULT 0");
        }
    };
    
    public static NoteDatabase getDatabase(Context context) {
        if (INSTANCE == null) {
            INSTANCE = Room.databaseBuilder(
                context.getApplicationContext(),
                NoteDatabase.class,
                "note_database")
                .addMigrations(MIGRATION_1_2)  // Add migration
                .build();
        }
        return INSTANCE;
    }
}
```

---

## 36.10 สรุป Part 36

ในบทนี้คุณได้เรียนรู้:

✅ Room Database components (@Entity, @Dao, @Database)  
✅ CRUD operations  
✅ LiveData (auto-update UI)  
✅ Repository pattern  
✅ ViewModel (survive configuration changes)  
✅ Background thread (ExecutorService)  
✅ Database migration  
✅ Full Note app example  

---

*[← Part 35: Fragments](./part-35-android-fragments.md) | [Part 37: Retrofit & API →](./part-37-android-retrofit.md)*
