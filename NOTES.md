# Free My TV: what we did (Sharp, October 2026)

Notes from the session of 24 October 2026.

## Context

On 24 October 2026, Free My TV got a new home screen design, a shelf of my own apps and settings for the Sharp TV,
then it was installed on the TV.

| Item | Value |
| --- | --- |
| TV | Sharp Google TV (MTC, MediaTek), Android 14 |
| IP address | 192.168.2.77 (ADB on port 5555, already allowed) |
| Installed app | `io.freemytv`, updated on 24 October 2026 at 14:08 |
| Mode | Free My TV (the stock Google TV home is turned off; *Original* brings it back) |

## App shelf

The bottom shelf shows YouTube, Netflix, Canal+, Prime Video and a web browser, in that order. The same order is set in
the Android app (`HomeApps.SHELF`) and in the LG/Samsung version (`SHELF` in `web/home.js`).

| App | Change | Android package |
| --- | --- | --- |
| YouTube | moved first | `com.google.android.youtube.tv` |
| Netflix | moved second | `com.netflix.ninja` |
| Canal+ | added (black tile, style in `HomeView`) | `com.canal.android.canal` |
| Prime Video | unchanged | `com.amazon.amazonvideo.livingroom` |
| Browser | added: TCL Browser (also on Sharp), TV Bro or Puffin TV | `com.tcl.browser`, `com.phlox.tvwebbrowser`, `com.cloudmosa.puffinTV` |
| Hulu, Peacock | kept at the end of the list (not on the Sharp) | |
| Paramount+ | removed from the shelf, still in *All Apps* | `com.cbs.ott` |
| Disney+, Max, Apple TV, Globoplay, Spotify | removed from the shelf | |

These lists apply to every user of the project: keep them apart from the rest before offering anything to the public
project. On the TV, the shelf settings also pick the apps without touching the code.

## Logos and icons

Six white logos were added to `web/icons/`, cropped like the existing ones. They are used by the preview and by
Samsung TVs; on LG, the icons come from the TV.

| File | Content | Source (licence) |
| --- | --- | --- |
| `appletv.svg` | Apple TV | Simple Icons (CC0) |
| `plex.svg` | Plex | Simple Icons (CC0) |
| `crunchyroll.svg` | Crunchyroll | Simple Icons (CC0) |
| `paramount.svg` | Paramount+ | Simple Icons (CC0) |
| `browser.svg` | globe for the browser | Material Symbols (Apache 2.0) |
| `settings.svg` | settings gear | Material Symbols (Apache 2.0) |

- The preview's sample apps (`SAMPLE` in `web/tv.js`) use these logos, plus the existing Netflix, YouTube, Prime Video,
  Max and Spotify ones.
- No free logo available: Canal+, Disney+, Hulu, Peacock and Globoplay. Their tiles show the name as text.

## New home screen design

The "centred" layout was picked out of six mockups: the date, the time and the weather centred over a sky whose colour
follows the hour. It shows when there are no movie picks, that is without a TMDB key; with a key, the home screen shows
the picks over the first one's blurred photo.

- **Date**: in capitals, letter-spaced, above the time.
- **Time**: very large (330 px), centred, no AM/PM.
- **Weather**: icon, temperature, conditions, the day's high and low, centred under the time.
- **Background**: the sky of the moment, with a sun glow behind the clock. The colours are the ones already in the
  Android app (`Shaders.sky`): night, dawn, day, sunset, dusk, worked out from the city's sunrise and sunset. They
  change by themselves through the day.
- **Top of the screen**: "For You" and the tabs are hidden without picks; the up arrow goes straight to the round
  buttons (Live TV, HDMI, Settings).
- **Crypto prices** (Android, when turned on): centred under the weather.

Where it is in the code:

- Android: `drawClock` and `drawClockSky` in `HomeView.java`.
- LG/Samsung: `renderClock` in `web/home.js`, `Data.sky` in `web/data.js`, the `#clock` and `#sky .hour` styles in
  `web/home.css`, the structure in `web/index.html`.

## Dark theme and font

In dark theme, the sky keeps the hour's tint but gets much deeper: navy by day, burgundy at sunset, almost black at
night. Before, the sky stayed light even in dark theme.

- The sky is covered by a very dark veil (#05060A at 72%), which follows the day/night transition.
- The sun glow is 60% fainter in dark theme, so the centre doesn't turn grey.
- The app shelf and the round buttons turn to dark glass (existing behaviour).
- To turn it on on the TV: *Home Settings* → *Theme* → *Dark*. On *Automatic*, it turns dark at sunset.

The clock is in **Inter Thin** (weight 100), the same font as the rest of the app, thinner. Quicksand, Nunito and
M PLUS Rounded were tried too. On Android, a new `Ui.hairline` weight was added to `Ui.java`.

## Sharp TV support

Fourteen Sharp system packages were added to `HomeMode.PROTECTED`: Free My TV never turns them off, even in Free My TV
mode.

| Role | Packages |
| --- | --- |
| Bluetooth | `com.airoha.btd` |
| Remote keys | `com.mediatek.android.tv.mtc.globalkeyoverlay` |
| Live TV and inputs | `com.mediatek.wwtv.tvcenter.mtc.overlay`, `com.mediatek.wwtv.tvcenter.mtc_sharp.overlay`, `com.mediatek.tv.oneworld.tvcenteroverlay`, `com.mediatek.tvinput.overlay`, `com.mediatek.tv.inputplayeroverlay`, `com.mediatek.overlay.tvservice` |
| Network and web | `com.mediatek.android.tv.mdns.offload.overlay`, `com.mediatek.wwtv.webview` |
| Boot | `com.mediatek.bootlogoservice` |
| Factory menu | `com.mediatek.mtc.factory` |
| Manual | `com.mtc.emanual`, `com.mtc.emanual.overlay.ok` |

TCL Browser (`com.tcl.browser`) is on the Sharp too: it is the browser on the shelf. On the TV,
`com.mtc.emanual.overlay.ok` and `com.android.tv.settings.mtc.overlay` are disabled, but neither is in the list of apps
Free My TV turns off: probably a factory setting.

## Tools and pitfalls

| Pitfall | Symptom | Fix |
| --- | --- | --- |
| Java 27 is the Mac's default | `./build.sh` crashes in d8 (`NullPointerException` on `HomeView$Quotes.class`) | build with Java 17, already installed with Homebrew |
| Preview opened as a file | the round buttons', weather's and drawer's icons stay blank (CSS masks blocked on `file://`) | serve the preview over HTTP; `webos/build.py`, `tizen/build.py` and `CLAUDE.md` now say so |
| `build.sh` empties `out/` | the preview in `out/webos` disappears after an Android build | rebuild the preview after each Android build |

```bash
# Android app (Java 17)
JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home ./build.sh

# LG/Samsung preview, then http://localhost:8000
python3 webos/build.py --preview
python3 -m http.server -d out/webos
```

## Deploying to the TV

```bash
adb connect 192.168.2.77:5555
adb -s 192.168.2.77:5555 install -r FreeMyTV.apk
```

1. If the TV asks *Allow USB debugging?*, tick *Always allow* and accept.
2. On the TV: *Free My TV* → *Home Screen* → *Free My TV*. The TV asks again to allow debugging: the app uses it to
   make the switch.
3. To check what the switch did: `adb logcat -s FreeMyTV`.
4. To bring everything back: *Free My TV* → *Home Screen* → *Original*.

Updates must be signed with the same key: keep `freemytv.keystore` (never committed) and back it up.

## Screen mirroring from iPhone and Mac

The Sharp has no AirPlay, Apple's system: no setting turns it on. Built-in Chromecast
(`com.google.android.apps.mediashell`) is on, so casting from Android works, and Free My TV doesn't touch it.

| From | How | To install |
| --- | --- | --- |
| Mac | Chrome → ⋮ → *Cast…* → *Sources* → *Cast screen* | nothing |
| iPhone, inside an app | the Cast button in YouTube, Netflix, Canal+, Prime Video… | nothing |
| iPhone or Mac, the whole screen | Control Centre → *Screen Mirroring* → the name AirScreen shows | AirScreen on the TV (Play Store, `com.ionitech.airscreen`, free with ads) |

The iPhone or the Mac must be on the same Wi-Fi, and AirScreen open on the TV (or its auto-start turned on in its
settings).

## Next

- [ ] Keep the personal settings (shelf with Canal+ and the browser) apart from the rest before offering anything to
  the public project.
- [ ] Look at the home screen on the TV in the evening, in automatic theme, to check the night sky.
- [ ] Never commit `config.properties`, `freemytv.keystore` or the TMDB key.
