# Camera HUD

A lightweight location, clock, and weather overlay for your webcam—built as a single HTML file and shared in Microsoft Teams through OBS Studio.

```text
📍 Houston, TX
🕒 10:46 AM CDT
☀️ 85°F  Clear
```

*Illustrative display; location, time, and weather depend on your settings and the data returned.*

OBS combines your webcam and the transparent overlay into one video feed. Teams receives that feed through **OBS Virtual Camera**.

```text
Webcam + camera-hud.html → OBS scene → OBS Virtual Camera → Teams
```

## Features

- Automatic location from your public IP address, or a fixed place name or coordinates.
- A local-time clock with 12-hour and 24-hour formats and a time-zone indicator.
- Temperature in Fahrenheit or Celsius, weather descriptions, and day/night weather icons.
- Four corner positions, adjustable size, and a translucent background.
- A browser preview with a gray background; a transparent canvas inside OBS.
- No build process, package installation, or API key configuration in the HTML.

This guide covers the original **`camera-hud.html`**, including its default 6% margins and 66% card-background opacity.

## Requirements

- [OBS Studio](https://obsproject.com/) with Browser Source and Virtual Camera available.
- A webcam and permission for OBS to access it.
- Microsoft Teams with access to the virtual camera.
- An internet connection for location and weather requests.

Camera HUD is the overlay. OBS supplies webcam capture, compositing, and the virtual camera. Third-party services have their own usage limits and terms; no API key configuration does not imply unrestricted service use.

## Set up OBS

### 1. Download and preview the HTML

Download or clone this repository and keep `camera-hud.html` in a stable local folder. If downloading the individual file from GitHub, save the **raw HTML**, rather than the GitHub page that displays its code.

Double-click the file to preview it in your browser. The gray gradient is intentional outside OBS. Location and weather need network access; a failed first load is retried after one minute.

### 2. Prepare a 1920 × 1080 scene

1. Open OBS and create a scene named **Camera HUD**.
2. In **Settings → Video**, set **Base (Canvas) Resolution** to **1920 × 1080**. Use the same **Output (Scaled) Resolution** for this setup.
3. In **Sources**, select **+ → Video Capture Device**, choose your webcam, and confirm that it appears.
4. Right-click the webcam source and select **Transform → Fit to Screen**. Adjust framing as needed.

Source names can vary by platform. See the official [OBS video capture documentation](https://obsproject.com/kb/video-capture-sources).

### 3. Add the overlay as a local Browser Source

In **Sources**, select **+ → Browser**, name it **Camera HUD Overlay**, and use:

| Property | Value |
| --- | --- |
| Local file | Enabled |
| Local file path | Your downloaded `camera-hud.html` |
| Width | `1920` |
| Height | `1080` |

Confirm the properties. Keep the Browser Source fitted to the full canvas; its transparent area lets the webcam show through. The HTML positions the card within that canvas.

For continuous updates between scenes, leave **Shutdown source when not visible** off. Leaving **Refresh browser source when scene becomes active** off also avoids repeated startup lookups. See [OBS Browser Source properties](https://obsproject.com/kb/browser-source).

### 4. Check source stacking

OBS draws sources higher in the list over sources below them. Arrange them like this, with both visibility icons enabled:

```text
Camera HUD Overlay      ← top
Webcam                  ← below
```

The card should appear at the bottom-right of your webcam image. Lock the sources when the layout looks right.

## Send the scene to Microsoft Teams

1. In OBS, select **Start Virtual Camera** in the **Controls** dock.
2. If the virtual camera is configured for a single source, change its output to **Program**, or select the complete **Camera HUD** scene. Both the webcam and overlay must be included.
3. Open Teams and go to **Settings and more (…) → Settings → Devices**.
4. Under the camera settings, choose **OBS Virtual Camera** and check the preview.
5. Turn on your camera when joining the meeting. Keep OBS and its virtual camera running throughout the call.

Select your microphone and speakers separately in Teams; this setup supplies video. You do not need to start an OBS recording or stream.

Official references: [OBS Virtual Camera guide](https://obsproject.com/kb/virtual-camera-guide) and [Teams device settings](https://support.microsoft.com/en-us/teams/notifications-settings/manage-your-device-settings-in-microsoft-teams).

## Configuration

Open `camera-hud.html` in a plain-text editor and find `const CONFIG`. The original defaults are:

```js
const CONFIG = {
  location: "auto",
  label: "",
  units: "F",
  clock24h: false,
  timeZone: "location",
  position: "bottom-right",
  scale: 1,

  show: {
    location: true,
    time: true,
    temperature: true,
    conditions: true,
  },

  weatherRefreshMinutes: 15,
  locationRefreshMinutes: 60,
};
```

Save your changes, then open the Browser Source properties in OBS and click **Refresh cache of current page**.

### Location and label

| Setting | Behavior |
| --- | --- |
| `location: "auto"` | Estimates your location from your public IP address. VPNs and corporate networks can change the result. |
| `location: "Houston, Texas"` | Looks up a fixed place name. |
| `location: "29.7604, -95.3698"` | Uses fixed latitude and longitude. Set a label too: coordinates are not reverse-geocoded into a place name. |
| `label: "Houston, TX"` | Replaces only the displayed location text. Weather and time still use the resolved location. |

For a fixed Houston display, change these two entries:

```js
location: "29.7604, -95.3698",
label: "Houston, TX",
```

Place-name lookup searches the name before the first comma and tries to match the remainder against the returned region or country. It falls back to the first result when no qualifier matches. Use coordinates when a name is ambiguous.

### Clock, units, and visible rows

| Setting | Options and notes |
| --- | --- |
| `units` | `"F"` or `"C"`; temperature is rounded to a whole degree. |
| `clock24h` | `false` for 12-hour time; `true` for 24-hour time. |
| `timeZone` | `"location"` uses the time zone returned by the weather request; `"system"` uses the computer's time zone. |
| `show.location` | Shows or hides the location row. |
| `show.time` | Shows or hides the clock row. |
| `show.temperature` | Shows or hides the entire weather row, including its icon and description. |
| `show.conditions` | Shows or hides the weather description next to the temperature. |

Until a successful weather response supplies a location time zone, the clock uses the system time zone. Time formatting and the time-zone indicator depend on the browser's locale. Hiding rows affects presentation; it does **not** stop location or weather requests.

### Position and size

Set `position` to `"top-left"`, `"top-right"`, `"bottom-left"`, or `"bottom-right"`.

Set `scale` to a positive number: `0.8` is smaller, `1` is the original size, and `1.3` is larger. This scales the font and the card's spacing and corners together.

Prefer these controls when the Browser Source fills the canvas. Moving or resizing the source in OBS transforms its entire 1920 × 1080 viewport, including the text.

### CSS appearance controls

The `:root` block near the top of the file contains:

```css
:root {
  --scale: 1;
  --font: calc(3.1vh * var(--scale));
  --margin-x: 6vw;
  --margin-y: 6vh;
  --card-bg: rgba(14, 16, 20, 0.66);
  --card-border: rgba(255, 255, 255, 0.14);
  --text: #ffffff;
}
```

| Control | What it changes |
| --- | --- |
| `--font` | Font size relative to viewport height; about 33.5 px at 1080p and scale `1`. |
| `--margin-x` | Horizontal inset; `6vw` is 6% of the viewport width. |
| `--margin-y` | Vertical inset; `6vh` is 6% of the viewport height. |
| `--card-bg` | Card color and opacity. The last number ranges from `0` to `1`. |
| `--card-border` | Border color and opacity. |
| `--text` | Text color. |
| `#hud` → `padding` | Internal space around the rows. |
| `#hud` → `gap`, `line-height` | Row spacing and text line height. |
| `#hud` → `border-radius` | Corner rounding. |

JavaScript sets `--scale` from `CONFIG.scale`, so change the configuration entry for overall scaling. The card sizes itself around its content; its rows do not wrap. Longer labels may require a smaller scale.

For optional styling, change `--card-bg` to `rgba(14, 16, 20, 0.75)` for a darker card, or reduce the margins to move it closer to the edges. Keep enough inset for meeting layouts that crop the video.

The earlier illustrative `.overlay` CSS is not needed: this file uses **`#hud`**, corner classes, and the variables above.

### Optional URL overrides

When loading the page through a URL, these query parameters override the configuration: `location`, `label`, `units`, `pos`, `scale`, and `h24` (`1` for 24-hour time, `0` for 12-hour time).

```text
camera-hud.html?location=Austin%2C%20Texas&units=C&pos=top-left&scale=0.8&h24=1
```

This illustrates the URL syntax. For the OBS **Local file** workflow above, edit `CONFIG`; do not append query text to the file-picker path.

## Data sources and update timing

| Purpose | Provider and endpoint |
| --- | --- |
| Automatic IP location | [ipapi.co](https://ipapi.co/api/) — `https://ipapi.co/json/` |
| Fallback IP location | [IPWhois](https://ipwhois.io/documentation) — `https://ipwho.is/` |
| Place-name lookup | [Open-Meteo Geocoding](https://open-meteo.com/en/docs/geocoding-api) — `https://geocoding-api.open-meteo.com/v1/search` |
| Weather and location time zone | [Open-Meteo Weather](https://open-meteo.com/en/docs) — `https://api.open-meteo.com/v1/forecast` |

The weather request asks for `temperature_2m`, `weather_code`, and `is_day`, with `timezone=auto`. The HTML maps weather codes to descriptions and emoji. Open-Meteo's current conditions are model-based data, rather than a reading from a sensor at your webcam. See its [weather documentation](https://open-meteo.com/en/docs).

| Activity | Original behavior |
| --- | --- |
| Clock | Checked every second; displays hours and minutes. |
| Initial location and weather | Requested at startup. A failed startup cycle is retried after a one-minute wait until both succeed. |
| Weather refresh | Every 15 minutes after the first successful cycle. |
| Automatic location refresh | Every 60 minutes after startup succeeds; a successful lookup also triggers a weather refresh. |
| Fixed place or coordinates | Resolved during startup; no hourly location refresh. |

Change `weatherRefreshMinutes` and `locationRefreshMinutes` to positive values to adjust the intervals. Timers run while the page is loaded; reloading starts the sequence again. Requests can fail because of connectivity, browser restrictions, provider outages, or rate limits. Check each provider's current terms and limits before relying on the service.

After a successful reading, a failed weather refresh leaves the last temperature and conditions visible. There is **no stale-data badge or last-updated timestamp**.

## Privacy

- Webcam capture and overlay compositing happen in OBS on your computer. The HTML does not capture your camera or microphone and does not send webcam frames to the location or weather providers.
- The overlay makes HTTPS requests directly from your computer. Providers receive your public IP address and normal request metadata. Place-name lookup sends the search name; weather lookup sends the resolved coordinates.
- Automatic location uses IP geolocation, not GPS. A fixed place or coordinates skips the IP-location services, but Open-Meteo still receives requests and sees the connection's public IP address.
- A custom `label` changes visible text only. Hiding location or weather rows does not disable their network lookups.
- Meeting participants can see the location, time, and weather included in your outgoing video. Teams handles transmission of that composed video.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Webcam appears, but no overlay | Put the Browser Source above the webcam, enable its visibility, verify the local file path, and refresh it. Check that rows are enabled in `CONFIG.show`. |
| Gray background covers the webcam | Gray is expected in a normal browser. In OBS, load the HTML directly as a Browser Source; the page detects `window.obsstudio` to enable transparency. |
| Weather is missing | The weather row stays hidden until a reading succeeds. Check internet access, provider access, and `show.temperature`. Preview the HTML in a browser and inspect its developer console for request errors. |
| Weather seems stuck | Failed updates preserve the last reading. Check connectivity and refresh the source. Startup retries every minute; later weather failures wait for another scheduled refresh. |
| Location is wrong | VPNs, proxies, and corporate networks can change IP-location results. Use a fixed place or coordinates. Automatic changes may take up to the next hourly lookup. |
| Location row is blank with coordinates | Set `label`; coordinate input does not look up a display name. |
| Time zone is wrong | Check `timeZone` and the resolved place. Location time needs a successful weather response; otherwise the clock uses system time. |
| Text is clipped or too large | Match the Browser Source to the 1920 × 1080 canvas, fit it to the screen, shorten the label, or reduce `scale`. |
| Teams crops the card | Increase `--margin-x` or `--margin-y`, or reduce the scale. Check the outgoing layout before a meeting. |
| Saved edits do not appear | Confirm you edited the file OBS is loading, then refresh the Browser Source. Use a plain-text editor and check for missing quotes, commas, or braces. |
| Teams shows only the webcam or only the overlay | Check the virtual camera output: use Program or the complete scene. Confirm Teams has selected OBS Virtual Camera. |
| OBS Virtual Camera is missing | Start it in OBS, restart Teams, and check camera permissions or organizational restrictions. Follow the official troubleshooting guide linked below. |
| Webcam is black in OBS | Confirm the capture device, grant camera permission, and close other applications that may be using the physical webcam. |
| Text looks mirrored in your self-view | Check the view seen by another participant before flipping the whole scene. |

For virtual camera installation and macOS extension approval, use the [OBS Virtual Camera troubleshooting guide](https://obsproject.com/kb/virtual-camera-troubleshooting). On macOS 15 and later, OBS documents camera-extension controls under **System Settings → General → Login Items & Extensions → Camera Extensions**.
