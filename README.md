# qiblatfinder 📍🕌

A cross‑platform mobile application (iOS + Android) that helps Muslims accurately determine the Qiblah direction and prayer times based on GPS location.

---

## ✨ Features

- **Dynamic Compass**
  - Uses the phone’s built‑in magnetometer.
  - Displays N, S, E, W markers with a red North needle.
  - Fixed stop arrow at the top of the dial.
  - When aligned with Kaabah direction, shows popup: *"You facing Kaabah"*.

- **Prayer Times**
  - Accurate calculation of Fajr, Dhuhr, Asr, Maghrib, Isha.
  - Based on GPS location and date.
  - Displays location name, date, and times.
  - Supports dark mode and bright mode.

- **Settings**
  - Configure alarms/notifications for each prayer.
  - Toggle between dark/bright themes.
  - Choose time format (12h/24h).

---

## 🛠 Tech Stack

- **Framework:** Flutter (Dart) or React Native
- **Compass:** `flutter_compass` / `react-native-sensors`
- **Prayer Times:** `adhan` library or equivalent
- **GPS/Location:** `geolocator` / `react-native-geolocation-service`

---

## 📐 Accuracy

- Qiblah direction calculated using azimuth formulas relative to Kaabah coordinates:
  - **Latitude:** 21.4225° N
  - **Longitude:** 39.8262° E
- Prayer times follow established astronomical methods (Muslim World League, Umm al‑Qura, ISNA).

---

## 📂 Project Structure

