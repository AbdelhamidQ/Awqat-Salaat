# Tray icon setting checks

Run the settings tests from the repository root:

```powershell
dotnet test tests/AwqatSalaat.Settings.Tests/AwqatSalaat.Settings.Tests.csproj -c Release
```

These tests use the test host's settings store, separate from the application. They cover the visible default, preview and Cancel, Save and reload, and labels and help text in the four supported languages.

## Windows runtime checklist

Use the WinUI Release build for these checks. The existing Debug build uses a fixed date and resets configuration, so it is unsuitable for verifying the running prayer countdown.

1. Open Settings > General. Confirm **Show system tray icon** starts enabled for a new settings profile.
2. Turn it off. Confirm the icon disappears from the notification area and its overflow while the taskbar widget and countdown continue working.
3. Cancel with the cross button. Confirm the icon returns. Turn it off again and Save with the checkmark.
4. Restart the app. Confirm the saved preference is applied. Preview turning it on, then Cancel. Confirm the saved hidden preference returns.
5. With the icon hidden, verify reminder and prayer-time notifications and sound playback.
6. Hide the widget. Confirm the tray icon becomes available so Show and Quit remain reachable. Show the widget again and confirm the saved icon preference returns.
7. Restart Windows Explorer. Confirm the widget recovers and the icon preference is retained. Repeat with the widget deliberately hidden and confirm the recovery icon remains available.
8. Verify display disconnect/reconnect recovery, including a show request while no display is available. The app must retain a recovery icon and show the widget when a display returns.
9. Quit with each icon preference and confirm the app exits without leaving a tray icon. Check normal Windows sign-out/shutdown separately.

### Recovery regression cases

- Start with no display available. Confirm the recovery icon stays visible, Show is enabled, and Hide and both positioning commands are disabled.
- Hide the widget, request Show while no display is available, then reconnect a display. Confirm Show resumes only when a display exists. An explicit Hide before reconnection must cancel that pending request.
- Lose the widget's display or native window while its managed object is still present. Confirm the recovery icon becomes available and the next Show disposes the stale object before creating a replacement.
- Deliver a taskbar-created or display-change callback after shutdown has started. Neither the tray icon nor widget should be recreated.
- Recreate the widget while position or animation updates are still queued. Updates for the disposed host must be ignored without a crash.
- Confirm the tray setting is next to the lock-screen setting in General. Check the Arabic label uses "درج النظام" and the help uses "الأداة".

The icon is hidden using the existing H.NotifyIcon visibility API. Its message window remains alive for Explorer restart and session-ending events. No prayer, notification, or audio services are disabled by the setting.
