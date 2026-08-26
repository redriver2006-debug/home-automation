# Home Automation Project: A No-Code Journey with AI

## Background

I do not have a software engineering background and still have no hands-on coding experience. This home automation system was built and iterated **totally with AI assistance**. Without AI help, I could not have completed this project on my own.

I focused on goals, validation, and decision-making, while the AI handled all technical implementation—writing scripts, debugging code, configuring systems, and integrating multiple hardware platforms.

---

## Project Overview

### Goal
Create an intelligent home monitoring system that can:
- Detect when the front gate is open/closed
- Send real-time alerts via messaging apps
- Integrate with smart home platforms (HomeKit)
- Run 24/7 on low-power hardware

### Hardware Used
- **DoorBird**: Video doorbell with motion detection and HTTP API
- **Reolink**: PTZ camera for backup monitoring
- **Zigbee Door Sensor**: Magnetic contact sensor for reliable gate detection
- **Zigbee Coordinator**: USB dongle for Zigbee network
- **Raspberry Pi**: Edge computing hub integrating all systems
- **MacBook**: Central hub running the AI assistant (OpenClaw)

### Software Stack
- **OpenClaw**: AI assistant framework for automation orchestration
- **Python**: Detection scripts and system integration
- **Zigbee2MQTT**: Zigbee device management
- **Homebridge**: HomeKit integration layer
- **Tailscale**: Secure remote access VPN
- **Telegram**: Alert delivery channel

---

## Timeline & Milestones

### Phase 1: Basic Setup (Early Feb 2026)
- Set up AI assistant (OpenClaw) on MacBook
- Configured Telegram bot for receiving alerts
- Established secure communication channels

### Phase 2: Camera Integration (Mid Feb 2026)
- Connected DoorBird via HTTP API
- Connected Reolink via HTTPS API with token authentication
- Implemented motion event webhooks
- Set up image capture on motion triggers

### Phase 3: Vision-Based Gate Detection (Feb 16-20, 2026)

This was the most challenging phase, with multiple algorithm iterations:

#### v1.0 - Simple Edge Detection
- **Approach**: Count vertical lines using Hough Transform
- **Problem**: Too many false positives from background objects

#### v2.0 - ROI-based Detection
- **Approach**: Limited detection to a specific region of interest
- **Problem**: Background fence bars still counted as gate bars

#### v3.0 - Refined Hough Parameters
- **Approach**: Tuned Canny edge detection and line detection thresholds
- **Problem**: Still confused by thin background bars vs thick foreground bars

#### v4.0 - Foreground Focus
- **Approach**: Narrowed ROI to only capture the nearest (thickest) fence bars
- **Problem**: Lighting variations caused inconsistent results

#### v5.3 - Dark Pixel Ratio
- **Approach**: Measure dark pixel ratio in foreground region
- **Result**: Better, but still unreliable under certain conditions

### Phase 4: The Pivot - Zigbee Door Sensor (Late Feb 2026)

**After extensive iteration, the AI recommended abandoning vision-based detection entirely.**

The reasoning was clear:
- Vision algorithms are inherently sensitive to lighting, weather, and camera angle
- A magnetic contact sensor provides binary, reliable open/closed detection
- Hardware solutions often beat software complexity

**AI recommended purchasing a Zigbee door sensor**, which I did. This turned out to be the right call.

### Phase 5: Multi-System Integration on Raspberry Pi (Mar 2026)

This is where the AI's value became most apparent. The AI helped me integrate **three completely different systems** on a single Raspberry Pi:

#### System 1: DoorBird Integration
- HTTP API for live snapshots
- Webhook receiver for doorbell and motion events
- Image archiving with automatic cleanup

#### System 2: Reolink Integration
- HTTPS API with session-based authentication
- Token management for API calls
- Backup camera failover logic

#### System 3: Zigbee Integration
- Zigbee2MQTT setup and configuration
- Door sensor pairing and monitoring
- MQTT message handling for state changes

**The integration script** monitors all three systems simultaneously:
- Zigbee sensor triggers → instant open/close detection
- DoorBird motion → captures snapshot for context
- Reolink backup → failover when DoorBird is unavailable
- Telegram alerts → unified notification delivery

**I could not have done this without AI assistance.** Each system has its own protocol, authentication method, and quirks. The AI wrote all the integration code, handled error cases, and debugged issues as they arose.

### Phase 6: Homebridge Integration (Early Mar 2026)
- Installed Homebridge on Raspberry Pi
- Configured DoorBird and Reolink plugins
- Enabled HomeKit access from iPhone
- Can now view cameras and receive alerts via Apple Home app

### Phase 7: Remote Access (Mar 2026)
- Set up Tailscale VPN for secure remote access
- Configured subnet routing for local device access
- **Incident**: Accidental `--reset` command caused temporary connectivity loss
- **Lesson**: Always get approval before running destructive network commands

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Internet                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Tailscale VPN
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Home Network                              │
│                                                              │
│  ┌──────────────┐                                           │
│  │   DoorBird   │───┐                                       │
│  │  (Doorbell)  │   │                                       │
│  └──────────────┘   │                                       │
│                     │      ┌──────────────┐                 │
│  ┌──────────────┐   ├─────▶│ Raspberry Pi │────────────┐    │
│  │   Reolink    │───┤      │   (Hub)      │            │    │
│  │  (PTZ Cam)   │   │      │              │            │    │
│  └──────────────┘   │      │ - Integration│            │    │
│                     │      │ - Zigbee2MQTT│            │    │
│  ┌──────────────┐   │      │ - Homebridge │            │    │
│  │Zigbee Sensor │───┘      └──────────────┘            │    │
│  │ (Gate)       │                                      │    │
│  └──────────────┘                                      │    │
│                                                        │    │
│                              ┌──────────────┐          │    │
│                              │   MacBook    │◀─────────┘    │
│                              │  (OpenClaw)  │               │
│                              └──────────────┘               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
             ┌──────────┐       ┌──────────┐       ┌──────────┐
             │ Telegram │       │ HomeKit  │       │  iPhone  │
             │  Alerts  │       │  (Home)  │       │   App    │
             └──────────┘       └──────────┘       └──────────┘
```

---

## Why Vision Detection Failed (And Why That's OK)

After spending days iterating on computer vision algorithms, we learned an important lesson:

**Sometimes the best solution is not more code, but better hardware.**

Vision-based detection challenges:
- **Lighting**: Sunrise, sunset, cloudy days, night—all produced different results
- **Weather**: Rain, fog, and shadows created false positives/negatives
- **Complexity**: Each "fix" introduced new edge cases
- **Maintenance**: Algorithm needed constant tuning

Zigbee door sensor advantages:
- **Binary output**: Open or closed, no ambiguity
- **Weather-proof**: Works in any lighting condition
- **Instant**: No image processing delay
- **Reliable**: Months of operation without false alerts
- **Low power**: Battery lasts 1-2 years

**The AI's recommendation to switch to hardware was the turning point.** It took humility to abandon days of vision algorithm work, but the result is a system that actually works.

---

## Key Challenges & Solutions

### Challenge 1: Integrating Three Different Systems
**Problem**: DoorBird (HTTP), Reolink (HTTPS+tokens), and Zigbee (MQTT) all use different protocols.

**Solution**: AI wrote a unified Python script that handles all three protocols, with proper error handling and failover logic.

### Challenge 2: Vision Detection Unreliability
**Problem**: Computer vision algorithms couldn't reliably detect gate state.

**Solution**: AI recommended switching to Zigbee door sensor—hardware beats software complexity.

### Challenge 3: Authentication Complexity
**Problem**: Reolink requires session tokens that expire.

**Solution**: AI implemented automatic token refresh and retry logic.

### Challenge 4: Remote Access
**Problem**: Need to access local devices from outside home network.

**Solution**: Tailscale VPN with careful configuration (learned the hard way about destructive commands).

---

## Lessons Learned

### Technical
1. **Hardware often beats software**: A $15 sensor solved what days of coding couldn't
2. **Integration is hard**: Different protocols, auth methods, and quirks require expertise
3. **Failover matters**: Having Reolink as backup saved us when DoorBird had issues
4. **Keep it simple**: The final solution is simpler and more reliable than early attempts

### Process
1. **AI can build, human must decide**: AI proposed the Zigbee pivot, I made the call
2. **Iterate fast, pivot faster**: Don't fall in love with your solution
3. **Test in production conditions**: Lab testing ≠ real-world conditions
4. **Document everything**: This writeup exists because we documented as we went

### On AI-Assisted Development
1. **AI handles complexity**: I couldn't have written the integration code myself
2. **AI suggests alternatives**: The Zigbee recommendation came from AI analysis
3. **Human provides direction**: I defined goals, AI found paths to achieve them
4. **Collaboration > Automation**: It's not AI replacing me, it's AI enabling me

---

## Current System Status

The system has been running reliably since early March 2026:

- ✅ **Zigbee gate sensor**: Instant, reliable open/close detection
- ✅ **DoorBird**: Motion alerts with snapshot capture
- ✅ **Reolink**: Backup camera, always recording
- ✅ **Telegram alerts**: Real-time notifications on phone
- ✅ **HomeKit**: View cameras from Apple Home app
- ✅ **Remote access**: Secure access via Tailscale from anywhere

---

## Conclusion

This project proves that **complex home automation systems can be built by non-programmers with AI assistance**.

Key takeaways:

1. **You don't need to code** - AI can write and debug all the code
2. **You need to think** - Define goals, validate results, make decisions
3. **Hardware can beat software** - Sometimes buying a sensor beats writing an algorithm
4. **Integration is valuable** - Connecting different systems is where AI shines
5. **Iterate and pivot** - The final solution looked nothing like the initial plan

Without AI assistance, this project would have been impossible for me. With AI, I built a professional-grade home automation system that runs 24/7 and gives me peace of mind.

---

## Part Two: From Scripts to Home Assistant (March - August 2026)

The system described above worked, but it was a collection of bespoke Python
scripts. Every new device meant more custom code, and only the AI really
understood how the pieces fitted together. Over the following months it was
rebuilt on **Home Assistant Core**, running in Docker on the same Raspberry Pi.

### What replaced what

| Before | After |
|---|---|
| Custom Python integration script | Home Assistant automations |
| Homebridge for HomeKit | Home Assistant's built-in HomeKit bridge |
| Sonoff Zigbee USB dongle | SMLIGHT SLZB-06U, an Ethernet-attached coordinator |
| Telegram alerts only | Telegram, iMessage and push notifications |

The Zigbee migration was the riskiest step. Moving between coordinators built on
different chip families cannot preserve existing pairings, so roughly twenty
devices had to be re-paired by hand. Because the friendly names live in the
Zigbee2MQTT configuration and are keyed by each device's hardware address, the
entity IDs survived and nothing downstream needed editing.

### What got added

- **BLE presence** - two ESPresense nodes, at the gate and in the living room,
  track phone proximity and open the sliding gate automatically on arrival
- **Infrared control of two air conditioners** - ESPHome nodes with an IR
  transmitter and receiver, so the units can be driven from Home Assistant and
  still tracked when the physical remote is used
- **Air quality automation** - CO2 sensors in three rooms, with alerts and a fan
  that responds to rising levels
- **Irrigation** - a Zigbee dual water valve with per-valve start time, run
  duration, and odd/even-date scheduling
- **Matter** - a light shared from Apple Home into Home Assistant, plus a
  scroll-wheel controller

The full configuration is published in [`config/`](../config), with credentials,
names, coordinates and addresses removed.

### Three more lessons

**Silent failures are the expensive ones.** After an iOS upgrade the companion
app re-registered itself and Home Assistant created a second device entry for
the same phone. Nothing errored. The app kept reporting faithfully - into an
entity that nothing was watching. Presence detection, automatic gate opening and
three notification automations were dead for two months before anyone noticed.
The giveaway was a battery level frozen at 45% for twelve hours while the phone
was plainly in use. A device tracker that stops changing looks exactly like a
person who never leaves.

**A backup you have never restored is not a backup.** The backup script staged
files into `/tmp`, which on this Pi is a RAM disk rather than storage. As the
system grew the copy stopped fitting; the job failed and left 1.7 GB sitting in
memory. It surfaced only because an unrelated build failed with "no space left
on device" while 18 GB of disk sat free. The same script had also been copying a
live database file, which would probably not have restored cleanly.

**Defaults are not free.** Home Assistant records every state change of every
entity for ten days unless told otherwise. That database reached 1.7 GB, and
roughly three quarters of it was Bluetooth presence readings updating several
times a second - data nobody would ever look at. Excluding the noisy entities
brought it back to a few tens of megabytes, and made backups finish in seconds
instead of hours.

### Current status (August 2026)

- Home Assistant Core on a Raspberry Pi, with around 50 devices and 470 entities
- Zigbee via SLZB-06U and Zigbee2MQTT
- Presence, gate automation, climate, air quality, irrigation and alerting
- Config-only backups, twice weekly, mirrored to a Mac and verified

---

## Acknowledgments

This project was built entirely with AI assistance, primarily using **Claude Opus 4.5** (Anthropic) through the OpenClaw framework. This demonstrates that the future of personal technology is human-AI collaboration.

---

*Last updated: August 2026*
