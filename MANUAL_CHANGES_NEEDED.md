# Manuelle Änderungen erforderlich

Da die Dateien zu groß für automatische Änderungen sind, musst du folgende 3 Änderungen manuell durchführen:

## ✅ Bereits erledigt:
- TypeScript Definitions (definitions.d.ts) mit LocationAccuracy enum ✓

## 🔧 Manuelle Änderungen:

### 1. Android: BackgroundGeolocationService.java (2 Änderungen)

**Datei öffnen**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocationService.java`

**Änderung 1** - Zeile 18, nach den anderen Location-Imports:
```java
import com.google.android.gms.location.Priority;
```

**Änderung 2** - Zeilen 75-87 komplett ersetzen:

**LÖSCHE DIESE ZEILEN (75-87):**
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

**ERSETZE DURCH:**
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

---

### 2. Android: BackgroundGeolocation.java (1 Änderung)

**Datei öffnen**: `android/src/main/java/com/equimaps/capacitor_background_geolocation/BackgroundGeolocation.java`

**Änderung** - Zeilen 159-163:

**LÖSCHE DIESE ZEILEN:**
```java
        service.addWatcher(
                call.getCallbackId(),
                backgroundNotification,
                call.getFloat("distanceFilter", 0f)
        );
```

**ERSETZE DURCH:**
```java
        service.addWatcher(
                call.getCallbackId(),
                backgroundNotification,
                call.getFloat("distanceFilter", 0f),
                call.getInt("accuracy", 102)
        );
```

---

### 3. iOS: Plugin.swift (1 Änderung)

**Datei öffnen**: `ios/Plugin/Swift/Plugin.swift`

**Änderung** - Zeilen 95-105:

**LÖSCHE DIESE ZEILEN:**
```swift
            let manager = watcher.locationManager
            manager.delegate = self
            let externalPower = [
                .full,
                .charging
            ].contains(UIDevice.current.batteryState)
            manager.desiredAccuracy = (
                externalPower
                ? kCLLocationAccuracyBestForNavigation
                : kCLLocationAccuracyBest
            )
```

**ERSETZE DURCH:**
```swift
            let manager = watcher.locationManager
            manager.delegate = self

            // Configure accuracy based on parameter (default: BALANCED = 102)
            let accuracy = call.getInt("accuracy") ?? 102

            switch accuracy {
            case 100: // HIGH
                manager.desiredAccuracy = kCLLocationAccuracyBest
                manager.activityType = .otherNavigation
            case 102: // BALANCED (Default)
                manager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
                manager.activityType = .other
            case 104: // LOW
                manager.desiredAccuracy = kCLLocationAccuracyHundredMeters
                manager.activityType = .other
            case 105: // PASSIVE
                manager.desiredAccuracy = kCLLocationAccuracyThreeKilometers
                manager.activityType = .other
            default:
                manager.desiredAccuracy = kCLLocationAccuracyNearestTenMeters
                manager.activityType = .other
            }
```

---

## 📤 Nach den Änderungen:

```bash
cd /c/JavaSD/workspace/background-geolocation

# Änderungen committen
git add .
git commit -m "feat: Add location accuracy configuration for battery optimization

- Add LocationAccuracy enum (HIGH, BALANCED, LOW, PASSIVE)
- Android: Use modern LocationRequest.Builder API with Priority
- iOS: Configure CLLocationManager desiredAccuracy based on parameter
- Default accuracy: BALANCED (102) for optimal battery/accuracy trade-off"

# Zu GitHub pushen
git push origin master
```

## 🚀 Plugin einbinden:

```bash
cd /c/JavaSD/workspace/silayoluneu/frontend
npm install github:AzaTek61/background-geolocation
npx cap sync
```

---

## ✨ Verwendung:

```typescript
import { LocationAccuracy } from '@capacitor-community/background-geolocation';

// Hohe Genauigkeit - GPS (Hoher Akkuverbrauch)
await locationTracker.start(LocationAccuracy.HIGH);

// Ausgeglichen - GPS+Netzwerk (Mittlerer Akkuverbrauch)
await locationTracker.start(LocationAccuracy.BALANCED);

// Niedrig - Netzwerk (Niedriger Akkuverbrauch)
await locationTracker.start(LocationAccuracy.LOW);

// Passiv - Minimal (Minimaler Akkuverbrauch)
await locationTracker.start(LocationAccuracy.PASSIVE);
```
