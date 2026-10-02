# Part 71: Reactive Programming with RxJava 3
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 71.1 Reactive Programming คืออะไร

```
Imperative (traditional):
List<User> users = getUsers();  // blocking
for (User u : users) process(u);

Reactive:
Observable<User> users = getUsersObservable();  // non-blocking stream
users.filter(u -> u.isActive())
     .map(User::getName)
     .subscribe(name -> display(name));

ข้อดี: non-blocking, composable, easy error handling, backpressure support
```

---

## 71.2 Observable Types

```java
// Observable - stream of 0..N items (no backpressure)
Observable<String> names = Observable.fromIterable(Arrays.asList("Alice", "Bob", "Charlie"));

// Flowable - stream with backpressure support
Flowable<Integer> numbers = Flowable.range(1, 1000);

// Single - exactly 1 item or error
Single<User> userById = Single.fromCallable(() -> userDao.getUserById(1));

// Maybe - 0 or 1 item or error (optional)
Maybe<User> userMaybe = Maybe.fromCallable(() -> userDao.findByEmail("a@b.com"));

// Completable - no items, just completion or error
Completable saveUser = Completable.fromAction(() -> userDao.insert(user));

// Observable creation
Observable<Integer> just = Observable.just(1, 2, 3);
Observable<Long> interval = Observable.interval(1, TimeUnit.SECONDS);
Observable<Integer> defer = Observable.defer(() -> Observable.just(currentValue));
Observable<Integer> create = Observable.create(emitter -> {
    emitter.onNext(1);
    emitter.onNext(2);
    emitter.onComplete();
});
```

---

## 71.3 Operators

```java
// Transformation operators
Observable<User> users = userApiService.getUsers();

users
    // map: transform each item
    .map(UserDto::toDomain)
    
    // filter: keep items matching predicate
    .filter(User::isActive)
    
    // flatMap: transform to another Observable (merge)
    .flatMap(user -> userApiService.getUserDetails(user.getId()))
    
    // concatMap: flatMap but maintains order
    .concatMap(user -> loadUserPosts(user.getId()))
    
    // take: only first N items
    .take(10)
    
    // skip: skip first N items
    .skip(5)
    
    // debounce: wait for pause in events (search)
    .debounce(300, TimeUnit.MILLISECONDS)
    
    // distinctUntilChanged: skip duplicate consecutive items
    .distinctUntilChanged()
    
    // buffer: collect items into lists
    .buffer(5)  // emit List<User> of size 5
    
    // toList: collect all to List (terminal)
    .toList()
    
    .subscribe(
        userList -> updateUI(userList),
        error -> showError(error.getMessage())
    );

// Combining operators
Observable<String> stream1 = Observable.just("A", "B", "C");
Observable<String> stream2 = Observable.just("1", "2", "3");

// merge: interleave two streams
Observable.merge(stream1, stream2)
    .subscribe(s -> Log.d("TAG", s));  // A, 1, B, 2, C, 3 (order varies)

// zip: combine corresponding items
Observable.zip(stream1, stream2, (s1, s2) -> s1 + s2)
    .subscribe(s -> Log.d("TAG", s));  // A1, B2, C3

// combineLatest: combine latest from both whenever either emits
Observable.combineLatest(stream1, stream2, (s1, s2) -> s1 + s2)
    .subscribe(s -> Log.d("TAG", s));
```

---

## 71.4 Error Handling

```java
userApiService.getUsers()
    // Retry on error
    .retry(3)
    
    // Retry with delay
    .retryWhen(errors -> errors
        .zipWith(Observable.range(1, 3), (e, count) -> count)
        .flatMap(count -> Observable.timer(count * 2L, TimeUnit.SECONDS)))
    
    // Fallback value on error
    .onErrorReturn(e -> Collections.emptyList())
    
    // Fallback to another Observable on error
    .onErrorResumeNext(e -> userDao.getAllUsersObservable())
    
    // Handle error and continue
    .doOnError(e -> logError(e))
    
    .subscribe(
        users -> updateUI(users),
        error -> showError(error)  // only reaches here if not handled above
    );
```

---

## 71.5 Schedulers

```java
// subscribeOn: where to execute the upstream work
// observeOn: where to deliver results to subscriber

Single.fromCallable(() -> userDao.getAllUsersSync())
    .subscribeOn(Schedulers.io())          // execute on IO thread
    .observeOn(AndroidSchedulers.mainThread())  // deliver on main thread
    .subscribe(users -> recyclerAdapter.submitList(users));

// Android schedulers:
// Schedulers.io()          → IO operations (network, disk)
// Schedulers.computation() → CPU-intensive tasks
// Schedulers.single()      → single background thread (sequential)
// Schedulers.trampoline()  → queues on current thread
// AndroidSchedulers.mainThread() → Android main thread

// Disposable management
CompositeDisposable disposable = new CompositeDisposable();

disposable.add(
    userApiService.getUsers()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(this::updateUI, this::handleError)
);

// In onDestroy / onCleared
disposable.clear();
```

---

## 71.6 Search with Debounce

```java
// Real-world: search box with debounce
// In Activity/Fragment
PublishSubject<String> searchSubject = PublishSubject.create();
CompositeDisposable disposable = new CompositeDisposable();

// Wire up search EditText
editTextSearch.addTextChangedListener(new TextWatcher() {
    @Override public void onTextChanged(CharSequence s, int st, int be, int co) {
        searchSubject.onNext(s.toString());
    }
    @Override public void beforeTextChanged(CharSequence s, int st, int co, int af) {}
    @Override public void afterTextChanged(Editable s) {}
});

// Search pipeline
disposable.add(
    searchSubject
        .debounce(300, TimeUnit.MILLISECONDS)
        .distinctUntilChanged()
        .filter(query -> query.length() >= 2)
        .switchMap(query ->  // switchMap: cancel previous search
            apiService.searchProducts(query)
                .subscribeOn(Schedulers.io())
                .onErrorReturn(e -> Collections.emptyList()))
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(results -> {
            adapter.submitList(results);
            tvEmpty.setVisibility(results.isEmpty() ? View.VISIBLE : View.GONE);
        })
);
```

---

## 71.7 สรุป Part 71

ในบทนี้คุณได้เรียนรู้:

✅ Observable, Flowable, Single, Maybe, Completable  
✅ Operators (map, filter, flatMap, debounce, zip)  
✅ Error handling (retry, onErrorReturn, onErrorResumeNext)  
✅ Schedulers (io, computation, mainThread)  
✅ CompositeDisposable  
✅ Search with debounce + switchMap  

---

*[← Part 70: Concurrency](./part-70-java-advanced-concurrency.md) | [Part 72: Java Design Patterns Advanced →](./part-72-java-design-patterns-advanced.md)*
