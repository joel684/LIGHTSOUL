<div align="center">

# LIGHTSOUL

**A tiny, fast home screen for low-end Android TVs.**
App grid, rotating wallpapers, and YouTube shelves for wrestling and international movies.
No dependencies, no accounts, no tracking.



![Android 5.0+](https://img.shields.io/badge/Android-5.0%2B-3DDC84?logo=android&logoColor=white)




![Dependencies: 0](https://img.shields.io/badge/dependencies-0-F2B84B)




![Build](https://img.shields.io/badge/build-GitHub%20Actions-181717?logo=githubactions&logoColor=white)




![Size](https://img.shields.io/badge/APK-under%202%20MB-black)


[

![Latest release](https://img.shields.io/github/v/release/joel684/LIGHTSOUL?label=release&color=F2B84B)

](https://github.com/joel684/LIGHTSOUL/releases/latest)

### [⬇ Download LIGHTSOUL.apk](https://github.com/joel684/LIGHTSOUL/releases/latest/download/LIGHTSOUL.apk)

[Install](#install) · [Features](#features) · [FAQ](#faq) · [Privacy](#privacy-and-data-collection) · [Customize](#customize)

</div>

<!--
  SCREENSHOTS: add 2-3 photos or screenshots of the home screen here, e.g.
  1. Create a folder docs/ and upload home.png, shelves.png
  2. Replace this comment with:
     <p align="center"><img src="docs/home.png" width="48%"> <img src="docs/shelves.png" width="48%"></p>
-->

---

## Why LIGHTSOUL

Many cheap smart TVs ship with about 500 MB of free memory and a stock launcher that is slow to scroll. LIGHTSOUL is written to stay out of the way: one activity, plain Android views, no AndroidX, no Kotlin runtime, no Compose, no image library. Everything is sized from the device's heap, so a weak TV simply gets smaller caches instead of running out of memory.

- **Small.** The APK is under 2 MB and the whole project is readable in an evening.
- **Private.** No account, no analytics, no ads. It only talks to YouTube (for the shelves) and GitHub (for updates).
- **Gentle on slow TVs.** Icons and wallpapers load one at a time on a low-priority thread, so scrolling never stalls.
- **Self-updating.** New versions arrive on their own, and you confirm each install.

## Features

- **Fast app grid.** Every launchable app on the TV, sorted by name, with icons decoded one at a time on a low-priority thread so scrolling never stalls.
- **Rotating wallpapers.** Four 1280x720 pictures, one bitmap in memory at a time, none while the home screen is out of sight. A weak device gets 16-bit colour to halve the memory.
- **Online shelves.** Two shelves read from public YouTube channel feeds:
  - **Wrestling:** full shows and full matches (clips, promos, fan events and talks are filtered out)
  - **International movies:** full-length films from channels that publish them themselves, such as the Korean Film Archive, FilmRise and Movie Central (trailers, songs and scenes are filtered out)

  Selecting a card opens the video in a YouTube app (YouTube for TV, YouTube, or SmartTube if installed).
- **Wi-Fi / Ethernet only.** Shelves and updates are only downloaded on Wi-Fi or Ethernet. Feeds refresh at most every 10 minutes, and never while another app is in front.
- **Memory saver.** Every 5 minutes it asks Android to stop idle background apps. It never clears app data, never touches the app you are using, and skips system apps like the Play Store and the stock TV launcher.
- **Starts after boot.** Brings itself to the front once the TV has booted, for TVs that otherwise open on the last HDMI input.
- **Self-update.** Checks the latest GitHub release of [joel684/LIGHTSOUL](https://github.com/joel684/LIGHTSOUL) every 2 hours and offers the new APK through the system installer, which always asks you to confirm.

## Install

### Quickest way: the Downloader app

1. Install the free **Downloader** app on your TV (Fire TV and Android TV stores).
2. Open it and type this address:

   ```
   github.com/joel684/LIGHTSOUL/releases/latest/download/LIGHTSOUL.apk
   ```

3. Let it download, choose **Install**, and allow installs from Downloader if the TV asks.
4. Press Home and choose **LIGHTSOUL** as the home app.
5. When asked, allow **Display over other apps**. This is only needed so it can open itself after the TV boots.

### Other ways

1. Download `LIGHTSOUL.apk` from the [latest release](../../releases/latest).
2. Sideload it onto the TV (USB drive, or `adb install LIGHTSOUL.apk`).
3. Continue from step 4 above.

Updates after that arrive on their own.

> **Coming from TJHome?** LIGHTSOUL has its own app id (`app.lightsoul`) and its own signing key, so it installs next to TJHome instead of replacing it, and TJHome will not update to it. Install LIGHTSOUL, pick it as the home app, then uninstall TJHome.

### Optional: adb permissions

Some TVs have no settings screen for these. Grant them once from a computer:

```bash
# open after boot (Android 10+)
adb shell appops set app.lightsoul SYSTEM_ALERT_WINDOW allow

# memory saver also spares the app currently in front
adb shell appops set app.lightsoul GET_USAGE_STATS allow
```

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
LIGHTSOUL/
├── .github/workflows/build.yml     build, sign, publish release
└── app/src/main/
    ├── AndroidManifest.xml
    ├── java/app/lightsoul/
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

## Contributing

Bug reports, channel suggestions and pull requests are welcome: open an [issue](https://github.com/joel684/LIGHTSOUL/issues). If LIGHTSOUL helps your TV, a star on the repo makes it easier for other people to find.

## Notes

- Videos belong to their channels' owners. LIGHTSOUL only reads public feeds and opens the video in YouTube; it does not download or re-host any video.
- **Wallpaper credits:** the four wallpapers are illustrations drawn by みかぶる (Mikaburu), released into the public domain under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) in the artist's repo [Mikanixonable/illusts](https://github.com/Mikanixonable/illusts) (files `451.png`, `169.png`, `172.png`, `5.png`). CC0 does not require credit; it is given anyway. They were scaled to 1280x720, darkened on the left so the app names stay readable, and one was mirrored.
```
