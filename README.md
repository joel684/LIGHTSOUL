<div align="center">

# TJHome

**A tiny, fast home screen for low-end Android TVs.**
App grid, rotating wallpapers, and YouTube shelves for wrestling, Hindi-dubbed movies and K-drama. No dependencies, no accounts, no tracking.

![Android 5.0+](https://img.shields.io/badge/Android-5.0%2B-3DDC84?logo=android&logoColor=white)
![Dependencies: 0](https://img.shields.io/badge/dependencies-0-F2B84B)
![Build](https://img.shields.io/badge/build-GitHub%20Actions-181717?logo=githubactions&logoColor=white)
![Size](https://img.shields.io/badge/APK-under%202%20MB-black)

</div>

---

## Why TJHome

Many cheap smart TVs ship with about 500 MB of free memory and a stock launcher that is slow to scroll. TJHome is written to stay out of the way: one activity, plain Android views, no AndroidX, no Kotlin runtime, no Compose, no image library. Everything is sized from the device's heap, so a weak TV simply gets smaller caches instead of running out of memory.

## Features

- **Fast app grid.** Every launchable app on the TV, sorted by name, with icons decoded one at a time on a low-priority thread so scrolling never stalls.
- **Rotating wallpapers.** Four 1280x720 pictures, one bitmap in memory at a time, none while the home screen is out of sight. A weak device gets 16-bit colour to halve the memory.
- **Online shelves.** Three shelves read from public YouTube channel feeds:
  - Wrestling: full shows and full matches (clips, promos, fan events and talks are filtered out)
  - Hindi-dubbed movies
  - Hindi-dubbed K-drama

  Selecting a card opens the video in a YouTube app (YouTube for TV, YouTube, or SmartTube if installed).
- **Wi-Fi / Ethernet only.** Shelves and updates are only downloaded on Wi-Fi or Ethernet. Feeds refresh at most every 10 minutes, and never while another app is in front.
- **Memory saver.** Every 5 minutes it asks Android to stop idle background apps. It never clears app data, never touches the app you are using, and skips system apps like the Play Store and the stock TV launcher.
- **Starts after boot.** Brings itself to the front once the TV has booted, for TVs that otherwise open on the last HDMI input.
- **Self-update.** Checks this repo's latest GitHub release every 2 hours and offers the new APK through the system installer, which always asks you to confirm.

## Install

1. Download `TJHome.apk` from the [latest release](../../releases/latest).
2. Sideload it onto the TV (USB drive, a downloader app, or `adb install TJHome.apk`).
3. Press Home and choose **TJHome** as the home app.
4. When asked, allow **Display over other apps**. This is only needed so it can open itself after the TV boots.

Updates after that arrive on their own.

### Optional: adb permissions

Some TVs have no settings screen for these. Grant them once from a computer:

```bash
# open after boot (Android 10+)
adb shell appops set app.tjhome SYSTEM_ALERT_WINDOW allow

# memory saver also spares the app currently in front
adb shell appops set app.tjhome GET_USAGE_STATS allow
```

## Customize

| What | Where |
|------|-------|
| Channels and the filters that decide what counts as a full show or movie | `app/src/main/java/app/tjhome/Sources.java` |
| Wallpapers (1280x720, subject on the right, dark on the left under the apps) | `app/src/main/res/drawable-nodpi/` (`wp_*.webp`) and the `WALLPAPERS` list in `MainActivity.java` |
| Apps the memory saver never stops | `KEEP` list in `MemoryKeeper.java` |
| Which repo it updates from | `REPO` in `Updater.java` |

To add a channel, open it on YouTube, go to **About > Share channel > Copy channel ID** (starts with `UC`), and add a line to `Sources.ALL`. A playlist ID (starts with `PL`) works the same way.

## Build

### With GitHub Actions (recommended)

Every push builds a signed APK and uploads it as a workflow artifact. Pushes to the default branch also publish a release (`v<build number>`) and keep the five newest. The build number is minutes since 1970, so every build counts as an update.

Android only accepts an update signed with the same key as the installed app, so create one permanent key and keep it safe:

```bash
keytool -genkeypair -v -keystore release.jks -alias tjhome \
  -keyalg RSA -keysize 2048 -validity 10000
base64 -w0 release.jks
```

Then add four repository secrets under **Settings > Secrets and variables > Actions**:

| Secret | Value |
|--------|-------|
| `KEYSTORE_BASE64` | the output of the `base64` command above |
| `KEYSTORE_PASSWORD` | the keystore password |
| `KEY_ALIAS` | `tjhome` |
| `KEY_PASSWORD` | the key password (the same as the keystore password for a default PKCS12 keystore) |

Never commit `release.jks` or a file containing these values. If the key is lost, the launcher has to be uninstalled and installed again once.

### Locally

There is no Gradle wrapper. Use Gradle 8.9 with JDK 17 and the Android SDK (compileSdk 34), or open the folder in Android Studio:

```bash
gradle assembleRelease -PverCode=7 -PverName=1.7
```

Without the `KEYSTORE_FILE` environment variable the build is signed with the debug key, which is fine for a test install but cannot update a release build.

## Privacy and data collection

**Short answer: TJHome does not collect your data.** There is no account, no analytics, no ads, no crash reporting and no third-party code, and the project has no server of its own that anything could be sent to.

### What leaves the TV

TJHome only ever talks to these hosts, over https, and only on Wi-Fi or Ethernet:

| Host | Why |
|------|-----|
| `www.youtube.com`, `i.ytimg.com` | public channel feeds and thumbnails for the shelves |
| `api.github.com`, `github.com`, `*.githubusercontent.com` | update check, and the APK download (GitHub serves it from a `githubusercontent.com` host) |

Like any web request, these servers can see the TV's IP address and one generic header, `User-Agent: Mozilla/5.0 (Linux; Android TV) TJHome`. TJHome adds nothing else: no device id, no account, no cookies, no list of your apps. Any other host is refused, including when a request is redirected, and plain http is switched off.

Videos open in the YouTube app you already have. That app is Google's and follows Google's privacy policy; TJHome only hands it the video link.

### What stays on the TV

- The list of installed apps, read locally to draw the app grid. It is never sent anywhere.
- A cache of the shelf feeds and thumbnails (the disk cache is kept under 3 MB), and a downloaded update until it is installed.
- One setting: whether the "Display over other apps" prompt has already been shown.
- Only if you grant the optional usage access: the memory saver reads which app is in front, on the TV, so it does not stop it.

### Check it yourself

All of the app's network code is in `Feed.java` and `Updater.java`, and the host allow-list is the `allowedHost` function in `Feed.java`. The project is small enough to read in an evening, and a network monitor on your router will show the same short list of hosts.

## Permissions

| Permission | Used for |
|------------|----------|
| `INTERNET`, `ACCESS_NETWORK_STATE` | shelves and updates, and checking that the TV is on Wi-Fi / Ethernet |
| `REQUEST_INSTALL_PACKAGES` | handing the downloaded update to the system installer |
| `RECEIVE_BOOT_COMPLETED`, `SYSTEM_ALERT_WINDOW` | coming to the front after boot |
| `KILL_BACKGROUND_PROCESSES` | the memory saver |
| `PACKAGE_USAGE_STATS` (optional, via adb) | letting the memory saver see which app is in front |

## Project layout

```
TJHome/
├── .github/workflows/build.yml     build, sign, publish release
└── app/src/main/
    ├── AndroidManifest.xml
    ├── java/app/tjhome/
    │   ├── MainActivity.java       home screen: app grid, wallpapers, shelves
    │   ├── Sources.java            channels and title filters
    │   ├── Feed.java               feed and thumbnail download, parsing, cache (allow-listed hosts only)
    │   ├── Shelf.java              one row of video cards
    │   ├── ShelfPanel.java         the column of shelves above the app grid
    │   ├── Updater.java            GitHub release check and download
    │   ├── InstallRx.java          opens the system's install confirmation
    │   ├── MemoryKeeper.java       idle-app memory saver
    │   ├── CacheCleaner.java       keeps the disk cache under 3 MB
    │   ├── BootRx.java             open after boot
    │   └── BootAgainRx.java        the delayed second attempt
    └── res/                        wallpapers, banner, focus ring, theme
```

## Notes

- Videos belong to their channels' owners. TJHome only reads public feeds and opens the video in YouTube; it does not download or re-host any video.
- **Wallpaper credits:** the four wallpapers are illustrations drawn by みかぶる (Mikaburu), released into the public domain under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) in the artist's repo [Mikanixonable/illusts](https://github.com/Mikanixonable/illusts) (files `451.png`, `169.png`, `172.png`, `5.png`). CC0 does not require credit; it is given anyway. They were scaled to 1280x720, darkened on the left so the app names stay readable, and one was mirrored.
