# Android Native Changes

## File 1: BackgroundGeolocationService.java

### Line 18: Add import
```java
import com.google.android.gms.location.Priority;
```

### Lines 75-87: Replace addWatcher method signature and LocationRequest creation

**REPLACE THIS:**
```java
void addWatcher(
        final String id,
        Notification backgroundNotification,
        float distanceFilter
) {
    FusedLocationProviderClient client = LocationServices.getFusedLocationProviderClient(
            BackgroundGeolocationService.this
    );
    LocationRequest locationRequest = new LocationRequest();
    locationRequest.setMaxWaitTime(1000);
    locationRequest.setInterval(1000);
    locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
    locationRequest.setSmallestDisplacement(distanceFilter);
```

**WITH THIS:**
```java
void addWatcher(
        final String id,
        Notification backgroundNotification,
        float distanceFilter,
        int accuracy
) {
    FusedLocationProviderClient client = LocationServices.getFusedLocationProviderClient(
            BackgroundGeolocationService.this
    );

    // Map accuracy enum to Android Priority
    int priority;
    long interval;

    switch (accuracy) {
        case 100: // HIGH
            priority = Priority.PRIORITY_HIGH_ACCURACY;
            interval = 5000; // 5 seconds
            break;
        case 102: // BALANCED (Default)
            priority = Priority.PRIORITY_BALANCED_POWER_ACCURACY;
            interval = 10000; // 10 seconds
            break;
        case 104: // LOW
            priority = Priority.PRIORITY_LOW_POWER;
            interval = 30000; // 30 seconds
            break;
        case 105: // PASSIVE
            priority = Priority.PRIORITY_PASSIVE;
            interval = 60000; // 60 seconds
            break;
        default:
            priority = Priority.PRIORITY_BALANCED_POWER_ACCURACY;
            interval = 10000;
    }

    // Use modern LocationRequest.Builder API
    LocationRequest locationRequest = new LocationRequest.Builder(priority, interval)
            .setMinUpdateDistanceMeters(distanceFilter)
            .setWaitForAccurateLocation(false)
            .setMaxUpdateDelayMillis(interval)
            .build();
```

## File 2: BackgroundGeolocation.java

Need to read this file first to see where to add the accuracy parameter.
