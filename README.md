# School Bus Tracking System
> **Role:** Freelance Full-Stack Developer  
> **Stack:** Laravel · MySQL · Kotlin · Android · Firebase Cloud Messaging · REST APIs · JWT  
> 🔗 **Type:** Freelance client project — deployed in production

---

## Overview

A real-time school bus tracking system built for a private school client, giving parents live visibility of their child's status throughout the school day — from home pickup through to school arrival and afternoon drop-off. The system consists of a Laravel backend API and two separate Kotlin Android applications: one for drivers and one for parents.

---

## The Problem

The school had no reliable way for parents to know whether their child had been picked up, was on the bus, or had arrived safely at school. Drivers had no standardised way to log student status updates. The result was frequent anxious calls to the school office from parents — a burden on staff and a source of daily stress for families.

---

## What I Built

### 1. Laravel Backend API
The backend is the single source of truth for all student status data.

**Key features:**
- **RESTful API** built with Laravel handling all business logic — student management, route management, status updates, and push notification triggers
- **Laravel Sanctum** for authentication — separate token scopes for drivers and parents with role-based access control
- **MySQL database** storing students, routes, drivers, parents, and status event history
- **Role-based permissions** — drivers can only write status updates, parents can only read them. Cross-role data access is blocked at the API level
- **Push notification triggers** — on every status update, the backend fires a Firebase Cloud Messaging (FCM) notification to the relevant parent's device
- Status event history — full audit log of every status change with timestamp and driver ID

**Status states:**
```
at_home → picked_up → on_route → arrived_at_school → ready_for_pickup → dropped_off
```

---

### 2. Kotlin Driver App (Android)
The app used by school bus drivers throughout their route.

**Features:**
- Authenticates against the Laravel API using bearer tokens
- Loads the driver's assigned route and student list for the day
- One-tap status update per student at each stop — designed for use while managing a bus
- Works on low-end Android devices common among drivers
- Offline-tolerant — status updates queue locally if connectivity drops and sync when reconnected

---

### 3. Kotlin Parent App (Android)
The app used by parents to track their child's status in real time.

**Features:**
- Authenticates via the Laravel API — each parent account is linked to their child's record
- Real-time status display — current status updates as the driver taps through the route
- Push notifications via **Firebase Cloud Messaging (FCM)** — parents receive an instant notification when their child is picked up and when they are dropped off
- Status history — parents can view the full timeline of their child's day
- Parents cannot see other children's status — strict data scoping at the API level

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Client Layer                          │
│   Driver Android App (Kotlin)  │  Parent Android App    │
│   [Status updates]             │  [Status reads + FCM]  │
└──────────┬──────────────────────────────┬───────────────┘
           │ REST API (Bearer Token)       │
           │ Laravel Sanctum Auth          │
┌──────────▼───────────────────────────────▼──────────────┐
│                Laravel Backend API                       │
│  Auth │ Student Mgmt │ Route Mgmt │ Status Updates       │
│  Role-based access: drivers write, parents read          │
└──────────┬───────────────────────────────────────────────┘
           │
┌──────────▼──────────────┐  ┌───────────────────────────┐
│   MySQL Database        │  │  Firebase Cloud Messaging  │
│   Students │ Routes     │  │  Push notifications        │
│   Drivers │ Parents     │  │  to parent devices         │
│   Status event log      │  └───────────────────────────┘
└─────────────────────────┘
```

---

## Key Technical Decisions

| Decision | Rationale |
|---|---|
| Laravel for backend | Client requirement + strong ecosystem for rapid API development with built-in auth, ORM, and validation |
| Sanctum over JWT library | Laravel Sanctum is the recommended first-party solution for mobile token auth — simpler and well-maintained |
| Kotlin for Android | Native Android gave better performance and offline handling than React Native for a low-connectivity environment |
| FCM for push notifications | Free, reliable, and well-supported on Android. Drivers and parents are on Android devices |
| Role scoping at API level | Enforcing permissions in the API — not just the app UI — means even direct API calls can't access cross-role data |
| Status event log | Audit trail was a client requirement — parents and school admin needed to review the day's events if a dispute arose |

---

## Challenges & Solutions

**Challenge: Connectivity drops on rural routes**
Drivers sometimes lost mobile data mid-route. Status updates couldn't be lost.

**Solution:** The driver app queues updates locally in SQLite when offline and syncs them to the Laravel API in order when connectivity restores. The backend accepts out-of-order timestamps and sorts the event log correctly.

---

**Challenge: Duplicate notifications**
Early testing showed parents occasionally received double notifications when the Laravel API retried a failed FCM request.

**Solution:** Added idempotency on notification dispatch — each status update event has a unique ID, and the notification service checks if a notification for that event ID has already been sent before firing.

---

## Metrics & Outcomes

| Metric | Value |
|---|---|
| Students tracked | Full school enrollment |
| Status updates per day | ~4 per student (pickup + arrival + ready + dropoff) |
| Parent anxiety calls to school office | Significantly reduced post-launch |
| Platforms | Android (Driver + Parent apps) |
| Notification delivery | Real-time via FCM |

---

## What I Learned

This project was my first time building a complete system with **Laravel as the primary backend** and **Kotlin for native Android**. The biggest learning was around **offline-first mobile design** — on a school bus route, connectivity is not guaranteed, and losing a status update is unacceptable. Designing the driver app to queue locally and sync reliably changed how I think about mobile apps in low-connectivity environments. It also reinforced the importance of **role-based access control done at the API level** — not just the UI — because the safety of a child's location data is not something to leave to client-side enforcement.

---

*Freelance project built and deployed for a private school client. Codebase is in a private repository.*
