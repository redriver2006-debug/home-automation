# Home Automation Project: A No-Code Journey with AI

## Background

I do not have a software engineering background and still have no hands-on coding experience. This home automation system was built and iterated **totally with AI assistance**, while I focused on goals, validation, and decision-making.

This document records how I built a home automation system from scratch, the challenges encountered, and the lessons learned along the way.

---

## Project Overview

### Goal
Create an intelligent home monitoring system that can:
- Detect when the front gate is open/closed
- Send real-time alerts via messaging apps
- Integrate with smart home platforms (HomeKit)
- Run 24/7 on low-power hardware

### Hardware Used
- **Video Doorbell**: IP-based doorbell camera with motion detection and API access
- **PTZ Camera**: Secondary camera for backup monitoring
- **Raspberry Pi**: Edge computing device running detection scripts
- **MacBook**: Central hub running the AI assistant (OpenClaw)

### Software Stack
- **OpenClaw**: AI assistant framework for automation orchestration
- **Python**: Detection scripts using OpenCV
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
- Connected doorbell camera via HTTP API
- Implemented motion event webhooks
- Set up image capture on motion triggers

### Phase 3: Gate Detection Algorithm (Feb 16-20, 2026)

This was the most challenging phase, requiring multiple iterations:

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

#### v5.3 - Dark Pixel Ratio (Final)
- **Approach**: Measure dark pixel ratio in foreground region
- **Logic**: 
  - ROI covers only the nearest thick fence bar area
  - High dark pixel ratio (≥0.92) = gate closed
  - Low dark pixel ratio (<0.92) = gate open
- **Result**: Reliable detection in various lighting conditions ✅

### Phase 4: Homebridge Integration (Early Mar 2026)
- Installed Homebridge on Raspberry Pi
- Configured doorbell and camera plugins
- Enabled HomeKit access from iPhone

### Phase 5: Remote Access (Mar 2026)
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
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Doorbell   │───▶│ Raspberry Pi │───▶│   MacBook    │  │
│  │   Camera     │    │  (Detection) │    │  (OpenClaw)  │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   │                    │          │
│         │                   │                    │          │
│  ┌──────────────┐    ┌──────────────┐           │          │
│  │  PTZ Camera  │    │  Homebridge  │           │          │
│  │   (Backup)   │    │  (HomeKit)   │           │          │
│  └──────────────┘    └──────────────┘           │          │
│                                                  │          │
└─────────────────────────────────────────────────────────────┘
                                                   │
                                                   ▼
                                          ┌──────────────┐
                                          │   Telegram   │
                                          │   Alerts     │
                                          └──────────────┘
```

---

## Detection Algorithm Details

### Final Algorithm (v5.3)

```
Input: Camera image (640x480)
   │
   ▼
┌──────────────────────────────────┐
│ 1. Extract ROI (foreground area) │
│    x: 10-70, y: 120-460          │
└──────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────┐
│ 2. Convert to grayscale          │
└──────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────┐
│ 3. Count dark pixels (value < 90)│
└──────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────┐
│ 4. Calculate dark pixel ratio    │
│    ratio = dark_pixels / total   │
└──────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────┐
│ 5. Decision                      │
│    ratio ≥ 0.92 → CLOSED         │
│    ratio < 0.92 → OPEN → ALERT!  │
└──────────────────────────────────┘
```

### Why This Works
- The thick foreground fence bars create a consistent dark region when the gate is closed
- When the gate is open, the dark bars are absent, resulting in a lighter region
- Using ratio instead of absolute values handles lighting variations

---

## Key Challenges & Solutions

### Challenge 1: False Positives from Background
**Problem**: Initial algorithms detected background objects as gate bars.

**Solution**: Narrowed the ROI to only include the nearest foreground area where the thick gate bars appear.

### Challenge 2: Lighting Variations
**Problem**: Different times of day produced different detection results.

**Solution**: Switched from edge-based detection to dark pixel ratio, which is more robust to lighting changes.

### Challenge 3: Network Stability
**Problem**: Occasional camera connection failures.

**Solution**: Implemented fallback to secondary camera, with graceful failure handling (log errors, don't spam alerts).

### Challenge 4: Remote Access Complexity
**Problem**: Need to access local devices from outside home network.

**Solution**: Used Tailscale VPN with subnet routing, though this required careful configuration to avoid connectivity issues.

---

## Lessons Learned

### Technical
1. **Start simple, iterate**: The final algorithm is much simpler than early attempts
2. **ROI matters**: Focusing on the right region eliminates most noise
3. **Ratio > Absolute**: Relative measurements handle variations better
4. **Fallbacks are essential**: Always have a backup plan for critical systems

### Process
1. **AI can build, human must validate**: AI handles implementation, but human judgment is crucial for requirements and validation
2. **Document as you go**: Recording iterations helps understand what works and why
3. **Test with real data**: Lab testing != production conditions
4. **Be cautious with network commands**: Destructive operations need explicit approval

### Operational
1. **Prefer false positives over false negatives**: For security, it's better to over-alert than miss events
2. **Local processing > Cloud dependency**: Edge computing on Pi reduces latency and improves reliability
3. **Secure by default**: Use VPN for remote access, not port forwarding

---

## Future Improvements

- [ ] Add machine learning-based detection for better accuracy
- [ ] Implement multi-frame analysis for motion tracking
- [ ] Add voice announcements via smart speakers
- [ ] Create a web dashboard for monitoring status

---

## Conclusion

This project demonstrates that complex home automation systems can be built without traditional coding skills, using AI as a collaborative partner. The key is to:

1. **Define clear goals** - Know what you want to achieve
2. **Iterate rapidly** - Fail fast, learn faster
3. **Trust but verify** - AI proposes, human validates
4. **Document everything** - Future you will thank present you

The system has been running reliably since late February 2026, providing peace of mind with automated gate monitoring and alerts.

---

## Acknowledgments

This project was built with assistance from AI coding agents, demonstrating the potential of human-AI collaboration in practical engineering tasks.

---

*Last updated: March 2026*
