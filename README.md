# Daylight BLE dashboard client

This folder contains the browser app for the ESP32 BLE firmware. The ESP32 does **not** serve a normal HTTP website over Bluetooth. Instead, it exposes the Daylight API through BLE GATT; this PWA is hosted on HTTPS and cached on your phone/computer so the interface can open offline.

## Browser compatibility

Use current **Google Chrome or Microsoft Edge** on Windows, macOS, Linux or Android. Web Bluetooth requires a secure HTTPS origin and browser support. Firefox and Safari (including iPhone/iPad Safari) do not provide the required Web Bluetooth API.

## Publish/install once while online

1. Create a repository for the dashboard or choose another static HTTPS host.
2. Upload the **contents** of this folder (`index.html`, `manifest.webmanifest`, `service-worker.js`, `icon-192.png`, and `icon-512.png`) to the published site root.
3. Enable HTTPS. GitHub Pages is a straightforward static host.
4. Open the published URL in Chrome or Edge while online. Wait until the header says **Offline app ready**.
5. Use the browser's **Install app** / **Add to Home screen** command. Launch the installed Daylight app once to confirm it opens.

The service worker caches the app shell. It does not cache device data: events, logs, shop and Pomodoro data are read from the ESP32 over BLE.

## Use outdoors

1. Power the ESP32. At boot it attempts the configured Wi-Fi network for up to `BLE_WIFI_CONNECT_TIMEOUT_MS` to obtain a Saudi Arabia (UTC+3) clock. If Wi-Fi connects, it waits up to `BLE_NTP_WAIT_TIMEOUT_MS` for NTP.
2. The firmware switches Wi-Fi off and starts BLE advertising as **Daylight-Todo**. The HTTP server is not started.
3. Open the installed Daylight app and press **Connect Bluetooth**. Select `Daylight-Todo` in the browser's device chooser.
4. The existing dashboard features then communicate over GATT. Persistent records remain on ESP32 LittleFS.

## Clock limitation

The ESP32 obtains time from NTP at boot. If it boots away from the configured Wi-Fi, BLE still starts after the Wi-Fi timeout. When the browser first connects, the app automatically supplies its current Unix time if the ESP32 clock is still invalid. The app assumes the phone/computer clock is correct. After this one-time fallback sync, time-based alerts, Pomodoro and timestamps work normally. A battery-backed RTC (for example, DS3231) is still the most reliable solution when the ESP32 must keep accurate time before a phone connects.

## Radio and antenna behaviour

The ESP32-WROOM shares its 2.4 GHz radio/antenna path between Wi-Fi and Bluetooth. The firmware explicitly calls `WiFi.mode(WIFI_OFF)` before starting NimBLE. The normal WROOM module does not need a separate antenna-switch GPIO. Once BLE starts, the ESP32 has no usable Wi-Fi IP address or HTTP website; the PWA talks to the GATT service.

## Privacy

The GATT service currently accepts compatible nearby devices without a pairing PIN. Keep the ESP32 physically controlled. Add BLE pairing/passkey protection before using it in a public or shared environment.
