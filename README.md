[English](https://github.com/aromajoin/controller-http-api) / [日本語](README-JP.md)

# Aroma Shooter HTTP APIs

[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/aromajoin-software/)

Aromajoin's HTTP APIs let you control an Aroma Shooter over Wi-Fi.

There are two families of endpoints:

- **`/asn/*` — generation-neutral (recommended).** Available on firmware **2.2.0 and later**. These work across the Aroma Shooter series and every response includes the device `serial`.
- **`/as2/*` — legacy.** The original endpoint paths, kept for backward compatibility with existing integrations. Still supported.

Using this API requires a Wi-Fi connection.

## I. Wi-Fi connection setup

There are two methods of connecting an Aroma Shooter to a Wi-Fi network: via Aromajoin's official iOS application or via a web browser.
**Notes: In order to control Aroma Shooter, you have to setup WiFi and connect Aroma Shooter to the router. It could be not controlled in Hotspot mode.**

### Recommended Option - Web Browser

Here's an alternative method for people without iOS devices.

After plugging in your Aroma Shooter to a power source, choose it from the available Wi-Fi networks on your computer or mobile device. It will be identifiable by its serial number (For example: "ASN3A05216").

- Password: aromajoin@1003

Using a web browser on your device, navigate to this address: `http://192.168.1.1/`

From the list of Wi-Fi networks, choose your preferred local network and enter your password. After about 30 seconds, the Aroma Shooter will connect to your local network. Please wait for the success message before refreshing or navigating away from the page. After you receive a message confirming a successful connection, tap the name of the Wi-Fi network again and take note of your device's IP Address -- you'll need this to send requests in Part II. You may now reconnect your computer/phone to the local network. It's time to try sending requests.

### Option 2 - Aroma Shooter iOS app

The Aroma Shooter app is available on the Apple App Store:

https://apps.apple.com/app/aroma-shooter/id1477144583

After you install the app, tap the settings button in the bottom left corner of the screen, tap "Others," then tap "Setup AromaShooter's Wi-Fi," and follow the firmware version on-screen prompts. On the screen titled "Aroma Shooter Wi-Fi Connection," choose your preferred local Wi-Fi network and enter your credentials. After you receive a message confirming a successful connection, tap the name of the Wi-Fi network again and take note of your device's IP Address -- you'll need this to send requests in Part II. Tap "OK" then "Done" to return to the settings menu.

You can also confirm the connection state at a glance from the device's LED — see [Part IV. LED status indicators](#iv-led-status-indicators).

## II. APIs

Now your Aroma Shooter is connected to a network through which it may receive HTTP requests, as long as you send the requests from a device on the same network. Using your preferred REST tools, submit requests via the following formats.

**Hostname:** `http://[Aroma-Shooter_IP-Address]` or `http://[Device-serial].local`

**Port:** 1003

The hostname structure should match one of these formats. Please do not copy these examples, as you must modify the IP addresses and/or serial numbers according to your Aroma Shooter(s):

- IP address: `http://192.168.1.10:1003` (This format is **recommended**, since it handles requests very quickly.)

- Device serial: `http://ASN3A00001.local:1003` (This format may seem intuitive, but it handles requests slowly and is incompatible with Android devices.)

> **CORS / browser apps:** every endpoint answers the `OPTIONS` preflight and returns permissive CORS headers, so you can call the API directly from a browser with `Content-Type: application/json`.

## III. `/asn/*` endpoints (recommended, firmware 2.2.0+)

These endpoints are generation-neutral and always return the device `serial`, so a client controlling several units always knows which one replied.

> **Note on `"received"`:** shoot responds `"received"`, not `"done"`. HTTP is a synchronous request/response, but diffusion continues after the response is sent — the device confirms it _accepted and started_ the command. A missing response (HTTP error or timeout) means the command was **not** accepted.

### 1. Shoot scents

- _Path:_ /asn/shoot

- _Method:_ POST

- _Header:_ “Content-Type: application/json”

- _Request body:_

```javascript
{
    "chambers": [Number, ...],      // Cartridge (chamber) number. Range: 1 ~ 6
    "concentrations": [Number, ...],// Scent concentration per chamber, as a percentage. Range: 0 ~ 100
    "duration": Number,             // Common diffusion time for all chambers, in ms. Range: 0 ~ 10000
    "internalBoosterIntensity": Number, // Required. Internal booster fan intensity, as a percentage. Range: 0 ~ 100.
    "externalBoosterIntensity": Number  // Optional. External booster fan intensity 0 ~ 100 (0/omitted = off). AS3 only (parsed but ignored on AS2).
}
```

> **`internalBoosterIntensity` is required.** The internal booster fan is what pushes the diffused scent out of the device — omit it (or set it to `0`) and no scent is emitted, even with valid `chambers`/`concentrations`. `externalBoosterIntensity` is optional (AS3 only).

Instead of a common `"duration"`, you may give per-chamber times with `"durations"` (array, same length/order as `"chambers"`):

```javascript
{
    "chambers": [1, 3, 5],
    "concentrations": [100, 50, 25],
    "durations": [1000, 2000, 3000],
    "internalBoosterIntensity": 100
}
```

_Request sample:_

```json
{
    "chambers": [1, 3],
    "concentrations": [75, 50],
    "duration": 3000,
    "internalBoosterIntensity": 100
}
```

- _Response sample:_

```json
{
    "serial": "ASN3A00001",
    "status": "received"
}
```

- _Error (malformed body):_ HTTP `400 Bad Request`

```json
{
    "status": "invalid"
}
```

### 2. Get device identity

- _Path:_ /asn/identity

- _Method:_ GET

- _Header:_ “Content-Type: application/json”

- _Response sample:_ (side-effect free — safe for discovery/health checks)

```json
{
    "serial": "ASN3A00001"
}
```

### 3. Get firmware version

- _Path:_ /asn/version

- _Method:_ GET

- _Header:_ “Content-Type: application/json”

- _Response sample:_ (the version comes from the firmware image itself, not a hardcoded string)

```json
{
    "serial": "ASN3A00001",
    "firmware": "2.2.0"
}
```

## IV. `/as2/*` endpoints (legacy, backward-compatible)

These remain available for existing integrations. On firmware 2.2.0 and later their responses also include the `serial`.

### 1. Get firmware information

- _Path:_ /as2/firmware

- _Method:_ GET

- _Header:_ “Content-Type: application/json”

- _Response sample:_

```json
{
    "serial": "ASN3A00001",
    "current": "1.0.0",
    "latest": "1.0.1",
    "internet": "true"
}
```

> On firmware **1.x.x** the `"serial"` field is not present.

### 2. Diffuse scents

- _Path:_ /as2/diffuse

- _Method:_ POST

- _Header:_ “Content-Type: application/json”

- _Request body:_

**Firmware version 2.x.x and later**

```javascript
{
    "channels": [Number, ...], // The cartridge number. Range: 1 ~ 6
    "intensities": [Number, ...], // The cartridge intensity as a percentage. Range: 0 ~ 100
    "durations": [Number, ...], // Diffusion time in milliseconds. Range: 0 ~ 10000
    "booster": Boolean // Set to true to activate the Aroma Shooter's booster fan. Default value is false.
}
```

_Request sample:_

```json
{
    "channels": [1, 3, 5],
    "intensities": [100, 50, 25],
    "durations": [1000, 2000, 3000],
    "booster": true
}
```

**Firmware version 1.x.x**

```javascript
{
    "duration": Number, // Diffusion time in milliseconds. Range: 0 ~ 10000
    "channel": Number, // The cartridge number. Range: 1 ~ 6
    "intensity": Number, // The cartridge intensity as a percentage. Range: 0 ~ 100
    "booster": Boolean // Set to true to activate the Aroma Shooter's booster fan. Default value is false.
}
```

```json
{
    "duration": 3000,
    "channel": 3,
    "intensity": 100,
    "booster": true
}
```

- _Response sample:_

```json
{
    "serial": "ASN3A00001",
    "status": "done"
}
```

> On firmware **1.x.x** the response is `{"status": "done"}` (no `"serial"`).

### 3. Stop diffusing

- _Path:_ /as2/stop_all

- _Method:_ POST

- _Header:_ “Content-Type: application/json”

- _Request body:_ None

- _Response sample:_

**Firmware version 2.x.x and later**

```json
{
    "serial": "ASN3A00001",
    "status": "done"
}
```

**Firmware version 1.x.x**

```json
{
    "status": "done"
}
```

## V. LED status indicators (firmware 2.2.0+)

The two front LEDs signal the connection state so you can diagnose it without a console:

| State                          | LED      | Pattern                          |
| ------------------------------ | -------- | -------------------------------- |
| Wi-Fi connected                | 🟢 Green | Blinks 3× (0.5 s on / 0.5 s off) |
| Bluetooth (BLE) connected      | 🔵 Blue  | Blinks 3× (0.5 s on / 0.5 s off) |
| Disconnected / connection lost | 🔴 Red   | Blinks 2× (short, ~0.15 s)       |

Red is reserved for the "disconnected / error" state — a successful connection is always green (Wi-Fi) or blue (BLE).

> **Changed in firmware 2.2.0.** This colour scheme was revised in 2.2.0: a successful **Wi-Fi** connection now blinks **green** (it blinked **red** on earlier firmware), and the **red disconnect / connection-lost** indicator is **new**. On firmware before 2.2.0 the Wi-Fi-connect blink was red and there was no dedicated disconnect indicator.

## FAQs

### Why I could not see Aroma Shooter’s Wi-Fi hotspot?

To save the device energy and resource, the hotspot will be automatically disabled in two following scenarios.
If Aroma Shooter is already connected to a Wi-Fi network, the hotspot is automatically off.
If Aroma Shooter is being controlled via USB or Bluetooth, the hotspot and Wi-Fi connection ability is disabled.
Therefore, to set up Wi-Fi for Aroma Shooter, it is necessary to turn it off and on again if it is controlled via PCs or smart phones.

### How can I confirm if Aroma Shooter is already connected to a Wi-Fi router?

The simplest way is to use IP scanning software on a PC or mobile phone that is connected to the same Wi-Fi router. It will help you to scan all the IP addresses in the same network, so you can check if Aroma Shooter IP address is listed there. If it is listed there, it means that it is connected successfully.
For example, if you are using Windows PC, you can use [Advanced IP Scanner](https://www.advanced-ip-scanner.com/).

You can also read the connection state from the device LED — see [Part V. LED status indicators](#v-led-status-indicators).

### Does the Wi-Fi settings are kept in case of power down?

Yes. The WiFi settings are kept even the power is down. It only changes when you connect the device to another network.

### How long the Wi-Fi information is retained?

It is retained forever and changes only when the device is connected to another network.

### How can I change the Wi-Fi settings?

When you bring the device to other places where the device can not connect to the last connected Wi-Fi access point, the device will automatically turn on its Wi-Fi hotspot. Then, you can connect and set up Wi-Fi settings again.

---

Copyright 2020 Aromajoin Corporation under [CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/) license.
