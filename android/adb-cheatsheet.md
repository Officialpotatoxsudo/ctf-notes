# ADB Cheatsheet

Android Debug Bridge commands for labs and devices you own.

## 1. Connect

```bash
adb devices
adb start-server
adb kill-server
adb connect 192.168.1.50:5555
adb disconnect
```

If the device shows `unauthorized`, accept the RSA prompt on the device. If it shows
`offline`, reconnect the cable or restart the server.

## 2. Shell and packages

```bash
adb shell
adb shell whoami
adb shell getprop ro.build.version.release
adb shell getprop ro.product.model
adb shell pm list packages
adb shell pm list packages -3        # third-party only
adb shell pm list packages -s        # system only
adb shell pm path com.example.app
adb shell dumpsys package com.example.app | head -80
```

Install, uninstall, and clear data:

```bash
adb install app.apk
adb install -r app.apk               # replace
adb install -d app.apk               # allow downgrade
adb uninstall com.example.app
adb shell pm clear com.example.app
```

## 3. Activities and intents

```bash
adb shell am start -n com.example.app/.MainActivity
adb shell am start -a android.intent.action.VIEW -d "https://example.com"
adb shell am start -a android.intent.action.SEND -t text/plain --es android.intent.extra.TEXT "hello"
adb shell am force-stop com.example.app
adb shell am broadcast -a com.example.MY_ACTION
```

Inspect the current activity:

```bash
adb shell dumpsys activity activities | grep -E "mResumedActivity|mFocusedActivity"
```

## 4. Logs

```bash
adb logcat
adb logcat -c                        # clear
adb logcat -v time
adb logcat *:E                       # errors only
adb logcat -s MyTag
adb logcat | grep -i "com.example"
```

Save a log:

```bash
adb logcat -d -v threadtime > logcat.txt
```

## 5. Files

```bash
adb pull /sdcard/Download/file.txt .
adb push local.txt /sdcard/Download/
adb shell ls -la /sdcard/Download
adb shell cat /sdcard/Download/file.txt
```

App-private data is usually under `/data/data/<package>` and requires root or a debuggable
build.

## 6. Screen and input

```bash
adb exec-out screencap -p > screen.png
adb shell screenrecord /sdcard/demo.mp4
adb shell input tap 500 1000
adb shell input swipe 500 1500 500 500 300
adb shell input text "hello%sworld"
adb shell input keyevent 26          # power
adb shell input keyevent 3           # home
adb shell input keyevent 4           # back
```

`input text` uses `%s` for spaces.

## 7. Settings and device state

```bash
adb shell settings list global
adb shell settings get global airplane_mode_on
adb shell settings put global airplane_mode_on 1
adb shell dumpsys battery
adb shell dumpsys wifi | head -50
adb shell dumpsys meminfo com.example.app
adb shell dumpsys gfxinfo com.example.app
```

## 8. Port forwarding

```bash
adb forward tcp:8080 tcp:8080
adb forward --list
adb forward --remove tcp:8080
adb reverse tcp:8080 tcp:8080
```

`forward` sends host traffic to the device. `reverse` sends device traffic to the host.

## 9. Root and remount

```bash
adb root
adb remount
adb shell id
adb shell su -c id
```

These only work on userdebug/eng builds or a rooted device. Do not assume root on production
hardware.

## 10. Bug reports and backups

```bash
adb bugreport bugreport.zip
adb backup -apk -shared -all -f backup.ab
adb restore backup.ab
```

Modern Android versions restrict `adb backup`. Prefer a lab build with debuggable apps.

## 11. Useful one-liners

```bash
# Current foreground package
adb shell dumpsys window | grep -E "mCurrentFocus|mFocusedApp"

# All activities of a package
adb shell dumpsys package com.example.app | grep -A2 "Activity Resolver Table"

# Exported components (quick grep)
adb shell dumpsys package com.example.app | grep -E "exported=true|permission"

# Device properties
adb shell getprop | grep -E "ro.build|ro.product|ro.debuggable"

# Screenshot without shell line-ending issues
adb exec-out screencap -p > screen.png
```

## 12. Troubleshooting

- `no permissions` on Linux: add a udev rule or run as a user in the `plugdev` group.
- `more than one device`: use `-s <serial>`.
- `closed` connection: `adb kill-server && adb start-server`.
- Wireless debugging: pair with `adb pair <ip>:<port>` first, then `adb connect`.
