# Part 94: Production-Ready Error Handling
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 94.1 Custom Exception Hierarchy

```java
// Base app exception
public class AppException extends RuntimeException {
    private final ErrorCode errorCode;
    private final Map<String, Object> context;
    
    public AppException(ErrorCode errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
        this.context   = new HashMap<>();
    }
    
    public AppException(ErrorCode errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.context   = new HashMap<>();
    }
    
    public AppException addContext(String key, Object value) {
        context.put(key, value);
        return this;  // fluent
    }
    
    public ErrorCode         getErrorCode() { return errorCode; }
    public Map<String, Object> getContext() { return Collections.unmodifiableMap(context); }
}

// Error codes
public enum ErrorCode {
    // Network
    NETWORK_TIMEOUT(1001, "Connection timed out"),
    NETWORK_NO_CONNECTION(1002, "No internet connection"),
    NETWORK_SERVER_ERROR(1003, "Server error"),
    
    // Auth
    AUTH_INVALID_CREDENTIALS(2001, "Invalid email or password"),
    AUTH_TOKEN_EXPIRED(2002, "Session expired, please login again"),
    AUTH_UNAUTHORIZED(2003, "You don't have permission"),
    
    // Data
    DATA_NOT_FOUND(3001, "Data not found"),
    DATA_VALIDATION_FAILED(3002, "Validation failed"),
    DATA_CONFLICT(3003, "Data conflict"),
    
    // App
    APP_GENERIC(9999, "Something went wrong");
    
    private final int    code;
    private final String defaultMessage;
    
    ErrorCode(int code, String message) {
        this.code = code;
        this.defaultMessage = message;
    }
    
    public int    getCode()           { return code; }
    public String getDefaultMessage() { return defaultMessage; }
}

// Specific exceptions
public class NetworkException extends AppException {
    public NetworkException(String message) {
        super(ErrorCode.NETWORK_TIMEOUT, message);
    }
    public NetworkException(String message, Throwable cause) {
        super(ErrorCode.NETWORK_TIMEOUT, message, cause);
    }
}

public class AuthException extends AppException {
    public AuthException(ErrorCode code) {
        super(code, code.getDefaultMessage());
    }
}

public class ValidationException extends AppException {
    private final List<String> violations;
    
    public ValidationException(List<String> violations) {
        super(ErrorCode.DATA_VALIDATION_FAILED, "Validation failed: " + violations);
        this.violations = violations;
    }
    
    public List<String> getViolations() { return violations; }
}
```

---

## 94.2 Global Error Handler

```java
// ErrorHandler.java - centralized error processing
@Singleton
public class ErrorHandler {
    
    private final Context context;
    private final CrashReporter crashReporter;
    
    @Inject
    public ErrorHandler(Application app, CrashReporter crashReporter) {
        this.context = app.getApplicationContext();
        this.crashReporter = crashReporter;
    }
    
    public ErrorUiState handle(Throwable throwable) {
        if (throwable instanceof AppException) {
            return handleAppException((AppException) throwable);
        } else if (throwable instanceof retrofit2.HttpException) {
            return handleHttpException((retrofit2.HttpException) throwable);
        } else if (throwable instanceof java.net.UnknownHostException
                || throwable instanceof java.net.ConnectException) {
            return noConnection();
        } else if (throwable instanceof java.net.SocketTimeoutException) {
            return timeout();
        } else {
            crashReporter.recordException(throwable);
            return generic();
        }
    }
    
    private ErrorUiState handleAppException(AppException e) {
        switch (e.getErrorCode()) {
            case AUTH_TOKEN_EXPIRED:
                // Could trigger auto-logout
                return new ErrorUiState(
                    context.getString(R.string.error_session_expired),
                    ErrorAction.NAVIGATE_TO_LOGIN);
            case NETWORK_NO_CONNECTION:
                return noConnection();
            case AUTH_INVALID_CREDENTIALS:
                return new ErrorUiState(
                    context.getString(R.string.error_invalid_credentials),
                    ErrorAction.SHOW_SNACKBAR);
            default:
                crashReporter.recordException(e);
                return generic();
        }
    }
    
    private ErrorUiState handleHttpException(retrofit2.HttpException e) {
        switch (e.code()) {
            case 401: return new ErrorUiState("กรุณาเข้าสู่ระบบใหม่", ErrorAction.NAVIGATE_TO_LOGIN);
            case 403: return new ErrorUiState("คุณไม่มีสิทธิ์เข้าถึง", ErrorAction.SHOW_SNACKBAR);
            case 404: return new ErrorUiState("ไม่พบข้อมูล", ErrorAction.SHOW_SNACKBAR);
            case 429: return new ErrorUiState("คำขอมากเกินไป กรุณารอสักครู่", ErrorAction.SHOW_SNACKBAR);
            case 500: return serverError();
            default:  return generic();
        }
    }
    
    private ErrorUiState noConnection() {
        return new ErrorUiState(
            context.getString(R.string.error_no_connection),
            ErrorAction.SHOW_RETRY);
    }
    
    private ErrorUiState timeout() {
        return new ErrorUiState(
            context.getString(R.string.error_timeout),
            ErrorAction.SHOW_RETRY);
    }
    
    private ErrorUiState serverError() {
        return new ErrorUiState(
            context.getString(R.string.error_server),
            ErrorAction.SHOW_SNACKBAR);
    }
    
    private ErrorUiState generic() {
        return new ErrorUiState(
            context.getString(R.string.error_generic),
            ErrorAction.SHOW_SNACKBAR);
    }
    
    public static class ErrorUiState {
        public final String      message;
        public final ErrorAction action;
        
        public ErrorUiState(String message, ErrorAction action) {
            this.message = message;
            this.action = action;
        }
    }
    
    public enum ErrorAction {
        SHOW_SNACKBAR,
        SHOW_RETRY,
        NAVIGATE_TO_LOGIN,
        SHOW_DIALOG
    }
}
```

---

## 94.3 ViewModel Error Handling

```java
// BaseViewModel.java
public abstract class BaseViewModel extends ViewModel {
    
    protected final ErrorHandler errorHandler;
    
    private final MutableLiveData<ErrorHandler.ErrorUiState> errorState = new MutableLiveData<>();
    
    protected BaseViewModel(ErrorHandler errorHandler) {
        this.errorHandler = errorHandler;
    }
    
    protected void handleError(Throwable throwable) {
        ErrorHandler.ErrorUiState state = errorHandler.handle(throwable);
        errorState.postValue(state);
    }
    
    public LiveData<ErrorHandler.ErrorUiState> getErrorState() { return errorState; }
}

// In concrete ViewModel
@HiltViewModel
public class CheckoutViewModel extends BaseViewModel {
    
    @Inject
    public CheckoutViewModel(OrderRepository repository, ErrorHandler errorHandler) {
        super(errorHandler);
        this.repository = repository;
    }
    
    public void placeOrder(Order order) {
        repository.placeOrder(order)
            .thenAccept(result -> orderSuccess.postValue(result))
            .exceptionally(e -> {
                handleError(e.getCause() != null ? e.getCause() : e);
                return null;
            });
    }
}

// In Fragment
viewModel.getErrorState().observe(getViewLifecycleOwner(), errorState -> {
    switch (errorState.action) {
        case SHOW_SNACKBAR:
            Snackbar.make(requireView(), errorState.message, Snackbar.LENGTH_LONG).show();
            break;
        case SHOW_RETRY:
            showRetryDialog(errorState.message);
            break;
        case NAVIGATE_TO_LOGIN:
            Navigation.findNavController(requireView()).navigate(R.id.loginFragment);
            break;
    }
});
```

---

## 94.4 สรุป Part 94

ในบทนี้คุณได้เรียนรู้:

✅ Custom exception hierarchy  
✅ ErrorCode enum (network, auth, data)  
✅ Specific exceptions (NetworkException, AuthException, ValidationException)  
✅ Centralized ErrorHandler  
✅ Map HTTP status codes to user messages  
✅ ErrorAction enum (snackbar, retry, navigate)  
✅ BaseViewModel with error handling  
✅ Fragment observes error state  

---

*[← Part 93: Java Generics](./part-93-java-generics.md) | [Part 95: Database Migration Strategies →](./part-95-database-migration.md)*
