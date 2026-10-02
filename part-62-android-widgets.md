# Part 62: App Widgets
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 62.1 App Widget คืออะไร

App Widget คือ UI เล็กๆ ที่แสดงบน Home Screen ของ Android

```
ประเภท Widget:
- Information widget: แสดงข้อมูล (clock, weather)
- Collection widget: แสดง list/grid (emails, tasks)
- Control widget: ควบคุมแอป (music player controls)
- Hybrid widget: ผสม information + control
```

---

## 62.2 Widget Provider

```java
// widget/NoteWidgetProvider.java
public class NoteWidgetProvider extends AppWidgetProvider {
    
    public static final String ACTION_REFRESH = "com.myapp.widget.REFRESH";
    public static final String ACTION_NEW_NOTE = "com.myapp.widget.NEW_NOTE";
    
    @Override
    public void onUpdate(Context context, AppWidgetManager manager, int[] widgetIds) {
        for (int widgetId : widgetIds) {
            updateWidget(context, manager, widgetId);
        }
    }
    
    public static void updateWidget(Context context, AppWidgetManager manager, int widgetId) {
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_note);
        
        // Set data
        views.setTextViewText(R.id.tvWidgetTitle, "My Notes");
        views.setTextViewText(R.id.tvNoteCount, "5 notes");
        
        // Click to open app
        Intent openApp = new Intent(context, MainActivity.class);
        PendingIntent openPendingIntent = PendingIntent.getActivity(context, 0, openApp,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        views.setOnClickPendingIntent(R.id.widgetRoot, openPendingIntent);
        
        // Refresh button
        Intent refreshIntent = new Intent(context, NoteWidgetProvider.class);
        refreshIntent.setAction(ACTION_REFRESH);
        refreshIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, widgetId);
        PendingIntent refreshPendingIntent = PendingIntent.getBroadcast(context, widgetId,
            refreshIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        views.setOnClickPendingIntent(R.id.btnRefresh, refreshPendingIntent);
        
        // New note button
        Intent newNoteIntent = new Intent(context, NoteEditorActivity.class);
        PendingIntent newNotePendingIntent = PendingIntent.getActivity(context, 1, newNoteIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        views.setOnClickPendingIntent(R.id.btnNewNote, newNotePendingIntent);
        
        manager.updateAppWidget(widgetId, views);
    }
    
    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        
        if (ACTION_REFRESH.equals(intent.getAction())) {
            int widgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
            if (widgetId != AppWidgetManager.INVALID_APPWIDGET_ID) {
                AppWidgetManager manager = AppWidgetManager.getInstance(context);
                updateWidget(context, manager, widgetId);
            }
        }
    }
    
    @Override
    public void onEnabled(Context context) {
        // Called when first widget instance added
    }
    
    @Override
    public void onDisabled(Context context) {
        // Called when last widget instance removed
    }
    
    // Call this from your app when data changes
    public static void notifyDataChanged(Context context) {
        AppWidgetManager manager = AppWidgetManager.getInstance(context);
        ComponentName component = new ComponentName(context, NoteWidgetProvider.class);
        int[] widgetIds = manager.getAppWidgetIds(component);
        
        Intent intent = new Intent(context, NoteWidgetProvider.class);
        intent.setAction(AppWidgetManager.ACTION_APPWIDGET_UPDATE);
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, widgetIds);
        context.sendBroadcast(intent);
    }
}
```

---

## 62.3 Widget Layout

```xml
<!-- res/layout/widget_note.xml -->
<!-- Widget supports only: FrameLayout, LinearLayout, RelativeLayout -->
<!-- And: TextView, ImageView, Button, ImageButton, ProgressBar, Chronometer, AnalogClock, ListView, GridView, StackView, AdapterViewFlipper -->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/widgetRoot"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@drawable/widget_background"
    android:padding="8dp">
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        
        <TextView
            android:id="@+id/tvWidgetTitle"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="My Notes"
            android:textStyle="bold"
            android:textSize="16sp" />
        
        <ImageButton
            android:id="@+id/btnRefresh"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/ic_refresh"
            android:background="?attr/selectableItemBackgroundBorderless" />
        
        <ImageButton
            android:id="@+id/btnNewNote"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/ic_add"
            android:background="?attr/selectableItemBackgroundBorderless" />
    </LinearLayout>
    
    <TextView
        android:id="@+id/tvNoteCount"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Loading..."
        android:textSize="14sp" />

</LinearLayout>
```

---

## 62.4 Widget Info (AppWidgetProviderInfo)

```xml
<!-- res/xml/widget_note_info.xml -->
<appwidget-provider
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="110dp"
    android:minHeight="40dp"
    android:targetCellWidth="2"
    android:targetCellHeight="1"
    android:updatePeriodMillis="1800000"
    android:previewLayout="@layout/widget_note"
    android:initialLayout="@layout/widget_note"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen"
    android:description="@string/widget_description" />
```

```xml
<!-- AndroidManifest.xml - register widget -->
<receiver
    android:name=".widget.NoteWidgetProvider"
    android:exported="true"
    android:label="Note Widget">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        <action android:name="com.myapp.widget.REFRESH" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/widget_note_info" />
</receiver>
```

---

## 62.5 Collection Widget (ListView)

```java
// widget/TaskListWidgetProvider.java
public class TaskListWidgetProvider extends AppWidgetProvider {
    
    @Override
    public void onUpdate(Context context, AppWidgetManager manager, int[] widgetIds) {
        for (int id : widgetIds) {
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_task_list);
            
            // Set up RemoteViews adapter for ListView
            Intent serviceIntent = new Intent(context, TaskWidgetService.class);
            serviceIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, id);
            serviceIntent.setData(Uri.parse(serviceIntent.toUri(Intent.URI_INTENT_SCHEME)));
            
            views.setRemoteAdapter(R.id.lvTasks, serviceIntent);
            views.setEmptyView(R.id.lvTasks, R.id.tvEmpty);
            
            // Template for item click
            Intent clickIntent = new Intent(context, TaskDetailActivity.class);
            PendingIntent clickPendingIntent = PendingIntent.getActivity(context, 0,
                clickIntent, PendingIntent.FLAG_MUTABLE);
            views.setPendingIntentTemplate(R.id.lvTasks, clickPendingIntent);
            
            manager.updateAppWidget(id, views);
            manager.notifyAppWidgetViewDataChanged(id, R.id.lvTasks);
        }
    }
}

// TaskWidgetService.java - provides data to ListView
public class TaskWidgetService extends RemoteViewsService {
    
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        int widgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
            AppWidgetManager.INVALID_APPWIDGET_ID);
        return new TaskListRemoteViewsFactory(getApplicationContext(), widgetId);
    }
    
    static class TaskListRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {
        
        private final Context context;
        private List<Task> tasks = new ArrayList<>();
        
        TaskListRemoteViewsFactory(Context context, int widgetId) {
            this.context = context;
        }
        
        @Override
        public void onCreate() { loadTasks(); }
        
        @Override
        public void onDataSetChanged() { loadTasks(); }
        
        private void loadTasks() {
            // Load from Room or other source (runs on background thread)
            TaskDao dao = AppDatabase.getInstance(context).taskDao();
            tasks = dao.getAllTasksSync();
        }
        
        @Override
        public RemoteViews getViewAt(int position) {
            Task task = tasks.get(position);
            RemoteViews view = new RemoteViews(context.getPackageName(), R.layout.widget_item_task);
            view.setTextViewText(R.id.tvTaskTitle, task.getTitle());
            
            // Fill-in intent (for item click)
            Intent fillIn = new Intent();
            fillIn.putExtra("task_id", task.getId());
            view.setOnClickFillInIntent(R.id.tvTaskTitle, fillIn);
            
            return view;
        }
        
        @Override public int getCount()               { return tasks.size(); }
        @Override public long getItemId(int pos)      { return tasks.get(pos).getId(); }
        @Override public boolean hasStableIds()       { return true; }
        @Override public RemoteViews getLoadingView() { return null; }
        @Override public int getViewTypeCount()       { return 1; }
        @Override public void onDestroy()             {}
    }
}
```

---

## 62.6 สรุป Part 62

ในบทนี้คุณได้เรียนรู้:

✅ App Widget concepts (information, collection, control)  
✅ AppWidgetProvider (onUpdate, onReceive)  
✅ RemoteViews (การ set UI จาก widget)  
✅ PendingIntent สำหรับ widget clicks  
✅ Widget layout limitations  
✅ AppWidgetProviderInfo XML (size, update period)  
✅ Collection Widget (ListView ใน widget)  
✅ RemoteViewsService + RemoteViewsFactory  

---

*[← Part 61: Multi-Module](./part-61-android-multi-module.md) | [Part 63: MotionLayout →](./part-63-android-motionlayout.md)*
