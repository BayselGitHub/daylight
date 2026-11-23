# Daylight BLE dashboard PWA (v3.1.1)

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
5. Use events, repeating activities, the reward shop, logs, CSV exports and Pomodoro as usual.

The clock continues running on the ESP32 after the phone disconnects, but without a battery-backed RTC it is lost after the ESP32 is powered off. Connect the PWA again after the next boot to synchronise time.

## Browser support

Web Bluetooth requires a secure HTTPS origin and a browser that implements Web Bluetooth. Use current Chrome or Edge on desktop or Android. Firefox and Safari/iOS Safari do not support this workflow. The interface itself is cached offline, but the first install/update requires internet access to the HTTPS host.

## Reliability

- GATT responses are delivered in ordered chunks. The browser acknowledges each response chunk before the ESP32 sends the next one.
- Activity log pages are bound to a ledger revision. If a task is completed or a Pomodoro entry is added during history loading, the page series is restarted against a fresh snapshot.
- The browser retries a timed-out GET once while the Bluetooth connection is still active. It does not auto-retry writes or purchases.
- The connection indicator is red when disconnected and green when GATT is connected.

## BLE UUIDs

- Service: `7d2ea28a-f7bd-485a-bd9d-92ad6ecfe93e`
- RX (browser writes requests/ACKs): `7d2ea28b-f7bd-485a-bd9d-92ad6ecfe93e`
- TX (ESP32 notifies responses): `7d2ea28c-f7bd-485a-bd9d-92ad6ecfe93e`

Keep these UUIDs in sync with `include/project_config.h` in the firmware project.
