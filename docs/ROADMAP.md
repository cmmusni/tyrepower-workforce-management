# Mobile App Development Plan

## Overview

This document outlines a **potential** future mobile app using React Native — only if needed after the PWA is fully deployed and in use.

**The PWA is the primary mobile experience.** It's installable, supports push notifications, works offline, and runs on all devices. A native app would only be built if the PWA proves insufficient for real-world usage.

**Status:** Not planned — evaluate after web platform is mature
**Prerequisite:** Web platform (PWA) stable in production for 3+ months, with clear evidence PWA limitations are blocking users
**Estimated Duration:** 10-14 weeks (if triggered)
**Framework:** React Native (Expo)

---

## PWA First — Native App Only If Needed

The web platform **is** the mobile app via PWA:

| Capability | PWA (what we're building) | Native App (only if needed) |
|-----------|--------------------------|----------------------------|
| Install on phone | ✅ Add to Home Screen | ✅ App Store install |
| Push notifications | ✅ Web Push (FCM) | ✅ Native push (APNs/FCM) |
| Offline clock-in | ✅ Service Worker + IndexedDB | ✅ SQLite |
| GPS clock-in | ✅ Browser Geolocation API | ✅ Background GPS |
| QR scanning | ✅ getUserMedia camera | ✅ Native camera |
| Works on iOS & Android | ✅ | ✅ |
| Biometric login | ❌ | ✅ Face ID / fingerprint |
| Background location | ❌ (foreground only) | ✅ Always-on tracking |
| App Store presence | ❌ | ✅ |

### When would a native app be justified?

Only if real users report these problems after months of using the PWA:
1. GPS is unreliable (frequent "can't get location" errors)
2. Push notifications aren't being delivered consistently
3. QR scanning is too slow or fails on certain devices
4. Store managers specifically request App Store presence for trust/discoverability
5. Background location tracking is needed (unlikely for clock-in use case)

**If none of these happen, the PWA is sufficient and no native app is needed.**

---

## Architecture: Code Reuse via Monorepo

The codebase is already structured as a **monorepo** specifically to support mobile:

```
packages/core/     ← SHARED — reused by mobile with zero rewrite
├── api/           ← Supabase client calls
├── hooks/         ← useAuth, useClockIn, useTimesheets, useLeave, etc.
├── types/         ← All TypeScript types
├── utils/         ← Pure functions (geofence math, date helpers, formatting)
└── constants/     ← Config, role maps, thresholds

apps/web/          ← Current web app (React + Vite + TailwindCSS)
apps/mobile/       ← Future mobile app (React Native + Expo)
```

### What Gets Reused (zero rewrite):
- ✅ All API client functions (`packages/core/api/`)
- ✅ All hooks — useAuth, useClockIn, useTimesheets, useLeave, etc. (`packages/core/hooks/`)
- ✅ All TypeScript types (`packages/core/types/`)
- ✅ All utility functions (`packages/core/utils/`)
- ✅ All constants, config, and business rules (`packages/core/constants/`)

### What Gets Built Fresh:
- 🔨 Native UI components (React Native / NativeWind)
- 🔨 Navigation (Expo Router)
- 🔨 Platform APIs (native GPS, camera, biometrics, push)
- 🔨 Native styling
- 🔨 App Store assets and configuration

### Effort Comparison

| Approach | Effort vs Web | Notes |
|----------|---------------|-------|
| Without shared core | 60-70% of web effort | Rewriting all business logic |
| **With shared core (current architecture)** | **30-40% of web effort** | **UI + platform layer only** |

---

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | React Native (Expo managed) | Expo simplifies native builds |
| Navigation | Expo Router | File-based routing (familiar) |
| Styling | NativeWind | TailwindCSS syntax for React Native |
| State / Logic | `@tyrepower/core` hooks | Shared from monorepo |
| Local Storage | Expo SecureStore + SQLite | Secure tokens + offline data |
| Maps | react-native-maps | Native MapView for geofence display |
| Camera | expo-camera | QR code scanning |
| Push | expo-notifications | Native push (APNs + FCM) |
| Biometrics | expo-local-authentication | Face ID / fingerprint |
| Location | expo-location | Background GPS tracking |
| Build & Deploy | EAS Build + EAS Submit | Expo Application Services |

---

## Mobile Phases

### Phase M1: Core — Clock-In/Out (3-4 weeks)

| Task | Description | Estimate |
|------|-------------|----------|
| Project setup | Expo init, link `@tyrepower/core`, NativeWind config | 4h |
| Auth screens | Login, forgot password (reuses `useAuth` hook) | 6h |
| Biometric login | Face ID / fingerprint after initial login | 4h |
| Navigation | Tab bar (Dashboard, Clock, Timesheets, Profile) | 4h |
| Employee dashboard | Today's hours, active shift, weekly summary | 6h |
| Clock-in screen | Big button, GPS status, site name | 6h |
| Native GPS | Background location with expo-location | 4h |
| Geofence validation | Uses shared `calculateDistance` util | 2h |
| QR scanner | Native camera QR scan → clock-in | 4h |
| Break tracking | Start/end break (paid/unpaid) during active shift | 4h |
| Active shift display | Live timer, break timer | 4h |
| Clock-out flow | End shift, duration display | 3h |
| Offline clock-in | SQLite queue, sync when online | 6h |

**M1 Deliverable:** Employees can clock in/out with native GPS, QR scan, take breaks, and work offline.

---

### Phase M2: Timesheets & Notifications (2-3 weeks)

| Task | Description | Estimate |
|------|-------------|----------|
| Timesheet list | Weekly timesheets with status badges | 4h |
| Timesheet detail | Shifts, breaks, hours summary, notes | 4h |
| Submit timesheet | Review + submit flow | 3h |
| Manager: approval list | Pending timesheets for assigned sites | 4h |
| Approve/reject | Quick actions with notes | 3h |
| Native push setup | expo-notifications + APNs/FCM registration | 4h |
| Push handlers | Timesheet status, overtime alerts, leave responses | 3h |
| Notification inbox | In-app notification list | 3h |

**M2 Deliverable:** Full timesheet workflow on mobile. Native push notifications working.

---

### Phase M3: Full Features (3-4 weeks)

| Task | Description | Estimate |
|------|-------------|----------|
| Leave request | Request form (type, dates, notes) | 4h |
| Leave balances | View current balances | 2h |
| Leave calendar | Team calendar view (manager) | 4h |
| Profile management | Avatar, phone, password change | 3h |
| Manager dashboard | Clocked-in count, overtime alerts, pending timesheets | 6h |
| Site map | Native MapView with geofence boundary | 4h |
| Shift history | Filterable list of past shifts | 3h |
| Overtime indicators | Warning badges, weekly hours tracker | 3h |
| Reports viewer | View reports (export via share sheet) | 4h |
| Announcements | View site announcements | 2h |
| Roster view | Upcoming shifts schedule (if Phase 4 built) | 4h |

**M3 Deliverable:** Feature-complete mobile app matching web capabilities.

---

### Phase M4: Polish & App Store Launch (2-3 weeks)

| Task | Description | Estimate |
|------|-------------|----------|
| App icons & splash | Tyrepower branded (all sizes) | 3h |
| App Store screenshots | iPhone + iPad | 3h |
| Store listing copy | Description, keywords, privacy policy | 2h |
| iOS testing | TestFlight beta with real users | 4h |
| Android testing | Internal testing track on Play Store | 4h |
| Performance optimization | Launch time, scroll performance, memory | 4h |
| Crash monitoring | Sentry React Native SDK | 2h |
| Accessibility | VoiceOver (iOS), TalkBack (Android) | 4h |
| App Store submission | iOS App Store + Google Play | 3h |
| Launch coordination | Notify stores, update training materials | 2h |

**M4 Deliverable:** App published on iOS App Store and Google Play Store.

---

## Timeline Summary

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| **M1: Core** | 3-4 weeks | Clock-in/out, GPS, QR, breaks, offline |
| **M2: Timesheets** | 2-3 weeks | Timesheets, approvals, push notifications |
| **M3: Full Features** | 3-4 weeks | Leave, reports, dashboards, roster |
| **M4: Launch** | 2-3 weeks | App Store submission, testing, polish |
| **Total** | **10-14 weeks** | |

---

## Decision Criteria

**Do NOT build the native app unless:**

- [ ] PWA has been in production for 3+ months
- [ ] Multiple stores actively using it daily
- [ ] Users have reported specific PWA limitations that can't be worked around
- [ ] The limitations are blocking real workflows (not just "nice to have")
- [ ] Budget/time available for 10-14 weeks of additional development

**If the PWA works well (which it likely will), skip this entirely.**

---

## Cost Considerations

| Item | Cost | Notes |
|------|------|-------|
| Apple Developer Account | $149 AUD/year | Required for App Store |
| Google Play Developer | $35 USD one-time | Required for Play Store |
| EAS Build (Expo) | Free tier or $99/mo | Cloud builds for iOS/Android |
| Sentry (mobile) | Included in web plan | Same project, mobile SDK |
| Push notifications | Included (APNs/FCM free) | No additional cost |

---

## Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| App Store rejection | Delayed launch | Follow guidelines strictly, test with TestFlight first |
| iOS/Android fragmentation | Bugs on some devices | Test on 3+ real devices per platform |
| Expo limitations | Can't access native API | Use custom dev client or bare workflow if needed |
| User adoption | Low downloads | Push migration from PWA, in-app banner |
| Maintenance burden | Two codebases to maintain | Shared core minimizes divergence |
