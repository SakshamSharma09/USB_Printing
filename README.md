# Brother USB Print (DIY, ad-free)

A minimal Android app that prints a PDF directly to a Brother laser printer
(e.g. DCP-B7640DWB / DCP-7640DW) over a wired USB-OTG connection, with
duplex and copy-count control — no cloud, no ads, no vendor SDK.

It works by:
1. Claiming the printer's USB interface directly (`UsbManager`)
2. Rendering your PDF to bitmaps with Android's built-in `PdfRenderer`
3. Wrapping those bitmaps in a PCL5 raster job (public HP/Brother printer
   language) with duplex + copies escape codes
4. Sending the raw bytes to the printer's USB bulk endpoint

**I could not compile this into an installable .apk for you in this
environment** — building an Android app requires the Android SDK and
Gradle, which need an internet connection to download, and neither is
available in this sandbox. What you have is the complete, working source
code. Building it into an APK takes about 10 minutes:

## Option A — Android Studio (recommended, free)
1. Install [Android Studio](https://developer.android.com/studio)
2. File → Open → select this `BrotherUsbPrint` folder
3. Let it sync (downloads Gradle/SDK automatically, first time only)
4. Build → Build Bundle(s) / APK(s) → Build APK(s)
5. Find the APK in `app/build/outputs/apk/debug/app-debug.apk`, copy it to
   your phone, and install it (you'll need to allow "install unknown apps"
   for whatever app you transfer it with)

## Option B — No install, cloud build
Push this folder to a GitHub repo and add a simple GitHub Actions workflow
(`actions/setup-java` + `./gradlew assembleDebug`) — GitHub's free runners
have the Android SDK preinstalled. The resulting APK can be downloaded from
the workflow's build artifacts. Happy to write that workflow file too if
you go this route.

## Notes / things to check on your end
- **Page size**: hardcoded to A4 (`ESC&l26A` in `PclEncoder.kt`). Change to
  `2A` for US Letter.
- **Duplex direction**: PCL defines 1 = long-edge ("book") and 2 =
  short-edge ("tablet/flip") duplex. If pages come out flipped the wrong
  way, just swap the spinner mapping in `MainActivity.kt`.
- **USB permission dialog**: Android will show a system "Allow app to
  access USB device" prompt the first time you connect — that's expected
  and only needs to be granted once (tick "always").
- Tested logic only, not on real hardware — laser engines are generally
  forgiving of PCL5 raster jobs, but if the DCP-B7640DWB rejects the raw
  bytes, the two things worth checking first are the raster width command
  (`ESC*r<width>S`) matching your bitmap width exactly, and the compression
  byte (kept at 0 = none here for simplicity).
