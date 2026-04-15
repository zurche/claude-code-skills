# Emulator Smoke Validation

Load this only if the user asks for device or emulator validation.

## Headless AVD Mode

- Prefer headless Android emulator boot for scripted smoke tests:

```bash
"${ANDROID_SDK_ROOT:-$HOME/Library/Android/sdk}/emulator/emulator" -avd <avd-name> -no-snapshot -no-window
```

- `-no-window` keeps Android running without opening the emulator UI.
- Leave the AVD booted while you install builds, launch activities, and send input events with `adb`.
- Do not restart the emulator between each smoke check unless the user explicitly wants repeated cold boots.

## Baseline Flow

1. Confirm the target AVD name and device serial, or discover them locally.
2. Boot the AVD if needed, then wait for `adb -s <serial> wait-for-device`.
3. Install the build artifact required for the smoke pass.
4. Launch the app or target activity with `adb shell am start ...`.
5. Drive the scenario with `adb shell input keyevent`, taps, deep links, or other repo-specific commands.
6. Confirm the result with `adb shell dumpsys activity activities`, `uiautomator dump`, screenshots, or logs.
7. Shut down the emulator only when validation is complete or the user asks.

## TV-Specific Notes

- For Android TV regression checks, scripted D-pad input is usually enough for cold-open navigation bugs.
- Verify that the expected activity remains resumed after the input sequence instead of assuming the UI stayed open.
