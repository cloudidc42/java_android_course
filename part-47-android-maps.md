# Part 47: Google Maps และ Location Services
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 47.1 Setup Google Maps

```groovy
// build.gradle (app)
implementation 'com.google.android.gms:play-services-maps:18.2.0'
implementation 'com.google.android.gms:play-services-location:21.0.1'
```

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

<application ...>
    <meta-data
        android:name="com.google.android.geo.API_KEY"
        android:value="YOUR_API_KEY_HERE"/>
</application>
```

```xml
<!-- activity_maps.xml -->
<fragment
    android:id="@+id/map"
    android:name="com.google.android.gms.maps.SupportMapFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

---

## 47.2 Basic Map Activity

```java
// MapsActivity.java
import com.google.android.gms.maps.*;
import com.google.android.gms.maps.model.*;

public class MapsActivity extends AppCompatActivity implements OnMapReadyCallback {
    
    private GoogleMap mMap;
    private FusedLocationProviderClient fusedClient;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_maps);
        
        fusedClient = LocationServices.getFusedLocationProviderClient(this);
        
        SupportMapFragment mapFragment = (SupportMapFragment)
            getSupportFragmentManager().findFragmentById(R.id.map);
        if (mapFragment != null) {
            mapFragment.getMapAsync(this);
        }
    }
    
    @Override
    public void onMapReady(GoogleMap googleMap) {
        mMap = googleMap;
        
        // Default location: Bangkok
        LatLng bangkok = new LatLng(13.7563, 100.5018);
        
        // Add marker
        mMap.addMarker(new MarkerOptions()
            .position(bangkok)
            .title("กรุงเทพมหานคร")
            .snippet("เมืองหลวงของประเทศไทย")
            .icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_RED)));
        
        // Move camera
        mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(bangkok, 12f));
        
        // Map settings
        UiSettings ui = mMap.getUiSettings();
        ui.setZoomControlsEnabled(true);
        ui.setCompassEnabled(true);
        ui.setMyLocationButtonEnabled(true);
        
        // Map type
        mMap.setMapType(GoogleMap.MAP_TYPE_NORMAL);  // SATELLITE, HYBRID, TERRAIN
        
        // Show user location
        showMyLocation();
        
        // Click listener
        mMap.setOnMarkerClickListener(marker -> {
            Toast.makeText(this, "คลิก: " + marker.getTitle(), Toast.LENGTH_SHORT).show();
            return false;  // false = still show info window
        });
        
        // Long click: add marker
        mMap.setOnMapLongClickListener(latLng -> {
            addMarkerAt(latLng, "จุดใหม่", "");
        });
    }
    
    @SuppressLint("MissingPermission")
    private void showMyLocation() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
                == PackageManager.PERMISSION_GRANTED) {
            mMap.setMyLocationEnabled(true);
            
            // Get last location
            fusedClient.getLastLocation().addOnSuccessListener(location -> {
                if (location != null) {
                    LatLng myLoc = new LatLng(location.getLatitude(), location.getLongitude());
                    mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(myLoc, 15f));
                }
            });
        }
    }
    
    public MarkerOptions addMarkerAt(LatLng latLng, String title, String snippet) {
        MarkerOptions options = new MarkerOptions()
            .position(latLng)
            .title(title)
            .snippet(snippet);
        mMap.addMarker(options);
        return options;
    }
}
```

---

## 47.3 Custom Marker และ InfoWindow

```java
// Custom marker icon
public Marker addCustomMarker(LatLng position, String title) {
    // Create icon from resource
    BitmapDescriptor icon = BitmapDescriptorFactory.fromResource(R.drawable.ic_marker_custom);
    
    // Or create programmatically
    Bitmap markerBitmap = createMarkerBitmap("A");
    BitmapDescriptor customIcon = BitmapDescriptorFactory.fromBitmap(markerBitmap);
    
    MarkerOptions options = new MarkerOptions()
        .position(position)
        .title(title)
        .icon(customIcon)
        .anchor(0.5f, 1.0f);  // bottom-center
    
    return mMap.addMarker(options);
}

private Bitmap createMarkerBitmap(String text) {
    int size = 80;
    Bitmap bitmap = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(bitmap);
    
    Paint circlePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    circlePaint.setColor(Color.parseColor("#FF5722"));
    canvas.drawCircle(size/2f, size/2f, size/2f - 2, circlePaint);
    
    Paint textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    textPaint.setColor(Color.WHITE);
    textPaint.setTextSize(36f);
    textPaint.setTextAlign(Paint.Align.CENTER);
    textPaint.setTypeface(Typeface.DEFAULT_BOLD);
    
    float y = size/2f - (textPaint.descent() + textPaint.ascent()) / 2;
    canvas.drawText(text, size/2f, y, textPaint);
    
    return bitmap;
}

// Custom InfoWindow
mMap.setInfoWindowAdapter(new GoogleMap.InfoWindowAdapter() {
    @Override
    public View getInfoWindow(Marker marker) {
        return null;  // use default window shape
    }
    
    @Override
    public View getInfoContents(Marker marker) {
        View view = getLayoutInflater().inflate(R.layout.map_info_window, null);
        ((TextView) view.findViewById(R.id.tvTitle)).setText(marker.getTitle());
        ((TextView) view.findViewById(R.id.tvSnippet)).setText(marker.getSnippet());
        return view;
    }
});
```

---

## 47.4 Polyline และ Polygon

```java
// Draw route (polyline)
public void drawRoute(List<LatLng> points) {
    PolylineOptions options = new PolylineOptions()
        .addAll(points)
        .width(8f)
        .color(Color.parseColor("#2196F3"))
        .geodesic(true);  // follow earth curvature
    
    mMap.addPolyline(options);
}

// Draw area (polygon)
public void drawArea(List<LatLng> boundary) {
    PolygonOptions options = new PolygonOptions()
        .addAll(boundary)
        .strokeColor(Color.parseColor("#F44336"))
        .strokeWidth(4f)
        .fillColor(Color.parseColor("#20F44336"));  // semi-transparent
    
    mMap.addPolygon(options);
}

// Draw circle
public void drawCircle(LatLng center, double radiusMeters) {
    CircleOptions options = new CircleOptions()
        .center(center)
        .radius(radiusMeters)
        .strokeColor(Color.parseColor("#2196F3"))
        .strokeWidth(4f)
        .fillColor(Color.parseColor("#202196F3"));
    
    mMap.addCircle(options);
}
```

---

## 47.5 Location Tracking (Continuous Updates)

```java
// LocationTracker.java
public class LocationTracker {
    
    private FusedLocationProviderClient fusedClient;
    private LocationCallback locationCallback;
    private boolean isTracking = false;
    
    public interface LocationListener {
        void onLocationChanged(Location location);
    }
    
    public LocationTracker(Context context) {
        fusedClient = LocationServices.getFusedLocationProviderClient(context);
    }
    
    @SuppressLint("MissingPermission")
    public void startTracking(LocationListener listener) {
        if (isTracking) return;
        
        LocationRequest request = new LocationRequest.Builder(
                Priority.PRIORITY_HIGH_ACCURACY, 5000)  // update every 5 seconds
            .setMinUpdateIntervalMillis(2000)
            .setMinUpdateDistanceMeters(10f)  // only update if moved 10m
            .build();
        
        locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(LocationResult result) {
                Location location = result.getLastLocation();
                if (location != null) {
                    listener.onLocationChanged(location);
                }
            }
        };
        
        fusedClient.requestLocationUpdates(request, locationCallback,
            Looper.getMainLooper());
        isTracking = true;
    }
    
    public void stopTracking() {
        if (isTracking && locationCallback != null) {
            fusedClient.removeLocationUpdates(locationCallback);
            isTracking = false;
        }
    }
    
    // Calculate distance between two points
    public static float distanceBetween(LatLng from, LatLng to) {
        float[] results = new float[1];
        Location.distanceBetween(
            from.latitude, from.longitude,
            to.latitude, to.longitude,
            results);
        return results[0];  // meters
    }
}
```

---

## 47.6 Geocoding (Address ↔ LatLng)

```java
// GeocodingHelper.java
public class GeocodingHelper {
    
    // Address → LatLng
    public static LatLng geocode(Context context, String address) {
        Geocoder geocoder = new Geocoder(context, Locale.getDefault());
        try {
            List<Address> addresses = geocoder.getFromLocationName(address, 1);
            if (addresses != null && !addresses.isEmpty()) {
                Address addr = addresses.get(0);
                return new LatLng(addr.getLatitude(), addr.getLongitude());
            }
        } catch (IOException e) {
            Log.e("Geocoding", "Error: " + e.getMessage());
        }
        return null;
    }
    
    // LatLng → Address
    public static String reverseGeocode(Context context, double lat, double lng) {
        Geocoder geocoder = new Geocoder(context, Locale.getDefault());
        try {
            List<Address> addresses = geocoder.getFromLocation(lat, lng, 1);
            if (addresses != null && !addresses.isEmpty()) {
                Address addr = addresses.get(0);
                return addr.getAddressLine(0);
            }
        } catch (IOException e) {
            Log.e("Geocoding", "Error: " + e.getMessage());
        }
        return null;
    }
    
    // Get full address details
    public static String formatAddress(Address address) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i <= address.getMaxAddressLineIndex(); i++) {
            sb.append(address.getAddressLine(i));
            if (i < address.getMaxAddressLineIndex()) sb.append(", ");
        }
        return sb.toString();
    }
}
```

---

## 47.7 สรุป Part 47

ในบทนี้คุณได้เรียนรู้:

✅ Google Maps SDK setup (API key, manifest)  
✅ Map initialization (onMapReady)  
✅ Add markers, move camera  
✅ Map types, UI settings  
✅ Custom marker icon  
✅ Custom InfoWindow  
✅ Polyline, Polygon, Circle  
✅ FusedLocationProvider (last location, continuous)  
✅ Geocoding (address ↔ coordinates)  
✅ Distance calculation  

---

*[← Part 46: Camera](./part-46-android-camera.md) | [Part 48: Firebase →](./part-48-android-firebase.md)*
