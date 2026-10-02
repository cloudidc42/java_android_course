# Part 51: Testing Android Apps
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 51.1 การทดสอบใน Android

```
Unit Tests (test/)          - ทดสอบ logic บน JVM (เร็ว)
Instrumented Tests (androidTest/) - ทดสอบบน Device/Emulator (ช้า)

ประเภท:
1. Unit Test     - ViewModel, Repository, utility classes
2. Integration   - ViewModel + Room, Retrofit mocks  
3. UI Test       - Espresso (Activity/Fragment UI tests)
4. Screenshot    - Paparazzi, Screengrab
```

---

## 51.2 Unit Testing - ViewModel

```java
// test/viewmodel/NoteViewModelTest.java
package com.example.app.viewmodel;

import androidx.arch.core.executor.testing.InstantTaskExecutorRule;
import org.junit.*;
import org.junit.runner.RunWith;
import org.mockito.*;
import org.mockito.junit.MockitoJUnitRunner;
import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class NoteViewModelTest {
    
    @Rule
    public InstantTaskExecutorRule instantTaskExecutorRule = new InstantTaskExecutorRule();
    
    @Mock
    private NoteRepository noteRepository;
    
    private NoteViewModel viewModel;
    
    @Before
    public void setup() {
        MutableLiveData<List<Note>> notesLiveData = new MutableLiveData<>();
        notesLiveData.setValue(createSampleNotes());
        when(noteRepository.getAllNotes()).thenReturn(notesLiveData);
        
        viewModel = new NoteViewModel(noteRepository);
    }
    
    @Test
    public void testGetNotes_returnsNotes() {
        List<Note> notes = LiveDataTestUtil.getOrAwaitValue(viewModel.getNotes());
        
        assertNotNull(notes);
        assertEquals(3, notes.size());
        assertEquals("Note 1", notes.get(0).getTitle());
    }
    
    @Test
    public void testInsert_callsRepository() {
        Note note = new Note("Test", "Content", "personal", "#FFFFFF");
        
        viewModel.insert(note);
        
        verify(noteRepository).insert(eq(note), any());
    }
    
    @Test
    public void testDelete_callsRepository() {
        Note note = createSampleNotes().get(0);
        
        viewModel.delete(note);
        
        verify(noteRepository).delete(note);
    }
    
    @Test
    public void testSearch_filtersNotes() {
        MutableLiveData<List<Note>> searchResults = new MutableLiveData<>();
        searchResults.setValue(createSampleNotes().subList(0, 1));
        when(noteRepository.searchNotes("Note 1")).thenReturn(searchResults);
        
        viewModel.search("Note 1");
        List<Note> result = LiveDataTestUtil.getOrAwaitValue(viewModel.getNotes());
        
        assertEquals(1, result.size());
        assertEquals("Note 1", result.get(0).getTitle());
    }
    
    private List<Note> createSampleNotes() {
        List<Note> notes = new ArrayList<>();
        notes.add(new Note("Note 1", "Content 1", "personal", "#FFEB3B"));
        notes.add(new Note("Note 2", "Content 2", "work", "#03A9F4"));
        notes.add(new Note("Note 3", "Content 3", "study", "#8BC34A"));
        return notes;
    }
}
```

---

## 51.3 LiveData Test Util

```java
// test/util/LiveDataTestUtil.java
public class LiveDataTestUtil {
    
    public static <T> T getOrAwaitValue(final LiveData<T> liveData) 
            throws InterruptedException {
        
        final Object[] data = new Object[1];
        final CountDownLatch latch = new CountDownLatch(1);
        
        Observer<T> observer = new Observer<T>() {
            @Override
            public void onChanged(T t) {
                data[0] = t;
                latch.countDown();
                liveData.removeObserver(this);
            }
        };
        
        liveData.observeForever(observer);
        
        // Wait max 2 seconds
        if (!latch.await(2, TimeUnit.SECONDS)) {
            throw new RuntimeException("LiveData value was never set");
        }
        
        @SuppressWarnings("unchecked")
        T result = (T) data[0];
        return result;
    }
}
```

---

## 51.4 Repository Unit Test (with mock DAO)

```java
// test/repository/NoteRepositoryTest.java
@RunWith(MockitoJUnitRunner.class)
public class NoteRepositoryTest {
    
    @Mock
    private NoteDao noteDao;
    
    @Mock
    private ExecutorService executor;
    
    private NoteRepository repository;
    
    @Before
    public void setup() {
        MutableLiveData<List<Note>> liveData = new MutableLiveData<>();
        liveData.setValue(Collections.emptyList());
        when(noteDao.getAllNotes()).thenReturn(liveData);
        
        repository = new NoteRepository(noteDao, executor);
    }
    
    @Test
    public void testInsert_executesOnBackground() {
        Note note = new Note("Test", "Content", "personal", "#FFF");
        
        // Capture the Runnable passed to executor
        ArgumentCaptor<Runnable> runnableCaptor = ArgumentCaptor.forClass(Runnable.class);
        
        repository.insert(note, null);
        
        verify(executor).execute(runnableCaptor.capture());
        
        // Run the captured runnable
        runnableCaptor.getValue().run();
        
        verify(noteDao).insert(note);
    }
}
```

---

## 51.5 Room Database Test (Instrumented)

```java
// androidTest/database/NoteDaoTest.java
@RunWith(AndroidJUnit4.class)
public class NoteDaoTest {
    
    private AppDatabase db;
    private NoteDao noteDao;
    
    @Before
    public void createDb() {
        Context context = ApplicationProvider.getApplicationContext();
        
        // Use in-memory database for testing
        db = Room.inMemoryDatabaseBuilder(context, AppDatabase.class)
            .allowMainThreadQueries()  // for testing only
            .build();
        
        noteDao = db.noteDao();
    }
    
    @After
    public void closeDb() {
        db.close();
    }
    
    @Test
    public void insertAndRead() throws InterruptedException {
        Note note = new Note("Test Title", "Test Content", "personal", "#FFF");
        noteDao.insert(note);
        
        List<Note> notes = LiveDataTestUtil.getOrAwaitValue(noteDao.getAllNotes());
        
        assertEquals(1, notes.size());
        assertEquals("Test Title", notes.get(0).getTitle());
    }
    
    @Test
    public void updateNote() throws InterruptedException {
        Note note = new Note("Original", "Content", "personal", "#FFF");
        long id = noteDao.insert(note);
        
        note.setId((int) id);
        note.setTitle("Updated");
        noteDao.update(note);
        
        List<Note> notes = LiveDataTestUtil.getOrAwaitValue(noteDao.getAllNotes());
        assertEquals("Updated", notes.get(0).getTitle());
    }
    
    @Test
    public void deleteNote() throws InterruptedException {
        Note note = new Note("ToDelete", "Content", "personal", "#FFF");
        long id = noteDao.insert(note);
        note.setId((int) id);
        
        noteDao.delete(note);
        
        List<Note> notes = LiveDataTestUtil.getOrAwaitValue(noteDao.getAllNotes());
        assertTrue(notes.isEmpty());
    }
    
    @Test
    public void searchNotes() throws InterruptedException {
        noteDao.insert(new Note("Java Tips", "Content", "study", "#FFF"));
        noteDao.insert(new Note("Android Dev", "Content", "work", "#FFF"));
        noteDao.insert(new Note("Java Best Practices", "Content", "work", "#FFF"));
        
        List<Note> results = LiveDataTestUtil.getOrAwaitValue(noteDao.searchNotes("%Java%"));
        
        assertEquals(2, results.size());
    }
}
```

---

## 51.6 Espresso UI Tests

```java
// androidTest/ui/MainActivityTest.java
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {
    
    @Rule
    public ActivityScenarioRule<MainActivity> activityRule =
        new ActivityScenarioRule<>(MainActivity.class);
    
    @Test
    public void testRecyclerViewDisplayed() {
        onView(withId(R.id.recyclerView)).check(matches(isDisplayed()));
    }
    
    @Test
    public void testFabOpensAddNoteActivity() {
        onView(withId(R.id.fab)).perform(click());
        
        // Check that AddEditNoteActivity is shown
        onView(withId(R.id.etTitle)).check(matches(isDisplayed()));
    }
    
    @Test
    public void testAddNote() {
        String testTitle = "Test Note " + System.currentTimeMillis();
        
        // Click FAB to add note
        onView(withId(R.id.fab)).perform(click());
        
        // Type title
        onView(withId(R.id.etTitle)).perform(typeText(testTitle), closeSoftKeyboard());
        
        // Type content
        onView(withId(R.id.etContent)).perform(typeText("Test Content"), closeSoftKeyboard());
        
        // Click save
        onView(withId(R.id.btnSave)).perform(click());
        
        // Verify note appears in list
        onView(withText(testTitle)).check(matches(isDisplayed()));
    }
    
    @Test
    public void testSearchFiltersNotes() {
        onView(withId(R.id.etSearch))
            .perform(typeText("Java"), closeSoftKeyboard());
        
        // After search, only Java-related notes should show
        onView(withId(R.id.recyclerView))
            .check(new RecyclerViewItemCountAssertion(greaterThanOrEqualTo(0)));
    }
}

// Custom RecyclerView assertion
class RecyclerViewItemCountAssertion implements ViewAssertion {
    private final Matcher<Integer> matcher;
    
    RecyclerViewItemCountAssertion(Matcher<Integer> matcher) {
        this.matcher = matcher;
    }
    
    @Override
    public void check(View view, NoMatchingViewException e) {
        if (e != null) throw e;
        
        RecyclerView recyclerView = (RecyclerView) view;
        RecyclerView.Adapter adapter = recyclerView.getAdapter();
        assertThat(adapter.getItemCount(), matcher);
    }
}
```

---

## 51.7 Retrofit Mock Testing

```java
// test/api/ApiServiceTest.java
@RunWith(JUnit4.class)
public class ApiServiceTest {
    
    private MockWebServer mockWebServer;
    private ApiService apiService;
    
    @Before
    public void setup() throws IOException {
        mockWebServer = new MockWebServer();
        mockWebServer.start();
        
        Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(mockWebServer.url("/"))
            .addConverterFactory(GsonConverterFactory.create())
            .build();
        
        apiService = retrofit.create(ApiService.class);
    }
    
    @After
    public void teardown() throws IOException {
        mockWebServer.shutdown();
    }
    
    @Test
    public void testGetPosts_success() throws IOException {
        // Enqueue mock response
        String json = "[{\"id\":1,\"title\":\"Post 1\",\"body\":\"Content\",\"userId\":1}]";
        mockWebServer.enqueue(new MockResponse()
            .setBody(json)
            .setResponseCode(200));
        
        // Execute
        Response<List<Post>> response = apiService.getPosts().execute();
        
        assertTrue(response.isSuccessful());
        List<Post> posts = response.body();
        assertNotNull(posts);
        assertEquals(1, posts.size());
        assertEquals("Post 1", posts.get(0).getTitle());
    }
    
    @Test
    public void testGetPosts_networkError() throws IOException {
        mockWebServer.enqueue(new MockResponse()
            .setResponseCode(500)
            .setBody("Server Error"));
        
        Response<List<Post>> response = apiService.getPosts().execute();
        
        assertFalse(response.isSuccessful());
        assertEquals(500, response.code());
    }
}
```

---

## 51.8 สรุป Part 51

ในบทนี้คุณได้เรียนรู้:

✅ Unit tests vs Instrumented tests  
✅ ViewModel testing with Mockito  
✅ LiveData testing utilities  
✅ Room DAO tests (in-memory database)  
✅ Espresso UI tests (click, type, check)  
✅ MockWebServer for Retrofit tests  
✅ Custom Espresso ViewAssertions  

---

*[← Part 50: Hilt DI](./part-50-android-hilt.md) | [Part 52: Performance Optimization →](./part-52-android-performance.md)*
