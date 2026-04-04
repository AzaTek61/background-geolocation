# Complete Implementation Guide - Background Geolocation mit Accuracy

## ✅ Was bereits erledigt ist:

1. **TypeScript Definitions** (`definitions.d.ts`) ✓
   - LocationAccuracy enum hinzugefügt
   - WatcherOptions.accuracy Parameter hinzugefügt

2. **Repository** ✓
   - Geforkt: https://github.com/AzaTek61/background-geolocation
   - Lokal geklont: C:\JavaSD\workspace\background-geolocation

## 🔧 Manuelle Änderungen erforderlich:

### 1. Android: BackgroundGeolocationService.java

**Datei**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocationService.java`

**Schritt 1**: Zeile 18 - Import hinzufügen:
```java
import com.google.android.gms.location.Priority;
```

**Schritt 2**: Zeile 75-88 - Methoden-Signatur ändern:

ERSETZE:
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

DURCH:
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

### 2. Android: BackgroundGeolocation.java

**Datei**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocation.java`

**Zeile 159-163** - Accuracy-Parameter hinzufügen:

ERSETZE:
```java
service.addWatcher(
        call.getCallbackId(),
        backgroundNotification,
        call.getFloat("distanceFilter", 0f)
);
```

DURCH:
```java
service.addWatcher(
        call.getCallbackId(),
        backgroundNotification,
        call.getFloat("distanceFilter", 0f),
        call.getInt("accuracy", 102)  // Default: BALANCED
);
```

### 3. iOS: Plugin.swift

**Datei**: `ios/Plugin/Swift/Plugin.swift`

In der `addWatcher` Methode, nach dem Auslesen der anderen Parameter:

```swift
@objc func addWatcher(_ call: CAPPluginCall) {
    // ... existing code ...

    // Get accuracy (default to BALANCED = 102)
    let accuracy = call.getInt("accuracy") ?? 102

    // Configure location manager accuracy
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

## 🚀 Nach den Änderungen:

### 1. Plugin zu GitHub pushen:
```bash
cd /c/JavaSD/workspace/background-geolocation
git add .
git commit -m "feat: Add location accuracy configuration for battery optimization"
git push origin main
```

### 2. In silayoluneu-Projekt einbinden:
```bash
cd /c/JavaSD/workspace/silayoluneu/frontend
npm install github:AzaTek61/background-geolocation
npx cap sync
```

### 3. LocationTrackerService erweitern:
```typescript
// frontend/src/app/services/location-tracker.service.ts
import { LocationAccuracy } from '@capacitor-community/background-geolocation';

export { LocationAccuracy } from '@capacitor-community/background-geolocation';

async start(accuracy: LocationAccuracy = LocationAccuracy.BALANCED) {
    // ...
    this.watcherId = await BackgroundGeolocation.addWatcher(
        {
            backgroundMessage: this.translateService.instant('home.geolocation.BG_MESSAGE'),
            backgroundTitle: this.translateService.instant('home.geolocation.BG_TITLE'),
            requestPermissions: true,
            stale: false,
            distanceFilter: 50,
            accuracy: accuracy  // NEU!
        },
        // ...
    );
}
```

### 4. Verwendung im Code:
```typescript
// Hohe Genauigkeit (GPS) - Hoher Akkuverbrauch
await locationTracker.start(LocationAccuracy.HIGH);

// Ausgeglichen (Standard) - Mittlerer Akkuverbrauch
await locationTracker.start(LocationAccuracy.BALANCED);

// Niedrig (Netzwerk) - Niedriger Akkuverbrauch
await locationTracker.start(LocationAccuracy.LOW);

// Passiv - Minimaler Akkuverbrauch
await locationTracker.start(LocationAccuracy.PASSIVE);
```

## 📊 Akkuverbrauch-Vergleich:

| Accuracy | Provider | Intervall | Genauigkeit | Akkuverbrauch |
|----------|----------|-----------|-------------|---------------|
| HIGH     | GPS      | 5s        | 5-10m       | ⚡⚡⚡⚡ Hoch    |
| BALANCED | GPS+Netz | 10s       | 10-20m      | ⚡⚡⚡ Mittel   |
| LOW      | Netzwerk | 30s       | 50-100m     | ⚡⚡ Niedrig   |
| PASSIVE  | Passiv   | 60s       | 100-1000m   | ⚡ Minimal    |

