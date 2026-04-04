# Background Geolocation Plugin - Required Changes

## ✅ Already Completed:
1. **TypeScript Definitions** (`definitions.d.ts`) - LocationAccuracy enum added
2. **Repository** forked to https://github.com/AzaTek61/background-geolocation

## 🔧 Remaining Changes:

### 1. Android - BackgroundGeolocationService.java

**File**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocationService.java`

**Change Line 75-87** from:
```java
void addWatcher(
        final String id,
        Notification backgroundNotification,
        float distanceFilter
) {
    //...
    LocationRequest locationRequest = new LocationRequest();
    locationRequest.setMaxWaitTime(1000);
    locationRequest.setInterval(1000);
    locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
    locationRequest.setSmallestDisplacement(distanceFilter);
```

**To**:
```java
void addWatcher(
        final String id,
        Notification backgroundNotification,
        float distanceFilter,
        int accuracy  // NEW parameter
) {
    // Map accuracy enum to Android Priority
    int priority;
    long interval;

    switch (accuracy) {
        case 100: // HIGH
            priority = com.google.android.gms.location.Priority.PRIORITY_HIGH_ACCURACY;
            interval = 5000; // 5 seconds
            break;
        case 102: // BALANCED (Default)
            priority = com.google.android.gms.location.Priority.PRIORITY_BALANCED_POWER_ACCURACY;
            interval = 10000; // 10 seconds
            break;
        case 104: // LOW
            priority = com.google.android.gms.location.Priority.PRIORITY_LOW_POWER;
            interval = 30000; // 30 seconds
            break;
        case 105: // PASSIVE
            priority = com.google.android.gms.location.Priority.PRIORITY_PASSIVE;
            interval = 60000; // 60 seconds
            break;
        default:
            priority = com.google.android.gms.location.Priority.PRIORITY_BALANCED_POWER_ACCURACY;
            interval = 10000;
    }

    // Use modern LocationRequest.Builder API
    LocationRequest locationRequest = new LocationRequest.Builder(priority, interval)
            .setMinUpdateDistanceMeters(distanceFilter)
            .setWaitForAccurateLocation(false)
            .build();
```

### 2. Android - BackgroundGeolocation.java

**File**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocation.java`

Find the `addWatcher` method and add accuracy parameter:

```java
// Get accuracy from options (default to BALANCED = 102)
int accuracy = call.getInt("accuracy", 102);

// Pass accuracy to service
localBinder.addWatcher(
    callbackId,
    notification,
    distanceFilter,
    accuracy  // NEW parameter
);
```

### 3. iOS - Plugin.swift

**File**: `ios/Plugin/Swift/Plugin.swift`

Add accuracy configuration:

```swift
@objc func addWatcher(_ call: CAPPluginCall) {
    // ... existing code ...

    // Get accuracy (default to BALANCED = 102)
    let accuracy = call.getInt("accuracy") ?? 102

    // Configure accuracy
    switch accuracy {
    case 100: // HIGH
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.activityType = .otherNavigation
    case 102: // BALANCED (Default)
        locationManager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
        locationManager.activityType = .other
    case 104: // LOW
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
        locationManager.activityType = .other
    case 105: // PASSIVE
        locationManager.desiredAccuracy = kCLLocationAccuracyThreeKilometers
        locationManager.activityType = .other
    default:
        locationManager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
        locationManager.activityType = .other
    }

    // ... rest of code ...
}
```

## 📝 Summary

The plugin needs to:
1. Accept `accuracy` parameter in `addWatcher` options
2. Map accuracy enum values (100, 102, 104, 105) to platform-specific priorities
3. Configure LocationRequest with the appropriate priority level

## 🎯 Battery Impact

- **HIGH (100)**: GPS only, updates every 5s, highest accuracy, highest battery drain
- **BALANCED (102)**: GPS + Network, updates every 10s, good accuracy, moderate battery
- **LOW (104)**: Network only, updates every 30s, lower accuracy, low battery drain
- **PASSIVE (105)**: Piggyback on other apps, updates every 60s, lowest battery

