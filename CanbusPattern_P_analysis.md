# CanbusPattern_P — Android Activity-to-CAN Bus Audio Event Bridge

## Overview

`CanbusPattern_P` is a system-level component observed running on a Chinese Android head unit (MediaTek AC8227L SoC). It acts as a bridge between the Android activity lifecycle and the vehicle's CAN bus, specifically for audio-related events.

It runs as its own persistent process (observed as PID 1141) and monitors which Android activity is in the foreground, then dispatches CAN bus messages in response to activity changes.

---

## Observed Log Event

```
06-08 01:30:33.452  1141  1141 D CanbusPattern_P: sendSound: activityName==com.android.settings.Settings$SystemDashboardActivity
```

| Field | Value |
|---|---|
| Timestamp | 2026-06-08 01:30:33.452 |
| PID / TID | 1141 / 1141 (single-threaded or main thread) |
| Log level | D (Debug) |
| Component | `CanbusPattern_P` |
| Method | `sendSound` |
| Trigger | Foreground activity = `com.android.settings.Settings$SystemDashboardActivity` |

---

## What It Does

When an Android activity comes to the foreground, `CanbusPattern_P` calls its `sendSound` method with the activity name as a parameter. This method is responsible for translating the current screen context into a CAN bus audio command.

Likely behaviors triggered by this mechanism:

- **Amp mute/unmute control** — suppress or restore media audio when non-media screens are active
- **UI transition sounds** — play click or chime sounds through the car's speaker system via CAN rather than Android's audio HAL
- **DSP/amplifier context signaling** — inform downstream audio hardware what the head unit is currently doing so it can adjust routing, gain, or EQ accordingly
- **Source switching** — tell the car's audio system to switch between Android media output and system UI audio

---

## Architecture Context

On MediaTek-based Chinese head units, the Android OS and the vehicle's legacy audio hardware operate on separate domains:

```
┌─────────────────────────────────┐
│         Android Userspace       │
│                                 │
│  ActivityManager  ──────────►  CanbusPattern_P (PID 1141)
│  (foreground app tracking)      │        │
│                                 │        │ sendSound(activityName)
└─────────────────────────────────┘        │
                                           ▼
                              ┌────────────────────────┐
                              │   CAN Bus TX Layer     │
                              │  (kernel driver /      │
                              │   vendor HAL)          │
                              └────────────┬───────────┘
                                           │
                                    CAN Bus (vehicle)
                                           │
                              ┌────────────▼───────────┐
                              │  Car Audio Hardware    │
                              │  (Amp, DSP, OEM HU     │
                              │   remnants if hybrid)  │
                              └────────────────────────┘
```

The `_P` suffix in `CanbusPattern_P` likely denotes a platform-specific variant — different car models or OEM configurations may have different pattern sets mapping activity names to CAN frame IDs and payloads.

---

## Why This Pattern Exists

The car's amplifier, DSP, and speaker system predate Android integration. They expect CAN messages to control audio state — things like "source active," "mute," or "UI sound event." Android's audio framework has no native CAN awareness, so a shim component like `CanbusPattern_P` is necessary to translate Android UI events into the language the car's hardware understands.

This is preferable to routing everything through Android's audio HAL because:

1. CAN-based amp control is faster and more reliable for hard mute/unmute operations
2. Some OEM audio systems won't respond to Android audio sessions at all — they only accept CAN commands
3. UI sounds may need to play even if Android's audio stack is busy or muted

---

## Notes

- The component fired specifically when the **Android System → System** settings screen (`SystemDashboardFragment`) became active, suggesting it monitors at least some stock Android activities in addition to head-unit-specific ones.
- The `zjinnova/zlink` Bluetooth daemon and `comautochipsbluetoothsdk` (AutoChips BT SDK) are separate IPC components on the same device, indicating a layered vendor middleware stack typical of these units.
- No CAN frame ID or payload data is visible in logcat at the debug level captured; the actual CAN message content would require a lower-level CAN sniffer or kernel log.
