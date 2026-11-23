# Daylight BLE dashboard PWA (v3.3.0)

The ESP32 does not host an HTTP web page. This folder is the offline-capable dashboard interface; it communicates with the ESP32 over Bluetooth Low Energy GATT.

## Install once

1. Publish the **contents** of this folder to a static HTTPS host (GitHub Pages is suitable).
2. Open the HTTPS URL while online in a current Chrome or Edge browser.
3. Wait for the page to show **Offline app ready**. The service worker caches the app shell.
4. Use the browser's Install app / Add to Home Screen command and open the installed app once.

## Use outdoors

1. Power the ESP32. It starts advertising immediately as `Daylight-Todo`; it does not start Wi-Fi.
2. Open the installed Daylight app.
3. Press **Connect Bluetooth** and select `Daylight-Todo`.
4. On connection, the PWA sends the current UTC epoch from the phone/computer to the ESP32. The firmware then displays and schedules time in Saudi Arabia (UTC+3).
5. You may connect a second central (for example, a PC while the iPhone stays connected). Both clients share the ESP32 API; writes are serialized and response traffic is routed to the requesting client.
6. Use events, repeating activities, the reward shop, logs, CSV exports and Pomodoro as usual.

The clock continues running on the ESP32 after the phone disconnects, but without a battery-backed RTC it is lost after the ESP32 is powered off. Connect the PWA again after the next boot to synchronise time.

## Browser support

Web Bluetooth requires a secure HTTPS origin and a browser that implements Web Bluetooth. Use current Chrome or Edge on desktop or Android. On iPhone, use Safari with the Beacio extension enabled; Beacio supplies Web Bluetooth support on iOS. The interface itself is cached offline, but the first install/update requires internet access to the HTTPS host.

## Reliability

- GATT responses are delivered in ordered chunks. The browser acknowledges each response chunk before the ESP32 sends the next one.
- Activity log pages are bound to a ledger revision. If a task is completed or a Pomodoro entry is added during history loading, the page series is restarted against a fresh snapshot.
- The browser retries a timed-out GET once while the Bluetooth connection is still active. It does not auto-retry writes or purchases.
- The connection indicator is red when disconnected and green when GATT is connected.

## BLE UUIDs

- Service: `7d2ea28a-f7bd-485a-bd9d-92ad6ecfe93e`
- RX (browser writes requests/ACKs): `7d2ea28b-f7bd-485a-bd9d-92ad6ecfe93e`
- TX (ESP32 notifies API responses): `7d2ea28c-f7bd-485a-bd9d-92ad6ecfe93e`
- ALERT (ESP32 notifies task start/end events): `7d2ea28d-f7bd-485a-bd9d-92ad6ecfe93e`

Keep these UUIDs in sync with `include/project_config.h` in the firmware project.

## Notifications

- **iPhone / Beacio:** Connect Daylight, then press **Enable notifications** in the web app. Grant Beacio background notification permission. The page requests a background connection and registers the task-alert characteristic. Beacio can then show native iOS notifications while Safari is backgrounded, subject to iOS/Beacio background delivery behaviour and the registration's 30-day lifetime.
- **PC:** Press **Enable notifications** and allow browser notifications. The PWA must remain open (it may be in the background) and the BLE connection must remain alive. Closing the browser or disconnecting BLE stops direct characteristic notifications; this firmware does not use a cloud push service.
- Both clients can subscribe to the same alert characteristic. The ESP32 broadcasts task-start and timed-task-end alerts to all subscribed centrals.

## Refresh / reconnect

On startup the PWA calls `navigator.bluetooth.getDevices()` and looks for a previously granted `Daylight-Todo` device. It reuses an already-connected GATT object where available, otherwise it reconnects to the saved device without requiring the user to toggle Bluetooth. The browser may still require a user tap on platforms that restrict background GATT reconnection.

The service worker cache is versioned; republish this folder over the same HTTPS origin and reload once while online to install this release.
