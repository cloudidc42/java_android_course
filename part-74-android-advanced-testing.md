# Part 74: Advanced Android Testing
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 74.1 Test Pyramid

```
         /──────────────────\
        /   UI Tests (E2E)   \   ← slow, expensive, fragile
       /────────────────────────\
      /   Integration Tests      \  ← medium
     /────────────────────────────\
    /      Unit Tests              \  ← fast, cheap, reliable
   /────────────────────────────────\

Goal: 70% unit, 20% integration, 10% UI
```

---

## 74.2 Unit Testing Best Practices

```java
// UserViewModelTest.java - comprehensive unit test
@RunWith(MockitoJUnitRunner.class)
public class UserViewModelTest {
    
    @Rule
    public InstantTaskExecutorRule instantExecutorRule = new InstantTaskExecutorRule();
    
    @Mock UserRepository mockRepository;
    @Mock GetUsersUseCase mockGetUsersUseCase;
    @Mock CreateUserUseCase mockCreateUserUseCase;
    
    private UserViewModel viewModel;
    
    @Before
    public void setUp() {
        viewModel = new UserViewModel(mockGetUsersUseCase, mockCreateUserUseCase);
    }
    
    @Test
    public void createUser_withValidData_postSuccess() throws Exception {
        // Given
        String name  = "John Doe";
        String email = "john@example.com";
        User expectedUser = new User(1, name, email, "user", System.currentTimeMillis());
        
        when(mockCreateUserUseCase.execute(name, email, "user"))
            .thenReturn(CompletableFuture.completedFuture(Result.success(expectedUser)));
        
        // When
        viewModel.createUser(name, email);
        
        // Wait for async operation
        Thread.sleep(100);
        
        // Then
        UiState state = viewModel.getUiState().getValue();
        assertNotNull(state);
        assertEquals(UiState.Type.SUCCESS, state.type);
        assertEquals("สร้างผู้ใช้สำเร็จ", state.message);
    }
    
    @Test
    public void createUser_withEmptyName_postError() throws Exception {
        // Given
        when(mockCreateUserUseCase.execute("", "john@example.com", "user"))
            .thenReturn(CompletableFuture.completedFuture(Result.failure("Name is required")));
        
        // When
        viewModel.createUser("", "john@example.com");
        Thread.sleep(100);
        
        // Then
        UiState state = viewModel.getUiState().getValue();
        assertNotNull(state);
        assertEquals(UiState.Type.ERROR, state.type);
    }
    
    @Test
    public void createUser_networkFailure_postError() throws Exception {
        // Given
        CompletableFuture<Result<User>> failed = new CompletableFuture<>();
        failed.completeExceptionally(new IOException("Network error"));
        
        when(mockCreateUserUseCase.execute(any(), any(), any())).thenReturn(failed);
        
        // When
        viewModel.createUser("John", "john@example.com");
        Thread.sleep(100);
        
        // Then
        assertEquals(UiState.Type.ERROR, viewModel.getUiState().getValue().type);
    }
}

// Test doubles
// Mock: Mockito generates fake with verification
// Stub: returns canned values
// Fake: real implementation but in-memory
// Spy: wraps real object, intercept specific methods

// Spy example
@Spy
UserMapper realMapper;  // real object

verify(realMapper, times(1)).toDomain(any());  // verify was called
```

---

## 74.3 Integration Testing (Room)

```java
// NoteDaoIntegrationTest.java
@RunWith(AndroidJUnit4.class)
@SmallTest
public class NoteDaoIntegrationTest {
    
    private AppDatabase database;
    private NoteDao noteDao;
    
    @Before
    public void initDb() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase.class)
            .allowMainThreadQueries()
            .build();
        noteDao = database.noteDao();
    }
    
    @After
    public void closeDb() { database.close(); }
    
    @Test
    public void insertAndRetrieveNote() throws Exception {
        // Arrange
        NoteEntity note = new NoteEntity();
        note.title = "Test Note";
        note.content = "Hello World";
        note.userId = 1;
        note.createdAt = System.currentTimeMillis();
        
        // Act
        long id = noteDao.insert(note);
        
        // Assert
        NoteEntity retrieved = LiveDataTestUtil.getValue(noteDao.getNoteById((int) id));
        assertNotNull(retrieved);
        assertEquals("Test Note", retrieved.title);
        assertEquals("Hello World", retrieved.content);
    }
    
    @Test
    public void deleteNote_removesFromDB() throws Exception {
        // Arrange
        NoteEntity note = new NoteEntity();
        note.title = "Delete Me";
        long id = noteDao.insert(note);
        
        // Act
        noteDao.deleteById((int) id);
        
        // Assert
        List<NoteEntity> all = LiveDataTestUtil.getValue(noteDao.getAllNotes());
        assertTrue(all.stream().noneMatch(n -> n.id == (int) id));
    }
    
    @Test
    public void searchNotes_returnsMatchingNotes() throws Exception {
        // Arrange
        insertNote("Android Notes", "Learn Android");
        insertNote("Java Notes", "Learn Java");
        insertNote("Kotlin Notes", "Learn Kotlin");
        
        // Act
        List<NoteEntity> results = noteDao.searchSync("%Java%");
        
        // Assert
        assertEquals(1, results.size());
        assertEquals("Java Notes", results.get(0).title);
    }
    
    private void insertNote(String title, String content) {
        NoteEntity note = new NoteEntity();
        note.title = title;
        note.content = content;
        noteDao.insert(note);
    }
}

// LiveDataTestUtil.java
public class LiveDataTestUtil {
    
    public static <T> T getValue(LiveData<T> liveData) throws InterruptedException {
        final Object[] data = new Object[1];
        CountDownLatch latch = new CountDownLatch(1);
        
        Observer<T> observer = value -> {
            data[0] = value;
            latch.countDown();
        };
        
        liveData.observeForever(observer);
        latch.await(5, TimeUnit.SECONDS);
        liveData.removeObserver(observer);
        
        return (T) data[0];
    }
}
```

---

## 74.4 UI Testing with Espresso

```java
// LoginActivityTest.java
@RunWith(AndroidJUnit4.class)
@LargeTest
public class LoginActivityTest {
    
    @Rule
    public ActivityScenarioRule<LoginActivity> activityRule =
        new ActivityScenarioRule<>(LoginActivity.class);
    
    @Test
    public void login_withCorrectCredentials_navigatesToHome() {
        // Type email
        onView(withId(R.id.etEmail))
            .perform(typeText("user@example.com"), closeSoftKeyboard());
        
        // Type password
        onView(withId(R.id.etPassword))
            .perform(typeText("password123"), closeSoftKeyboard());
        
        // Click login
        onView(withId(R.id.btnLogin))
            .perform(click());
        
        // Verify home is shown
        onView(withId(R.id.homeFragment))
            .check(matches(isDisplayed()));
    }
    
    @Test
    public void login_withEmptyEmail_showsError() {
        onView(withId(R.id.etPassword))
            .perform(typeText("password123"), closeSoftKeyboard());
        
        onView(withId(R.id.btnLogin)).perform(click());
        
        onView(withText("กรุณากรอกอีเมล"))
            .check(matches(isDisplayed()));
    }
    
    @Test
    public void login_withInvalidEmail_showsValidationError() {
        onView(withId(R.id.etEmail))
            .perform(typeText("not-an-email"), closeSoftKeyboard());
        
        onView(withId(R.id.btnLogin)).perform(click());
        
        // Check TextInputLayout error
        onView(withId(R.id.tilEmail))
            .check(matches(hasDescendant(withText("รูปแบบอีเมลไม่ถูกต้อง"))));
    }
    
    @Test
    public void recyclerView_displaysItems() {
        // Verify RecyclerView has items
        onView(withId(R.id.recyclerView))
            .check(matches(hasMinimumChildCount(1)));
        
        // Click first item
        onView(withId(R.id.recyclerView))
            .perform(RecyclerViewActions.actionOnItemAtPosition(0, click()));
        
        // Verify detail screen shown
        onView(withId(R.id.tvDetailTitle))
            .check(matches(isDisplayed()));
    }
}
```

---

## 74.5 สรุป Part 74

ในบทนี้คุณได้เรียนรู้:

✅ Test pyramid (70/20/10 rule)  
✅ Unit testing best practices  
✅ Mock, Stub, Fake, Spy differences  
✅ Testing async code (CompletableFuture)  
✅ Room integration tests (inMemoryDatabase)  
✅ LiveDataTestUtil  
✅ Espresso UI tests (typeText, click, check)  
✅ RecyclerViewActions  

---

*[← Part 73: Performance](./part-73-android-app-performance.md) | [Part 75: Real-World Project →](./part-75-android-realworld-project.md)*
