# Part 92: Android Architecture Patterns Comparison
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 92.1 MVC vs MVP vs MVVM

```
MVC (Model-View-Controller):
┌─────────┐    user input    ┌────────────┐
│  View   │ ───────────────> │ Controller │
│(Activity│ <─────────────── │            │
│/Fragment│   update view    └────────────┘
└─────────┘                       │ updates
                                   ▼
                              ┌─────────┐
                              │  Model  │
                              └─────────┘
Problems: Controller becomes fat, View and Controller tightly coupled

MVP (Model-View-Presenter):
┌─────────┐   calls          ┌───────────┐
│  View   │ ───────────────> │ Presenter │
│(interface)◄─────────────── │           │
└─────────┘   updates view   └───────────┘
                                   │ uses
                                   ▼
                              ┌─────────┐
                              │  Model  │
                              └─────────┘
Better: Presenter has no Android dependencies (testable)
Problem: View interface verbose, memory leak if not careful

MVVM (Model-View-ViewModel):
┌──────────┐  observe         ┌───────────┐
│  View    │ ───────────────> │ ViewModel │
│(Activity/│ <─── LiveData ── │           │
│ Fragment)│                  └───────────┘
└──────────┘                       │ uses
                                   ▼
                              ┌─────────┐
                              │  Model  │
                              │(Repo)   │
                              └─────────┘
Best for Android: ViewModel survives rotation, LiveData is lifecycle-aware
```

---

## 92.2 MVP Implementation

```java
// Contract (interface defining View and Presenter)
public interface LoginContract {
    
    interface View {
        void showLoading();
        void hideLoading();
        void showError(String message);
        void navigateToHome();
        void clearForm();
    }
    
    interface Presenter {
        void onLoginClicked(String email, String password);
        void onForgotPasswordClicked(String email);
        void onDestroy();
    }
}

// LoginPresenter.java
public class LoginPresenter implements LoginContract.Presenter {
    
    private LoginContract.View view;  // NOT WeakReference (clean in onDestroy)
    private final UserRepository repository;
    private final CompositeDisposable disposables = new CompositeDisposable();
    
    public LoginPresenter(LoginContract.View view, UserRepository repository) {
        this.view = view;
        this.repository = repository;
    }
    
    @Override
    public void onLoginClicked(String email, String password) {
        if (email.isEmpty() || password.isEmpty()) {
            view.showError("กรุณากรอกอีเมลและรหัสผ่าน");
            return;
        }
        
        view.showLoading();
        
        disposables.add(
            repository.login(email, password)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
                    user -> {
                        view.hideLoading();
                        view.navigateToHome();
                    },
                    error -> {
                        view.hideLoading();
                        view.showError(error.getMessage());
                    }
                )
        );
    }
    
    @Override
    public void onForgotPasswordClicked(String email) {
        if (email.isEmpty()) {
            view.showError("กรุณากรอกอีเมล");
            return;
        }
        repository.sendPasswordReset(email)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                () -> view.showError("ส่งลิงก์รีเซ็ตรหัสผ่านแล้ว"),
                e -> view.showError(e.getMessage())
            );
    }
    
    @Override
    public void onDestroy() {
        disposables.clear();
        view = null;  // prevent memory leak
    }
}

// LoginActivity.java (View)
public class LoginActivity extends AppCompatActivity implements LoginContract.View {
    
    private LoginPresenter presenter;
    private EditText etEmail, etPassword;
    private ProgressBar progressBar;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        presenter = new LoginPresenter(this, new UserRepositoryImpl());
        
        etEmail    = findViewById(R.id.etEmail);
        etPassword = findViewById(R.id.etPassword);
        progressBar = findViewById(R.id.progressBar);
        
        findViewById(R.id.btnLogin).setOnClickListener(v ->
            presenter.onLoginClicked(
                etEmail.getText().toString(),
                etPassword.getText().toString()));
        
        findViewById(R.id.tvForgotPassword).setOnClickListener(v ->
            presenter.onForgotPasswordClicked(etEmail.getText().toString()));
    }
    
    @Override protected void onDestroy() {
        super.onDestroy();
        presenter.onDestroy();
    }
    
    @Override public void showLoading()  { progressBar.setVisibility(View.VISIBLE); }
    @Override public void hideLoading()  { progressBar.setVisibility(View.GONE); }
    @Override public void clearForm()    { etEmail.setText(""); etPassword.setText(""); }
    
    @Override public void showError(String message) {
        Snackbar.make(findViewById(android.R.id.content), message, Snackbar.LENGTH_LONG).show();
    }
    
    @Override public void navigateToHome() {
        startActivity(new Intent(this, MainActivity.class));
        finish();
    }
}
```

---

## 92.3 Architecture Decision Guide

```
Choose based on project scale:

Small project / Learning:
  → MVC (simple, built-in Android pattern)

Medium project / Team of 2-3:
  → MVP (clear separation, easy to test Presenter)
  → OR MVVM with LiveData

Large project / Team 4+:
  → MVVM + Clean Architecture
  → Multi-module (feature isolation)
  → Hilt (DI at scale)

Key questions:
1. Can I unit-test the business logic without Android SDK? 
   → Yes: MVP Presenter or MVVM ViewModel
   → No: MVC (Activity/Fragment has logic)

2. Does UI need to survive rotation?
   → Yes: MVVM with ViewModel

3. Multiple developers on same feature?
   → Clean Architecture (domain layer = shared contract)
   → Data/Domain/Presentation layers owned separately
```

---

## 92.4 สรุป Part 92

ในบทนี้คุณได้เรียนรู้:

✅ MVC vs MVP vs MVVM comparison  
✅ MVP Contract interface  
✅ LoginPresenter (testable, no Android deps)  
✅ onDestroy cleanup (prevent leak)  
✅ View interface in Activity  
✅ Architecture decision guide  
✅ Clean Architecture for large teams  

---

*[← Part 91: Java Reflection](./part-91-java-reflection.md) | [Part 93: Java Generics ขั้นสูง →](./part-93-java-generics.md)*
