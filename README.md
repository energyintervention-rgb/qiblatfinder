# 🧭 The Qiblat

A single-file progressive web app for **Qiblah direction** and **daily prayer times**. No install, no dependencies, no server — drop one HTML file anywhere and open it.

---

## Live demo

Host `the_qiblat.html` on any HTTPS server or open it locally. GitHub Pages works out of the box.

---

## Features at a glance

| Tab | What it does |
|-----|-------------|
| 🧭 **Qiblah** | Live compass driven by the device magnetometer. Rotating dial with N/S/E/W markers, red North needle, fixed gold stop-arrow at the top, and a green Qiblah arc that glows when you face Makkah. Popup alert fires at ±3°. |
| 🕋 **Prayer** | GPS-located prayer times (Fajr, Dhuhr, Asr, Maghrib, Isha) for today. Highlights the current and next prayer. Per-prayer alarm toggle using the Web Notifications API. |
| ⚙️ **Settings** | Dark / light theme, 12h / 24h time format, individual prayer alarm switches, and five calculation method choices. All preferences persist in `localStorage`. |

---

## Screenshots

> Add screenshots to a `docs/` folder and link them here.

```
docs/
  compass-dark.png
  prayer-dark.png
  settings-dark.png
  compass-light.png
```

---

## Usage

### Option A — open locally (Android / desktop)

```
open the_qiblat.html          # macOS
start the_qiblat.html         # Windows
xdg-open the_qiblat.html      # Linux
```

### Option B — GitHub Pages

1. Push `the_qiblat.html` to a repository as `index.html`.
2. Enable **Settings → Pages → Deploy from branch**.
3. Open the generated `https://username.github.io/repo/` URL on your phone.

### Option C — local HTTPS (required for iOS compass)

```bash
# Python 3 one-liner
python3 -m http.server 8080
# Then open http://localhost:8080/the_qiblat.html
```

For iOS you need a real HTTPS origin. Use [ngrok](https://ngrok.com) or any static host.

---

## Browser & device support

| Feature | Chrome Android | Safari iOS | Firefox | Desktop Chrome |
|---------|---------------|-----------|---------|----------------|
| GPS location | ✅ | ✅ HTTPS only | ✅ | ✅ |
| Live compass | ✅ | ✅ HTTPS + permission prompt | ⚠️ partial | ❌ demo mode |
| Prayer times | ✅ | ✅ | ✅ | ✅ |
| Notifications | ✅ | ✅ iOS 16.4+ | ✅ | ✅ |
| Dark mode | ✅ | ✅ | ✅ | ✅ |

**Desktop / no sensor:** if no `deviceorientation` events arrive within 2 seconds, a slow demo rotation starts automatically so the UI is still usable.

---

## Permissions requested

| Permission | When | Why |
|------------|------|-----|
| Location | On first load | GPS coordinates for Qiblah azimuth and prayer times |
| Device orientation | On first load (iOS only shows a prompt) | Magnetometer heading for the live compass |
| Notifications | When you toggle an alarm on | Browser notification at prayer time |

No data leaves the device except two outbound requests:

- **Nominatim** (`nominatim.openstreetmap.org`) — reverse geocoding to resolve city name. Falls back to raw coordinates silently if the request fails.
- No analytics, no tracking, no third-party scripts.

---

## Qiblah calculation

Great-circle bearing formula, computed entirely in the browser:

```
y = sin(Δλ) · cos(φ_kaabah)
x = cos(φ_user) · sin(φ_kaabah) − sin(φ_user) · cos(φ_kaabah) · cos(Δλ)
bearing = atan2(y, x)   normalised to [0°, 360°)
```

Kaabah reference: **21.4225° N, 39.8262° E**

Alignment is triggered when the absolute difference between compass heading and Qiblah azimuth is ≤ 3°. The popup resets after the user rotates away and returns.

---

## Prayer time calculation

Pure-JS astronomical algorithm — no external library.

**Pipeline per prayer:**

1. Compute Julian Day for the current date.
2. Derive solar declination and equation of time from mean longitude and mean anomaly.
3. Calculate solar noon from longitude, equation of time, and local UTC offset.
4. Apply the hour angle formula for each prayer's depression angle or shadow ratio.

**Asr** uses the Shafi'i / standard shadow factor of 1 (object shadow equals object height).

**Dhuhr** is set 1 minute after solar noon to ensure the sun has passed the meridian.

**Umm al-Qura Isha** is 90 minutes after Maghrib instead of an angle.

### Calculation methods

| ID | Name | Fajr angle | Isha |
|----|------|-----------|------|
| `MWL` | Muslim World League | 18° | 17° |
| `ISNA` | ISNA (North America) | 15° | 15° |
| `UMQ` | Umm al-Qura, Makkah | 18.5° | 90 min after Maghrib |
| `EGY` | Egyptian General Survey | 19.5° | 17.5° |
| `KAR` | Karachi / Hanafi | 18° | 18° |

Default is **Muslim World League**, which is used across Malaysia, Southeast Asia, and most of Europe.

---

## Compass calibration

If the needle drifts or reads incorrectly:

1. Hold the phone flat and horizontal.
2. Slowly trace a figure-8 pattern in the air several times.
3. Tap **↺** (top-right of the Qiblah screen) to re-acquire GPS and recalculate the azimuth.

Magnetic interference from metal surfaces, speakers, or cases can affect accuracy. Move away from these when reading direction.

---

## Settings reference

| Setting | Default | Notes |
|---------|---------|-------|
| Dark mode | On | Toggle on Prayer screen (🌙/☀️) or in Settings |
| 24-hour time | Off | Applies to the prayer time display |
| Fajr alarm | Off | Browser notification at calculated Fajr time |
| Dhuhr alarm | Off | |
| Asr alarm | Off | |
| Maghrib alarm | Off | |
| Isha alarm | Off | |
| Calculation method | MWL | Changes prayer times; reloads prayer screen |

Settings are saved to `localStorage` under key `qiblat_prefs` and restored on every load.

---

## File structure

Everything is in one file. Internal layout:

```
the_qiblat.html
│
├── <style>
│   ├── CSS custom properties (dark + light token sets)
│   ├── App shell (screen, bottom-nav, topbar)
│   ├── Compass screen styles
│   ├── Prayer times screen styles
│   ├── Settings screen styles
│   └── Popup overlay + spinner
│
├── <body>
│   ├── #compass-screen   → canvas + stop-arrow + align banner
│   ├── #prayer-screen    → location header + prayer card list
│   ├── #settings-screen  → toggles + method radio options
│   ├── .bottom-nav       → three tab buttons
│   └── #aligned-overlay  → modal popup
│
└── <script>
    ├── State object S{}
    ├── Persistence (localStorage save/load)
    ├── Theme (applyTheme)
    ├── calcQiblah()         great-circle bearing
    ├── calcPrayerTimes()    Julian Day → solar position → hour angles
    ├── getLocation()        Geolocation API + Nominatim reverse geocode
    ├── drawCompass()        Canvas2D custom painter
    ├── startCompass()       deviceorientation listener + smoothing + demo fallback
    ├── checkAlignment()     ±3° detection + popup trigger
    ├── initCompass()        orchestrates location → azimuth → canvas
    ├── initPrayer()         orchestrates location → times → render
    ├── renderPrayerList()   DOM render + alarm toggle wiring
    ├── scheduleAlarms()     Web Notifications API setTimeout scheduling
    ├── getHijriDate()       Intl.DateTimeFormat islamic calendar (with tabular fallback)
    ├── buildSettings()      Renders alarm toggles + method radio options
    └── Navigation + event wiring
```

---

## Known limitations

- **Magnetic declination** — the compass reads magnetic North. In most of Malaysia, Southeast Asia, and the Middle East the declination is small (< 2°) and within the alignment tolerance. Extreme latitudes (> 60°) may see larger offsets.
- **High latitudes** — prayer times become undefined when the sun does not set or rise. The hour angle formula returns `null` in these cases and falls back to estimated offsets. A dedicated high-latitude rule (such as the nearest-latitude method) is not implemented.
- **Asr madhab** — calculation uses the Shafi'i shadow factor (1). Hanafi shadow factor (2) is not a separate toggle; selecting the Karachi method as a whole provides the closest approximation.
- **Notifications on iOS** — requires iOS 16.4+, must be added to the Home Screen as a PWA, and notifications must be enabled in device Settings.

---

## License

MIT — see [LICENSE](LICENSE)

---

## Acknowledgements

- Qiblah formula: standard geodetic great-circle bearing
- Prayer time algorithm: based on the method documented by the University of Islamic Sciences, Karachi, and cross-referenced with PrayTimes.org
- Reverse geocoding: [Nominatim / OpenStreetMap](https://nominatim.openstreetmap.org) (ODbL licence)
- Hijri date: `Intl.DateTimeFormat` with `calendar: 'islamic'`, tabular fallback for unsupported environments
