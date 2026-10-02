# Part 45: Content Providers และ File Storage
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 45.1 Content Provider คืออะไร

Content Provider ให้แอปอื่นเข้าถึงข้อมูลของเราผ่าน URI  
Android system ใช้ ContentProvider สำหรับ: Contacts, MediaStore, Calendar

```
URI format: content://authority/path/id
ตัวอย่าง:   content://com.android.contacts/contacts/1
```

---

## 45.2 อ่าน Contacts

```java
// ContactsHelper.java
import android.provider.ContactsContract;

public class ContactsHelper {
    
    public static List<Contact> getAllContacts(Context context) {
        List<Contact> contacts = new ArrayList<>();
        
        // Columns to retrieve
        String[] projection = {
            ContactsContract.Contacts._ID,
            ContactsContract.Contacts.DISPLAY_NAME_PRIMARY,
            ContactsContract.Contacts.HAS_PHONE_NUMBER,
            ContactsContract.Contacts.PHOTO_URI
        };
        
        // Sort by name
        String sortOrder = ContactsContract.Contacts.DISPLAY_NAME_PRIMARY + " ASC";
        
        try (Cursor cursor = context.getContentResolver().query(
                ContactsContract.Contacts.CONTENT_URI,
                projection,
                null, null,
                sortOrder)) {
            
            if (cursor == null) return contacts;
            
            while (cursor.moveToNext()) {
                long id = cursor.getLong(cursor.getColumnIndexOrThrow(
                    ContactsContract.Contacts._ID));
                String name = cursor.getString(cursor.getColumnIndexOrThrow(
                    ContactsContract.Contacts.DISPLAY_NAME_PRIMARY));
                boolean hasPhone = cursor.getInt(cursor.getColumnIndexOrThrow(
                    ContactsContract.Contacts.HAS_PHONE_NUMBER)) > 0;
                String photoUri = cursor.getString(cursor.getColumnIndexOrThrow(
                    ContactsContract.Contacts.PHOTO_URI));
                
                // Get phone numbers
                List<String> phones = new ArrayList<>();
                if (hasPhone) {
                    phones = getPhoneNumbers(context, id);
                }
                
                contacts.add(new Contact(id, name, phones, photoUri));
            }
        }
        
        return contacts;
    }
    
    private static List<String> getPhoneNumbers(Context context, long contactId) {
        List<String> phones = new ArrayList<>();
        
        String selection = ContactsContract.CommonDataKinds.Phone.CONTACT_ID + " = ?";
        String[] selArgs = {String.valueOf(contactId)};
        
        try (Cursor cursor = context.getContentResolver().query(
                ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                new String[]{ContactsContract.CommonDataKinds.Phone.NUMBER},
                selection, selArgs, null)) {
            
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    phones.add(cursor.getString(0));
                }
            }
        }
        
        return phones;
    }
    
    // Search by name
    public static List<Contact> searchContacts(Context context, String query) {
        List<Contact> results = new ArrayList<>();
        
        String selection = ContactsContract.Contacts.DISPLAY_NAME_PRIMARY + " LIKE ?";
        String[] selArgs = {"%" + query + "%"};
        
        try (Cursor cursor = context.getContentResolver().query(
                ContactsContract.Contacts.CONTENT_URI,
                new String[]{
                    ContactsContract.Contacts._ID,
                    ContactsContract.Contacts.DISPLAY_NAME_PRIMARY
                },
                selection, selArgs, null)) {
            
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    long id = cursor.getLong(0);
                    String name = cursor.getString(1);
                    results.add(new Contact(id, name, null, null));
                }
            }
        }
        
        return results;
    }
    
    // Model
    public static class Contact {
        public long id;
        public String name;
        public List<String> phones;
        public String photoUri;
        
        public Contact(long id, String name, List<String> phones, String photoUri) {
            this.id = id;
            this.name = name;
            this.phones = phones;
            this.photoUri = photoUri;
        }
    }
}
```

---

## 45.3 MediaStore - อ่านรูปภาพ

```java
// MediaHelper.java
import android.provider.MediaStore;

public class MediaHelper {
    
    public static List<MediaItem> getImages(Context context) {
        List<MediaItem> images = new ArrayList<>();
        
        String[] projection = {
            MediaStore.Images.Media._ID,
            MediaStore.Images.Media.DISPLAY_NAME,
            MediaStore.Images.Media.DATE_ADDED,
            MediaStore.Images.Media.SIZE,
            MediaStore.Images.Media.WIDTH,
            MediaStore.Images.Media.HEIGHT
        };
        
        String sortOrder = MediaStore.Images.Media.DATE_ADDED + " DESC";
        
        try (Cursor cursor = context.getContentResolver().query(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                projection,
                null, null,
                sortOrder)) {
            
            if (cursor == null) return images;
            
            int idCol   = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID);
            int nameCol = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME);
            int dateCol = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATE_ADDED);
            int sizeCol = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.SIZE);
            
            while (cursor.moveToNext()) {
                long id   = cursor.getLong(idCol);
                String name = cursor.getString(nameCol);
                long date   = cursor.getLong(dateCol);
                long size   = cursor.getLong(sizeCol);
                
                // Build content URI
                Uri contentUri = ContentUris.withAppendedId(
                    MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id);
                
                images.add(new MediaItem(id, name, contentUri, date, size));
            }
        }
        
        return images;
    }
    
    // Save image to MediaStore (Android 10+)
    public static Uri saveImageToMediaStore(Context context, Bitmap bitmap, String displayName) 
            throws IOException {
        
        ContentValues values = new ContentValues();
        values.put(MediaStore.Images.Media.DISPLAY_NAME, displayName);
        values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
        values.put(MediaStore.Images.Media.DATE_ADDED, System.currentTimeMillis() / 1000);
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            values.put(MediaStore.Images.Media.RELATIVE_PATH, 
                Environment.DIRECTORY_PICTURES + "/MyApp");
            values.put(MediaStore.Images.Media.IS_PENDING, 1);
        }
        
        Uri uri = context.getContentResolver().insert(
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
        
        if (uri != null) {
            try (OutputStream os = context.getContentResolver().openOutputStream(uri)) {
                bitmap.compress(Bitmap.CompressFormat.JPEG, 90, os);
            }
            
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                values.clear();
                values.put(MediaStore.Images.Media.IS_PENDING, 0);
                context.getContentResolver().update(uri, values, null, null);
            }
        }
        
        return uri;
    }
    
    public static class MediaItem {
        public long id;
        public String name;
        public Uri uri;
        public long dateAdded;
        public long size;
        
        public MediaItem(long id, String name, Uri uri, long dateAdded, long size) {
            this.id = id;
            this.name = name;
            this.uri = uri;
            this.dateAdded = dateAdded;
            this.size = size;
        }
    }
}
```

---

## 45.4 Storage Access Framework (SAF) - File Picker

```java
// FilePickerActivity.java
public class FilePickerActivity extends AppCompatActivity {
    
    // Open file picker
    private final ActivityResultLauncher<String[]> openFileLauncher =
        registerForActivityResult(
            new ActivityResultContracts.OpenDocument(),
            uri -> {
                if (uri != null) {
                    handleSelectedFile(uri);
                }
            });
    
    // Open image picker
    private final ActivityResultLauncher<String> pickImageLauncher =
        registerForActivityResult(
            new ActivityResultContracts.GetContent(),
            uri -> {
                if (uri != null) {
                    ImageView imageView = findViewById(R.id.imageView);
                    imageView.setImageURI(uri);
                }
            });
    
    // Create file
    private final ActivityResultLauncher<String> createFileLauncher =
        registerForActivityResult(
            new ActivityResultContracts.CreateDocument("text/plain"),
            uri -> {
                if (uri != null) {
                    writeTextToFile(uri, "Hello from MyApp!\nContent here.");
                }
            });
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_file_picker);
        
        // Open any file
        findViewById(R.id.btnOpenFile).setOnClickListener(v ->
            openFileLauncher.launch(new String[]{"*/*"}));
        
        // Open image
        findViewById(R.id.btnPickImage).setOnClickListener(v ->
            pickImageLauncher.launch("image/*"));
        
        // Create text file
        findViewById(R.id.btnCreateFile).setOnClickListener(v ->
            createFileLauncher.launch("myfile.txt"));
    }
    
    private void handleSelectedFile(Uri uri) {
        // Get file name
        String fileName = getFileNameFromUri(uri);
        long fileSize = getFileSizeFromUri(uri);
        
        Toast.makeText(this, fileName + " (" + fileSize + " bytes)", Toast.LENGTH_SHORT).show();
        
        // Read text content
        try (InputStream is = getContentResolver().openInputStream(uri);
             BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
            
            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line).append("\n");
            }
            
            TextView tvContent = findViewById(R.id.tvContent);
            tvContent.setText(sb.toString());
            
        } catch (IOException e) {
            Toast.makeText(this, "ไม่สามารถอ่านไฟล์ได้", Toast.LENGTH_SHORT).show();
        }
    }
    
    private String getFileNameFromUri(Uri uri) {
        String name = "unknown";
        try (Cursor cursor = getContentResolver().query(
                uri, null, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                int nameIdx = cursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                if (nameIdx >= 0) name = cursor.getString(nameIdx);
            }
        }
        return name;
    }
    
    private long getFileSizeFromUri(Uri uri) {
        long size = 0;
        try (Cursor cursor = getContentResolver().query(
                uri, null, null, null, null)) {
            if (cursor != null && cursor.moveToFirst()) {
                int sizeIdx = cursor.getColumnIndex(OpenableColumns.SIZE);
                if (sizeIdx >= 0 && !cursor.isNull(sizeIdx)) {
                    size = cursor.getLong(sizeIdx);
                }
            }
        }
        return size;
    }
    
    private void writeTextToFile(Uri uri, String content) {
        try (OutputStream os = getContentResolver().openOutputStream(uri);
             BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(os))) {
            writer.write(content);
            Toast.makeText(this, "บันทึกไฟล์สำเร็จ", Toast.LENGTH_SHORT).show();
        } catch (IOException e) {
            Toast.makeText(this, "บันทึกไม่สำเร็จ", Toast.LENGTH_SHORT).show();
        }
    }
}
```

---

## 45.5 Internal Storage (App Private Files)

```java
// InternalStorageHelper.java
public class InternalStorageHelper {
    
    // Save text to internal storage
    public static boolean saveText(Context context, String filename, String content) {
        try (FileOutputStream fos = context.openFileOutput(filename, Context.MODE_PRIVATE);
             BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(fos))) {
            writer.write(content);
            return true;
        } catch (IOException e) {
            return false;
        }
    }
    
    // Read text from internal storage
    public static String readText(Context context, String filename) {
        try (FileInputStream fis = context.openFileInput(filename);
             BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) sb.append(line).append("\n");
            return sb.toString();
        } catch (IOException e) {
            return null;
        }
    }
    
    // Save object as JSON
    public static <T> boolean saveObject(Context context, String filename, T obj) {
        String json = new Gson().toJson(obj);
        return saveText(context, filename, json);
    }
    
    // Read object from JSON
    public static <T> T readObject(Context context, String filename, Class<T> type) {
        String json = readText(context, filename);
        if (json == null) return null;
        return new Gson().fromJson(json, type);
    }
    
    // Delete file
    public static boolean deleteFile(Context context, String filename) {
        return context.deleteFile(filename);
    }
    
    // List all files
    public static String[] listFiles(Context context) {
        return context.fileList();
    }
    
    // Get file size
    public static long getFileSize(Context context, String filename) {
        File file = new File(context.getFilesDir(), filename);
        return file.exists() ? file.length() : -1;
    }
    
    // Cache directory (cleared when storage low)
    public static File getCacheDir(Context context) {
        return context.getCacheDir();
    }
}
```

---

## 45.6 Manifest สำหรับ Contacts และ MediaStore

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.READ_CONTACTS"/>
<uses-permission android:name="android.permission.WRITE_CONTACTS"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="28"/>

<!-- Android 13+ -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO"/>
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO"/>
```

---

## 45.7 สรุป Part 45

ในบทนี้คุณได้เรียนรู้:

✅ ContentProvider overview  
✅ Read Contacts (ContentResolver, Cursor)  
✅ MediaStore (query images, save image)  
✅ Storage Access Framework (file picker, create file)  
✅ Read/write files via URI  
✅ Internal Storage (openFileOutput/Input)  
✅ Cache directory  

---

*[← Part 44: Android Animations](./part-44-android-animations.md) | [Part 46: Camera และ Image →](./part-46-android-camera.md)*
