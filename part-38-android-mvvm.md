# Part 38: MVVM Architecture
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 38.1 MVVM คืออะไร

**MVVM = Model - View - ViewModel**

```
View (Activity/Fragment)
    ↕ observes LiveData
ViewModel
    ↕ calls
Repository
    ↕ uses
Model (Room DB + Retrofit API)
```

**ข้อดีของ MVVM:**
- Separation of concerns
- ViewModel survive rotation
- Testable
- LiveData auto-update UI

---

## 38.2 Full MVVM Note App

**โครงสร้าง Project:**
```
app/src/main/java/com/example/noteapp/
├── model/
│   └── Note.java               (@Entity)
├── dao/
│   └── NoteDao.java            (@Dao)
├── database/
│   └── NoteDatabase.java       (@Database)
├── repository/
│   └── NoteRepository.java     (data layer)
├── viewmodel/
│   └── NoteViewModel.java      (business logic)
├── adapter/
│   └── NoteAdapter.java        (RecyclerView)
├── ui/
│   ├── MainActivity.java       (note list)
│   ├── AddEditNoteActivity.java (add/edit)
│   └── NoteDetailActivity.java (view note)
└── util/
    └── DateUtils.java
```

---

## 38.3 Model

```java
// model/Note.java
package com.example.noteapp.model;

import androidx.room.*;

@Entity(tableName = "notes",
    indices = {@Index(value = "title")})
public class Note {
    
    @PrimaryKey(autoGenerate = true)
    private int id;
    
    @ColumnInfo(name = "title")
    private String title;
    
    @ColumnInfo(name = "content")
    private String content;
    
    @ColumnInfo(name = "category")
    private String category;  // "personal", "work", "study"
    
    @ColumnInfo(name = "color")
    private String color;  // hex color
    
    @ColumnInfo(name = "is_pinned")
    private boolean isPinned;
    
    @ColumnInfo(name = "created_at")
    private long createdAt;
    
    @ColumnInfo(name = "updated_at")
    private long updatedAt;
    
    public Note(String title, String content, String category, String color) {
        this.title = title;
        this.content = content;
        this.category = category;
        this.color = color;
        this.isPinned = false;
        this.createdAt = System.currentTimeMillis();
        this.updatedAt = System.currentTimeMillis();
    }
    
    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    
    public String getColor() { return color; }
    public void setColor(String color) { this.color = color; }
    
    public boolean isPinned() { return isPinned; }
    public void setPinned(boolean pinned) { isPinned = pinned; }
    
    public long getCreatedAt() { return createdAt; }
    public void setCreatedAt(long createdAt) { this.createdAt = createdAt; }
    
    public long getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(long updatedAt) { this.updatedAt = updatedAt; }
}
```

---

## 38.4 DAO

```java
// dao/NoteDao.java
package com.example.noteapp.dao;

import androidx.lifecycle.LiveData;
import androidx.room.*;
import com.example.noteapp.model.Note;
import java.util.List;

@Dao
public interface NoteDao {
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    long insert(Note note);
    
    @Update
    void update(Note note);
    
    @Delete
    void delete(Note note);
    
    @Query("SELECT * FROM notes ORDER BY is_pinned DESC, updated_at DESC")
    LiveData<List<Note>> getAllNotes();
    
    @Query("SELECT * FROM notes WHERE id = :id")
    LiveData<Note> getNoteById(int id);
    
    @Query("SELECT * FROM notes WHERE category = :category ORDER BY updated_at DESC")
    LiveData<List<Note>> getNotesByCategory(String category);
    
    @Query("SELECT * FROM notes WHERE title LIKE :query OR content LIKE :query ORDER BY updated_at DESC")
    LiveData<List<Note>> searchNotes(String query);
    
    @Query("SELECT COUNT(*) FROM notes")
    LiveData<Integer> getNoteCount();
    
    @Query("UPDATE notes SET is_pinned = :pinned WHERE id = :id")
    void setPinned(int id, boolean pinned);
}
```

---

## 38.5 Repository

```java
// repository/NoteRepository.java
package com.example.noteapp.repository;

import android.app.Application;
import androidx.lifecycle.LiveData;
import com.example.noteapp.dao.NoteDao;
import com.example.noteapp.database.NoteDatabase;
import com.example.noteapp.model.Note;
import java.util.List;
import java.util.concurrent.Future;
import java.util.concurrent.ExecutorService;

public class NoteRepository {
    
    private final NoteDao noteDao;
    private final ExecutorService executor;
    
    public NoteRepository(Application app) {
        NoteDatabase db = NoteDatabase.getDatabase(app);
        noteDao = db.noteDao();
        executor = NoteDatabase.databaseWriteExecutor;
    }
    
    // Queries (return LiveData - read on UI thread)
    public LiveData<List<Note>> getAllNotes() {
        return noteDao.getAllNotes();
    }
    
    public LiveData<Note> getNoteById(int id) {
        return noteDao.getNoteById(id);
    }
    
    public LiveData<List<Note>> getNotesByCategory(String category) {
        return noteDao.getNotesByCategory(category);
    }
    
    public LiveData<List<Note>> searchNotes(String query) {
        return noteDao.searchNotes("%" + query + "%");
    }
    
    public LiveData<Integer> getNoteCount() {
        return noteDao.getNoteCount();
    }
    
    // Writes (must run on background thread)
    public void insert(Note note, InsertCallback callback) {
        executor.execute(() -> {
            long id = noteDao.insert(note);
            if (callback != null) callback.onInserted((int) id);
        });
    }
    
    public void update(Note note) {
        executor.execute(() -> noteDao.update(note));
    }
    
    public void delete(Note note) {
        executor.execute(() -> noteDao.delete(note));
    }
    
    public void setPinned(int id, boolean pinned) {
        executor.execute(() -> noteDao.setPinned(id, pinned));
    }
    
    public interface InsertCallback {
        void onInserted(int newId);
    }
}
```

---

## 38.6 ViewModel

```java
// viewmodel/NoteViewModel.java
package com.example.noteapp.viewmodel;

import android.app.Application;
import androidx.lifecycle.*;
import com.example.noteapp.model.Note;
import com.example.noteapp.repository.NoteRepository;
import java.util.List;

public class NoteViewModel extends AndroidViewModel {
    
    private final NoteRepository repository;
    
    // Current filter
    private final MutableLiveData<String> searchQuery = new MutableLiveData<>("");
    private final MutableLiveData<String> selectedCategory = new MutableLiveData<>("all");
    
    // Notes based on filter
    private LiveData<List<Note>> notes;
    
    // Insert result
    private final MutableLiveData<Integer> newNoteId = new MutableLiveData<>();
    
    public NoteViewModel(Application app) {
        super(app);
        repository = new NoteRepository(app);
        notes = repository.getAllNotes();
    }
    
    public LiveData<List<Note>> getNotes() { return notes; }
    
    public LiveData<Note> getNoteById(int id) {
        return repository.getNoteById(id);
    }
    
    public LiveData<Integer> getNoteCount() {
        return repository.getNoteCount();
    }
    
    public LiveData<Integer> getNewNoteId() { return newNoteId; }
    
    public void insert(Note note) {
        repository.insert(note, id -> newNoteId.postValue(id));
    }
    
    public void update(Note note) {
        note.setUpdatedAt(System.currentTimeMillis());
        repository.update(note);
    }
    
    public void delete(Note note) {
        repository.delete(note);
    }
    
    public void togglePin(Note note) {
        repository.setPinned(note.getId(), !note.isPinned());
    }
    
    public void search(String query) {
        if (query == null || query.isEmpty()) {
            notes = repository.getAllNotes();
        } else {
            notes = repository.searchNotes(query);
        }
    }
    
    public void filterByCategory(String category) {
        if ("all".equals(category)) {
            notes = repository.getAllNotes();
        } else {
            notes = repository.getNotesByCategory(category);
        }
    }
}
```

---

## 38.7 View - MainActivity

```java
// ui/MainActivity.java
package com.example.noteapp.ui;

import android.content.Intent;
import android.os.Bundle;
import android.text.TextWatcher;
import android.text.Editable;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.*;
import androidx.recyclerview.widget.*;
import com.example.noteapp.R;
import com.example.noteapp.adapter.NoteAdapter;
import com.example.noteapp.model.Note;
import com.example.noteapp.viewmodel.NoteViewModel;
import com.google.android.material.floatingactionbutton.FloatingActionButton;

public class MainActivity extends AppCompatActivity {

    private NoteViewModel viewModel;
    private NoteAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Setup UI
        RecyclerView recyclerView = findViewById(R.id.recyclerView);
        EditText etSearch        = findViewById(R.id.etSearch);
        FloatingActionButton fab = findViewById(R.id.fab);
        TextView tvCount         = findViewById(R.id.tvCount);

        // RecyclerView
        StaggeredGridLayoutManager layoutManager =
            new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL);
        recyclerView.setLayoutManager(layoutManager);
        
        adapter = new NoteAdapter();
        recyclerView.setAdapter(adapter);

        // ViewModel
        viewModel = new ViewModelProvider(this).get(NoteViewModel.class);

        // Observe notes
        viewModel.getNotes().observe(this, notes -> {
            adapter.submitList(notes);
        });

        // Observe count
        viewModel.getNoteCount().observe(this, count -> {
            tvCount.setText(count + " โน้ต");
        });

        // Click note
        adapter.setOnNoteClickListener(note -> {
            Intent intent = new Intent(this, NoteDetailActivity.class);
            intent.putExtra("noteId", note.getId());
            startActivity(intent);
        });

        // Long click: pin/unpin
        adapter.setOnNoteLongClickListener(note -> {
            viewModel.togglePin(note);
            String msg = note.isPinned() ? "เลิกปักหมุดแล้ว" : "ปักหมุดแล้ว";
            Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
        });

        // FAB: add note
        fab.setOnClickListener(v -> {
            startActivity(new Intent(this, AddEditNoteActivity.class));
        });

        // Search
        etSearch.addTextChangedListener(new TextWatcher() {
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                viewModel.search(s.toString());
                // Re-observe
                viewModel.getNotes().observe(MainActivity.this, notes ->
                    adapter.submitList(notes));
            }
            @Override public void beforeTextChanged(CharSequence s, int a, int b, int c) {}
            @Override public void afterTextChanged(Editable s) {}
        });
    }
}
```

---

## 38.8 View - AddEditNoteActivity

```java
// ui/AddEditNoteActivity.java
public class AddEditNoteActivity extends AppCompatActivity {

    private NoteViewModel viewModel;
    private EditText etTitle, etContent;
    private Note existingNote;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_edit_note);

        etTitle   = findViewById(R.id.etTitle);
        etContent = findViewById(R.id.etContent);
        Button btnSave = findViewById(R.id.btnSave);

        viewModel = new ViewModelProvider(this).get(NoteViewModel.class);

        // Edit mode: load existing note
        int noteId = getIntent().getIntExtra("noteId", -1);
        if (noteId != -1) {
            viewModel.getNoteById(noteId).observe(this, note -> {
                if (note != null) {
                    existingNote = note;
                    etTitle.setText(note.getTitle());
                    etContent.setText(note.getContent());
                }
            });
        }

        btnSave.setOnClickListener(v -> saveNote());
    }

    private void saveNote() {
        String title   = etTitle.getText().toString().trim();
        String content = etContent.getText().toString().trim();

        if (title.isEmpty()) {
            etTitle.setError("กรุณาใส่หัวข้อ");
            return;
        }

        if (existingNote == null) {
            // New note
            Note note = new Note(title, content, "personal", "#FFEB3B");
            viewModel.insert(note);
        } else {
            // Update
            existingNote.setTitle(title);
            existingNote.setContent(content);
            viewModel.update(existingNote);
        }

        finish();
    }
}
```

---

## 38.9 สรุป Part 38

ในบทนี้คุณได้เรียนรู้:

✅ MVVM architecture pattern  
✅ Model → Repository → ViewModel → View flow  
✅ Complete Note app with MVVM  
✅ LiveData observing  
✅ ViewModel (survive rotation)  
✅ MutableLiveData for UI events  
✅ Search and filter  
✅ Add/Edit note with existing data  

---

*[← Part 37: Retrofit](./part-37-android-retrofit.md) | [Part 39: Android SharedPreferences & Settings →](./part-39-android-preferences.md)*
