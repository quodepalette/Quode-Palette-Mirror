# Quode Palette Mirror

Mirror and control an Android phone from Windows over USB, in a clean dark window with your own branding.

Quode Palette Mirror is a small Python and Tk app that wraps [scrcpy](https://github.com/Genymobile/scrcpy). scrcpy does the heavy lifting (video, input, ADB). This project adds a friendlier front end, phone display controls, and a proper Windows identity for the mirror window.

<p align="center">
  <img src="docs/Screenshot1.png" alt="Quode Palette Screenshot" width="100%">
  <b>Screenshot I</b>
</p>

<table>
    <td width="50%" align="center">
      <a href="docs/Screenshot2.png"><img src="docs/Screenshot2.png" alt="Screenshot" width="100%"></a>
      <br>
      <b>Screenshot II</b>
      <br>
      <!-- <sub>Convert common audio formats with FFmpeg</sub> -->
    </td>
    <td width="50%" align="center">
      <a href=docs/Screenshot3.png"><img src="docs/Screenshot3.png" alt="Screenshot" width="100%"></a>
      <br>
      <b>Screenshot III</b>
      <br>
      <!-- <sub>Convert common video formats with FFmpeg</sub> -->
    </td>
</table>

## Features

- One click mirroring for any USB debugging phone
- Dims the phone while mirroring and restores brightness afterwards
- Mirror window carries the Quode Palette name and icon in the taskbar
- Controller lives in the system tray when you close it
- Ships as a single EXE with scrcpy and ADB inside, nothing else to install

## Quick start

1. Download `QuodePaletteMirror.exe` from the [Releases](../../releases) page.
2. On the phone, turn on Developer options and USB debugging.
3. Plug the phone in with a data cable and unlock it.
4. Run the EXE and accept the USB debugging prompt on the phone.
5. Pick your phone and press **Connect and mirror**.

Windows SmartScreen may warn about the EXE because it is not code signed. Only run builds you trust.

## Build from source

You need Windows 10 or 11, Python 3.11 or newer, and an internet connection.

```bat
git clone <your fork or this repo>
cd QPM
build.bat
```

The result is `dist\QuodePaletteMirror.exe`. `build.bat` closes any running copy, downloads the official scrcpy Windows release into `vendor\scrcpy`, gives `scrcpy.exe` this app's name and icon with [rcedit](https://github.com/electron/rcedit), installs the build tools and runs PyInstaller. The app itself never downloads anything at runtime.

To run straight from source after the first build, put the scrcpy files in `vendor\scrcpy`, then:

```bat
py -3 -m pip install pillow pystray
py -3 main.py
```

## How it works

The controller starts `scrcpy.exe` as a child process and lets it open its own native window. The app does not embed or resize that window, so scrcpy keeps handling rotation and scaling by itself. What the app adds on top:

- **Window title and icon.** scrcpy 5.x has no icon flag, so the app points `SCRCPY_ICON_DIR` at a temp folder containing the logo as `scrcpy.png`.
- **Taskbar identity.** Windows labels the taskbar right-click menu after the program that made the window. The build rewrites the icon and file description inside `scrcpy.exe`, and at runtime `winshell.py` also gives the window its own AppUserModelID, display name and relaunch command, so it reads as Quode Palette Mirror instead of scrcpy.
- **Sharp icons.** `winshell.py` loads the small and large icon at the exact size the current DPI asks for, straight from the multi-size `QPM-LOGO.ico`.
- **Brightness.** Before mirroring, the app saves the phone brightness and adaptive mode, sets brightness to zero, and restores both when the session ends. Some vendors block these settings.

## Project layout

| File | Purpose |
| --- | --- |
| `main.py` | App window, device list, scrcpy process and session lifecycle |
| `ui_kit.py` | Theme, fonts and the custom Tk widgets |
| `winshell.py` | Windows helpers for icons and taskbar identity |
| `build.bat` | Local build: fetches scrcpy and runs PyInstaller |
| `.github/workflows/release.yml` | Builds the EXE and attaches it to a GitHub release on `v*` tags |
| `version_info.txt` | Version details embedded in the EXE |

## Releasing

Update the version in `version_info.txt` (`filevers`, `prodvers` and the two version strings), then push a tag:

```bash
git tag v1.0.1
git push origin v1.0.1
```

The workflow builds on a Windows runner and publishes `QuodePaletteMirror.exe` with generated release notes. It can also be started by hand from the Actions tab.

## Troubleshooting

- **No device found:** Try another cable or port. Some cables only charge.
- **Authorization required:** Unlock the phone, accept the USB debugging prompt, then press **Refresh**.
- **Device is offline:** Unplug and reconnect the cable, then refresh.
- **Mirror does not start:** Check USB debugging and authorization, then read the error message.
- **Taskbar menu still says scrcpy:** Rebuild with `build.bat` and watch step 2 for a warning. Windows also caches taskbar details, so close the mirror and start it again, or sign out and back in.
- **SmartScreen warning:** Expected for unsigned builds. Signing needs a code signing certificate.

## Fonts

The font is under the SIL Open Font License.

## License

Released under the [PolyForm Noncommercial License 1.0.0](LICENSE). You are free to use, modify and share it for personal, educational and other noncommercial purposes. For commercial use, please get in touch.
[Email Us](mailto:hello@quodepalette.com)