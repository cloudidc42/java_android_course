# Part 57: Advanced RecyclerView - Paging 3 & Multiple ViewTypes
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 57.1 Multiple View Types

```java
// MultiTypeAdapter.java - Feed with different item types
public class FeedAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    
    // View type constants
    private static final int TYPE_TEXT   = 0;
    private static final int TYPE_IMAGE  = 1;
    private static final int TYPE_VIDEO  = 2;
    private static final int TYPE_AD     = 3;
    private static final int TYPE_HEADER = 4;
    
    private List<FeedItem> items = new ArrayList<>();
    
    @Override
    public int getItemViewType(int position) {
        FeedItem item = items.get(position);
        switch (item.getType()) {
            case TEXT:   return TYPE_TEXT;
            case IMAGE:  return TYPE_IMAGE;
            case VIDEO:  return TYPE_VIDEO;
            case AD:     return TYPE_AD;
            case HEADER: return TYPE_HEADER;
            default:     return TYPE_TEXT;
        }
    }
    
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        switch (viewType) {
            case TYPE_TEXT:
                return new TextViewHolder(inflater.inflate(R.layout.item_text, parent, false));
            case TYPE_IMAGE:
                return new ImageViewHolder(inflater.inflate(R.layout.item_image, parent, false));
            case TYPE_VIDEO:
                return new VideoViewHolder(inflater.inflate(R.layout.item_video, parent, false));
            case TYPE_AD:
                return new AdViewHolder(inflater.inflate(R.layout.item_ad, parent, false));
            case TYPE_HEADER:
                return new HeaderViewHolder(inflater.inflate(R.layout.item_header, parent, false));
            default:
                throw new IllegalArgumentException("Unknown view type: " + viewType);
        }
    }
    
    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        FeedItem item = items.get(position);
        if (holder instanceof TextViewHolder) {
            ((TextViewHolder) holder).bind((TextFeedItem) item);
        } else if (holder instanceof ImageViewHolder) {
            ((ImageViewHolder) holder).bind((ImageFeedItem) item);
        }
        // etc.
    }
    
    @Override
    public int getItemCount() { return items.size(); }
    
    // ViewHolders
    static class TextViewHolder extends RecyclerView.ViewHolder {
        TextView tvTitle, tvContent;
        
        TextViewHolder(View view) {
            super(view);
            tvTitle   = view.findViewById(R.id.tvTitle);
            tvContent = view.findViewById(R.id.tvContent);
        }
        
        void bind(TextFeedItem item) {
            tvTitle.setText(item.getTitle());
            tvContent.setText(item.getContent());
        }
    }
    
    static class ImageViewHolder extends RecyclerView.ViewHolder {
        ImageView ivImage;
        TextView tvCaption;
        
        ImageViewHolder(View view) {
            super(view);
            ivImage   = view.findViewById(R.id.ivImage);
            tvCaption = view.findViewById(R.id.tvCaption);
        }
        
        void bind(ImageFeedItem item) {
            tvCaption.setText(item.getCaption());
            Glide.with(itemView).load(item.getImageUrl())
                .centerCrop().into(ivImage);
        }
    }
}
```

---

## 57.2 Paging 3 Library

```groovy
// build.gradle
implementation "androidx.paging:paging-runtime:3.2.1"
```

```java
// paging/UserPagingSource.java
import androidx.paging.*;

public class UserPagingSource extends PagingSource<Integer, User> {
    
    private final UserApiService apiService;
    
    public UserPagingSource(UserApiService apiService) {
        this.apiService = apiService;
    }
    
    @Override
    public Single<LoadResult<Integer, User>> loadSingle(LoadParams<Integer> params) {
        int page = params.getKey() != null ? params.getKey() : 1;
        int pageSize = params.getLoadSize();
        
        // Fetch from API
        try {
            Response<PagedResponse<UserDto>> response = apiService
                .getUsers(page, pageSize).execute();
            
            if (!response.isSuccessful() || response.body() == null) {
                return Single.just(new LoadResult.Error<>(
                    new Exception("API Error: " + response.code())));
            }
            
            PagedResponse<UserDto> body = response.body();
            List<User> users = body.getData().stream()
                .map(UserMapper::fromDto)
                .collect(Collectors.toList());
            
            Integer prevPage = page > 1 ? page - 1 : null;
            Integer nextPage = page < body.getTotalPages() ? page + 1 : null;
            
            return Single.just(new LoadResult.Page<>(users, prevPage, nextPage));
            
        } catch (IOException e) {
            return Single.just(new LoadResult.Error<>(e));
        }
    }
    
    @Override
    public Integer getRefreshKey(PagingState<Integer, User> state) {
        Integer anchorPosition = state.getAnchorPosition();
        if (anchorPosition == null) return null;
        
        LoadResult.Page<Integer, User> anchorPage = state.closestPageToPosition(anchorPosition);
        if (anchorPage == null) return null;
        
        Integer prevKey = anchorPage.getPrevKey();
        if (prevKey != null) return prevKey + 1;
        
        Integer nextKey = anchorPage.getNextKey();
        if (nextKey != null) return nextKey - 1;
        
        return null;
    }
}

// Repository
public class UserRepository {
    
    private final UserApiService apiService;
    
    public Pager<Integer, User> getUsersPager() {
        return new Pager<>(
            new PagingConfig(
                20,          // page size
                5,           // prefetch distance
                false        // enable placeholders
            ),
            () -> new UserPagingSource(apiService)
        );
    }
    
    public LiveData<PagingData<User>> getUsersLiveData() {
        return PagingLiveData.getLiveData(getUsersPager());
    }
}

// ViewModel
@HiltViewModel
public class UsersViewModel extends ViewModel {
    
    private final UserRepository repository;
    
    @Inject
    public UsersViewModel(UserRepository repository) {
        this.repository = repository;
    }
    
    public LiveData<PagingData<User>> users = 
        PagingLiveData.cachedIn(repository.getUsersLiveData(), this);
}

// Paging Adapter
public class UserPagingAdapter extends PagingDataAdapter<User, UserPagingAdapter.ViewHolder> {
    
    public UserPagingAdapter() {
        super(USER_COMPARATOR);
    }
    
    static final DiffUtil.ItemCallback<User> USER_COMPARATOR = 
        new DiffUtil.ItemCallback<User>() {
            @Override
            public boolean areItemsTheSame(User old, User newUser) {
                return old.getId() == newUser.getId();
            }
            
            @Override
            public boolean areContentsTheSame(User old, User newUser) {
                return old.getName().equals(newUser.getName()) &&
                       old.getEmail().equals(newUser.getEmail());
            }
        };
    
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
            .inflate(R.layout.item_user, parent, false);
        return new ViewHolder(view);
    }
    
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        User user = getItem(position);
        if (user != null) holder.bind(user);
        // null = placeholder
    }
    
    static class ViewHolder extends RecyclerView.ViewHolder {
        TextView tvName, tvEmail;
        
        ViewHolder(View view) {
            super(view);
            tvName  = view.findViewById(R.id.tvName);
            tvEmail = view.findViewById(R.id.tvEmail);
        }
        
        void bind(User user) {
            tvName.setText(user.getName());
            tvEmail.setText(user.getEmail());
        }
    }
}

// In Fragment/Activity
@AndroidEntryPoint
public class UsersFragment extends Fragment {
    
    private UsersViewModel viewModel;
    private UserPagingAdapter adapter;
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        RecyclerView recyclerView = view.findViewById(R.id.recyclerView);
        
        adapter = new UserPagingAdapter();
        recyclerView.setAdapter(adapter.withLoadStateFooter(
            new LoadStateAdapter() {
                @Override
                public LoadStateViewHolder onCreateViewHolder(ViewGroup parent, LoadState loadState) {
                    View v = LayoutInflater.from(parent.getContext())
                        .inflate(R.layout.item_loading_state, parent, false);
                    return new LoadStateViewHolder(v, () -> adapter.retry());
                }
                
                @Override
                public void onBindViewHolder(LoadStateViewHolder holder, LoadState loadState) {
                    holder.bind(loadState);
                }
            }
        ));
        
        viewModel = new ViewModelProvider(this).get(UsersViewModel.class);
        
        viewModel.users.observe(getViewLifecycleOwner(), pagingData -> {
            adapter.submitData(getLifecycle(), pagingData);
        });
        
        // Refresh on swipe
        SwipeRefreshLayout swipeRefresh = view.findViewById(R.id.swipeRefresh);
        swipeRefresh.setOnRefreshListener(() -> {
            adapter.refresh();
            swipeRefresh.setRefreshing(false);
        });
        
        // Handle load states
        adapter.addLoadStateListener(loadStates -> {
            CombinedLoadStates states = loadStates;
            if (states.getRefresh() instanceof LoadState.Loading) {
                // show progress
            } else if (states.getRefresh() instanceof LoadState.Error) {
                LoadState.Error error = (LoadState.Error) states.getRefresh();
                Toast.makeText(requireContext(), 
                    "Error: " + error.getError().getMessage(), Toast.LENGTH_SHORT).show();
            }
            return null;
        });
    }
}
```

---

## 57.3 Expandable RecyclerView

```java
// ExpandableAdapter.java
public class ExpandableAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    
    private static final int TYPE_HEADER = 0;
    private static final int TYPE_CHILD  = 1;
    
    private final List<Group> groups;
    private final Set<Integer> expandedGroups = new HashSet<>();
    
    public ExpandableAdapter(List<Group> groups) {
        this.groups = groups;
    }
    
    @Override
    public int getItemCount() {
        int count = 0;
        for (int i = 0; i < groups.size(); i++) {
            count++;  // header
            if (expandedGroups.contains(i)) {
                count += groups.get(i).children.size();
            }
        }
        return count;
    }
    
    @Override
    public int getItemViewType(int position) {
        return isHeader(position) ? TYPE_HEADER : TYPE_CHILD;
    }
    
    private boolean isHeader(int flatPosition) {
        int pos = 0;
        for (int i = 0; i < groups.size(); i++) {
            if (pos == flatPosition) return true;
            pos++;
            if (expandedGroups.contains(i)) pos += groups.get(i).children.size();
        }
        return false;
    }
    
    private void toggleGroup(int groupIndex) {
        if (expandedGroups.contains(groupIndex)) {
            expandedGroups.remove(groupIndex);
            notifyDataSetChanged();
        } else {
            expandedGroups.add(groupIndex);
            notifyDataSetChanged();
        }
    }
    
    static class Group {
        String title;
        List<String> children;
        
        Group(String title, List<String> children) {
            this.title = title;
            this.children = children;
        }
    }
}
```

---

## 57.4 Swipe to Delete & Drag to Reorder

```java
// ItemTouchHelperCallback.java
public class ItemTouchHelperCallback extends ItemTouchHelper.SimpleCallback {
    
    private final OnItemActionListener listener;
    
    public interface OnItemActionListener {
        void onItemSwipedLeft(int position);
        void onItemMoved(int from, int to);
    }
    
    public ItemTouchHelperCallback(OnItemActionListener listener) {
        super(ItemTouchHelper.UP | ItemTouchHelper.DOWN,  // drag directions
              ItemTouchHelper.LEFT);                       // swipe directions
        this.listener = listener;
    }
    
    @Override
    public boolean onMove(RecyclerView rv, RecyclerView.ViewHolder source,
            RecyclerView.ViewHolder target) {
        listener.onItemMoved(source.getAdapterPosition(), target.getAdapterPosition());
        return true;
    }
    
    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        if (direction == ItemTouchHelper.LEFT) {
            listener.onItemSwipedLeft(viewHolder.getAdapterPosition());
        }
    }
    
    // Draw red background + delete icon when swiping
    @Override
    public void onChildDraw(Canvas canvas, RecyclerView recyclerView,
            RecyclerView.ViewHolder viewHolder, float dX, float dY,
            int actionState, boolean isCurrentlyActive) {
        
        View itemView = viewHolder.itemView;
        
        if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE && dX < 0) {
            // Red background
            Paint paint = new Paint();
            paint.setColor(Color.parseColor("#F44336"));
            canvas.drawRect(
                itemView.getRight() + dX, itemView.getTop(),
                itemView.getRight(), itemView.getBottom(), paint);
            
            // Delete icon
            Drawable icon = ContextCompat.getDrawable(
                recyclerView.getContext(), R.drawable.ic_delete);
            if (icon != null) {
                int margin = (itemView.getHeight() - icon.getIntrinsicHeight()) / 2;
                int left = itemView.getRight() - margin - icon.getIntrinsicWidth();
                int top  = itemView.getTop() + margin;
                icon.setBounds(left, top,
                    left + icon.getIntrinsicWidth(),
                    top + icon.getIntrinsicHeight());
                icon.setTint(Color.WHITE);
                icon.draw(canvas);
            }
        }
        
        super.onChildDraw(canvas, recyclerView, viewHolder,
            dX, dY, actionState, isCurrentlyActive);
    }
}

// Usage in Fragment/Activity
ItemTouchHelper.Callback callback = new ItemTouchHelperCallback(
    new ItemTouchHelperCallback.OnItemActionListener() {
        @Override
        public void onItemSwipedLeft(int position) {
            // Show undo snackbar
            Note deleted = adapter.removeItem(position);
            Snackbar.make(recyclerView, "ลบแล้ว", Snackbar.LENGTH_LONG)
                .setAction("เลิกทำ", v -> adapter.addItem(position, deleted))
                .show();
        }
        
        @Override
        public void onItemMoved(int from, int to) {
            adapter.moveItem(from, to);
        }
    });

new ItemTouchHelper(callback).attachToRecyclerView(recyclerView);
```

---

## 57.5 สรุป Part 57

ในบทนี้คุณได้เรียนรู้:

✅ Multiple ViewTypes in RecyclerView  
✅ Paging 3 (PagingSource, Pager, PagingDataAdapter)  
✅ Load state footer (loading/error indicators)  
✅ Expandable RecyclerView  
✅ ItemTouchHelper (swipe to delete, drag to reorder)  
✅ Custom swipe drawing (background + icon)  
✅ Undo delete with Snackbar  

---

*[← Part 56: Architecture](./part-56-android-architecture.md) | [Part 58: Accessibility →](./part-58-android-accessibility.md)*
