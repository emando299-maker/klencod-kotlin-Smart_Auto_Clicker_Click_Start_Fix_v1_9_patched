# Smart Auto Clicker — patched source

## What changed
- Configures and logs AccessibilityServiceInfo at connection and checks the gesture capability before dispatch.
- Uses each marker View's actual `getLocationOnScreen()` position rather than assuming WindowManager `params.x/y` are screen coordinates.
- Logs dispatch request coordinates, return/acceptance status, completion/cancellation callbacks, service capability and running/active-marker state.
- Uses an explicit tiny non-zero path for the 60 ms tap stroke.
- Keeps one gesture in flight at a time; stop does not falsely clear the in-flight flag before Android's callback arrives.
- Start remains active if there are no active points and checks again periodically. Activating a point later lets it run; double-tapping the last active point off stops the run.
- Interval setting is clamped to 1–86,400,000 ms. Android's input dispatcher and callback timing mean real throughput can be slower than the configured interval.

## Manifest / accessibility configuration
The service remains declared with `BIND_ACCESSIBILITY_SERVICE`; `auto_click_config.xml` includes `android:canPerformGestures="true"`. The service also logs the runtime gesture capability.

## Build status
The debug build was attempted with `./gradlew assembleDebug`, but this environment could not download Gradle 8.14 because network DNS access to `services.gradle.org` is unavailable. No APK was built here. This patch has not been tested on a physical Android device.

## Practical device test
1. Install a debug APK built with Android Studio/Gradle, grant “Display over other apps”, then enable the Accessibility service in Android Settings.
2. Open the calculator and add one marker. Place its center over a digit and double-tap the marker to activate it.
3. Start clicking and verify that the digit responds. Repeat with an inactive marker, several markers, hidden markers, and landscape orientation.
4. Capture logs with `adb logcat -s AutoClickService:D`. Confirm `dispatchGesture accepted=true` followed by `Gesture onCompleted`. If it reports `onCancelled`, inspect whether another service or gesture is interrupting it. If accepted is false, verify Accessibility is enabled and the service capability line.
5. Test 1 ms interval only as a lower bound; do not expect 1000 clicks/second on Android.
