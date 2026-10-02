# Part 83: Bluetooth & NFC
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 83.1 Bluetooth Low Energy (BLE)

```java
// BleManager.java
public class BleManager {
    
    private final Context context;
    private BluetoothAdapter bluetoothAdapter;
    private BluetoothLeScanner bleScanner;
    private BluetoothGatt bluetoothGatt;
    private final List<BleDevice> foundDevices = new ArrayList<>();
    
    // Characteristic UUIDs (example: Heart Rate service)
    private static final UUID HR_SERVICE_UUID     = UUID.fromString("0000180D-0000-1000-8000-00805f9b34fb");
    private static final UUID HR_CHAR_UUID        = UUID.fromString("00002A37-0000-1000-8000-00805f9b34fb");
    private static final UUID CLIENT_CONFIG_UUID  = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");
    
    public interface BleCallback {
        void onDeviceFound(BleDevice device);
        void onConnected(BluetoothDevice device);
        void onDisconnected();
        void onDataReceived(String charUuid, byte[] data);
        void onError(String message);
    }
    
    private BleCallback callback;
    
    public BleManager(Context context) {
        this.context = context;
        BluetoothManager bm = (BluetoothManager) context.getSystemService(Context.BLUETOOTH_SERVICE);
        bluetoothAdapter = bm != null ? bm.getAdapter() : null;
    }
    
    public void setCallback(BleCallback cb) { this.callback = cb; }
    
    // --- Scanning ---
    
    @SuppressLint("MissingPermission")
    public void startScan() {
        if (bluetoothAdapter == null || !bluetoothAdapter.isEnabled()) {
            if (callback != null) callback.onError("Bluetooth not available");
            return;
        }
        
        bleScanner = bluetoothAdapter.getBluetoothLeScanner();
        
        ScanSettings settings = new ScanSettings.Builder()
            .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
            .build();
        
        // Filter by service UUID
        List<ScanFilter> filters = Collections.singletonList(
            new ScanFilter.Builder()
                .setServiceUuid(ParcelUuid.fromString(HR_SERVICE_UUID.toString()))
                .build());
        
        bleScanner.startScan(filters, settings, scanCallback);
        
        // Stop scan after 10 seconds
        new Handler(Looper.getMainLooper()).postDelayed(this::stopScan, 10_000);
    }
    
    @SuppressLint("MissingPermission")
    public void stopScan() {
        if (bleScanner != null) bleScanner.stopScan(scanCallback);
    }
    
    private final ScanCallback scanCallback = new ScanCallback() {
        @Override
        public void onScanResult(int callbackType, @NonNull ScanResult result) {
            BluetoothDevice device = result.getDevice();
            BleDevice bleDevice = new BleDevice(device, result.getRssi());
            
            if (!foundDevices.contains(bleDevice)) {
                foundDevices.add(bleDevice);
                if (callback != null) callback.onDeviceFound(bleDevice);
            }
        }
        
        @Override
        public void onScanFailed(int errorCode) {
            if (callback != null) callback.onError("Scan failed: " + errorCode);
        }
    };
    
    // --- Connection ---
    
    @SuppressLint("MissingPermission")
    public void connect(BluetoothDevice device) {
        bluetoothGatt = device.connectGatt(context, false, gattCallback,
            BluetoothDevice.TRANSPORT_LE);
    }
    
    @SuppressLint("MissingPermission")
    public void disconnect() {
        if (bluetoothGatt != null) {
            bluetoothGatt.disconnect();
            bluetoothGatt.close();
            bluetoothGatt = null;
        }
    }
    
    private final BluetoothGattCallback gattCallback = new BluetoothGattCallback() {
        
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
            if (newState == BluetoothProfile.STATE_CONNECTED) {
                Log.d("BLE", "Connected, discovering services...");
                gatt.discoverServices();
                if (callback != null) callback.onConnected(gatt.getDevice());
            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                if (callback != null) callback.onDisconnected();
            }
        }
        
        @Override
        @SuppressLint("MissingPermission")
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                BluetoothGattService service = gatt.getService(HR_SERVICE_UUID);
                if (service != null) {
                    enableNotification(gatt, service.getCharacteristic(HR_CHAR_UUID));
                }
            }
        }
        
        @Override
        public void onCharacteristicChanged(BluetoothGatt gatt,
                BluetoothGattCharacteristic characteristic) {
            byte[] data = characteristic.getValue();
            if (callback != null) {
                callback.onDataReceived(characteristic.getUuid().toString(), data);
            }
        }
        
        @Override
        public void onCharacteristicRead(BluetoothGatt gatt,
                BluetoothGattCharacteristic characteristic, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                byte[] data = characteristic.getValue();
                if (callback != null) {
                    callback.onDataReceived(characteristic.getUuid().toString(), data);
                }
            }
        }
    };
    
    @SuppressLint("MissingPermission")
    private void enableNotification(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
        gatt.setCharacteristicNotification(characteristic, true);
        
        BluetoothGattDescriptor descriptor = characteristic.getDescriptor(CLIENT_CONFIG_UUID);
        if (descriptor != null) {
            descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
            gatt.writeDescriptor(descriptor);
        }
    }
    
    @SuppressLint("MissingPermission")
    public void writeData(UUID serviceUuid, UUID charUuid, byte[] data) {
        if (bluetoothGatt == null) return;
        BluetoothGattService service = bluetoothGatt.getService(serviceUuid);
        if (service == null) return;
        
        BluetoothGattCharacteristic characteristic = service.getCharacteristic(charUuid);
        if (characteristic == null) return;
        
        characteristic.setValue(data);
        characteristic.setWriteType(BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT);
        bluetoothGatt.writeCharacteristic(characteristic);
    }
    
    // Parse heart rate from BLE data
    public static int parseHeartRate(byte[] data) {
        int flag = data[0] & 0xFF;
        if ((flag & 0x01) == 0) {
            return data[1] & 0xFF;  // 8-bit HR
        } else {
            return ((data[2] & 0xFF) << 8) | (data[1] & 0xFF);  // 16-bit HR
        }
    }
}
```

---

## 83.2 NFC

```java
// NfcManager.java
public class NfcManager {
    
    private final Activity activity;
    private final android.nfc.NfcAdapter nfcAdapter;
    
    public NfcManager(Activity activity) {
        this.activity = activity;
        nfcAdapter = android.nfc.NfcAdapter.getDefaultAdapter(activity);
    }
    
    public boolean isNfcAvailable() {
        return nfcAdapter != null;
    }
    
    public boolean isNfcEnabled() {
        return nfcAdapter != null && nfcAdapter.isEnabled();
    }
    
    // Enable foreground dispatch (receive tags when app is in foreground)
    public void enableForegroundDispatch() {
        if (nfcAdapter == null) return;
        
        Intent intent = new Intent(activity, activity.getClass())
            .addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            activity, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        String[][] techLists = new String[][] {
            new String[] { Ndef.class.getName() },
            new String[] { NdefFormatable.class.getName() }
        };
        
        nfcAdapter.enableForegroundDispatch(activity, pendingIntent, null, techLists);
    }
    
    public void disableForegroundDispatch() {
        if (nfcAdapter != null) nfcAdapter.disableForegroundDispatch(activity);
    }
    
    // Read NDEF from tag
    public static NdefRecord readNdefRecord(Tag tag) {
        Ndef ndef = Ndef.get(tag);
        if (ndef == null) return null;
        
        try {
            ndef.connect();
            NdefMessage message = ndef.getNdefMessage();
            ndef.close();
            
            if (message != null && message.getRecords().length > 0) {
                return message.getRecords()[0];
            }
        } catch (Exception e) {
            Log.e("NFC", "Read failed", e);
        }
        return null;
    }
    
    // Parse text record
    public static String parseTextRecord(NdefRecord record) {
        if (record.getTnf() != NdefRecord.TNF_WELL_KNOWN) return null;
        if (!Arrays.equals(record.getType(), NdefRecord.RTD_TEXT)) return null;
        
        byte[] payload = record.getPayload();
        int langCodeLen = payload[0] & 0x3F;
        String text = new String(payload, langCodeLen + 1,
            payload.length - langCodeLen - 1, StandardCharsets.UTF_8);
        return text;
    }
    
    // Write NDEF to tag
    public static boolean writeNdefText(Tag tag, String text) {
        Ndef ndef = Ndef.get(tag);
        if (ndef == null) return false;
        
        NdefRecord record = NdefRecord.createTextRecord("en", text);
        NdefMessage message = new NdefMessage(record);
        
        try {
            ndef.connect();
            if (!ndef.isWritable()) {
                ndef.close();
                return false;
            }
            if (ndef.getMaxSize() < message.toByteArray().length) {
                ndef.close();
                return false;
            }
            ndef.writeNdefMessage(message);
            ndef.close();
            return true;
        } catch (Exception e) {
            Log.e("NFC", "Write failed", e);
            return false;
        }
    }
    
    // Write URI record
    public static boolean writeNdefUri(Tag tag, String uri) {
        Ndef ndef = Ndef.get(tag);
        if (ndef == null) return false;
        
        NdefRecord record = NdefRecord.createUri(uri);
        NdefMessage message = new NdefMessage(record);
        
        try {
            ndef.connect();
            ndef.writeNdefMessage(message);
            ndef.close();
            return true;
        } catch (Exception e) {
            Log.e("NFC", "Write URI failed", e);
            return false;
        }
    }
    
    // Handle in Activity onNewIntent
    public static void handleNfcIntent(Intent intent, Consumer<String> onTextRead) {
        String action = intent.getAction();
        if (!android.nfc.NfcAdapter.ACTION_NDEF_DISCOVERED.equals(action)
            && !android.nfc.NfcAdapter.ACTION_TAG_DISCOVERED.equals(action)) return;
        
        Tag tag = intent.getParcelableExtra(android.nfc.NfcAdapter.EXTRA_TAG);
        if (tag == null) return;
        
        NdefRecord record = readNdefRecord(tag);
        if (record != null) {
            String text = parseTextRecord(record);
            if (text != null) onTextRead.accept(text);
        }
    }
}
```

---

## 83.3 สรุป Part 83

ในบทนี้คุณได้เรียนรู้:

✅ BLE Scanning (ScanFilter, ScanSettings)  
✅ GATT connection and service discovery  
✅ Enable BLE notifications  
✅ Write data to BLE characteristic  
✅ Parse heart rate BLE data  
✅ NFC foreground dispatch  
✅ Read/write NDEF text records  
✅ Read/write NFC URI records  

---

*[← Part 82: Camera & Media](./part-82-android-camera-media.md) | [Part 84: Java Streams & Collectors →](./part-84-java-streams-collectors.md)*
