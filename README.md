# Language Traveler

**A cross-platform mobile app for learning languages through curated articles, structured courses, and spaced-repetition vocabulary practice.**

![React Native](https://img.shields.io/badge/React_Native-0.81-61DAFB?logo=react&logoColor=white)
![Expo](https://img.shields.io/badge/Expo_SDK-54-000020?logo=expo&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-BaaS-3FCF8E?logo=supabase&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-lightgrey)

## Overview

Language Traveler is a full-featured language learning mobile application built from the ground up as a solo project. It combines article-based reading practice, structured course lessons, and SRS-powered vocabulary review into a single cohesive experience — currently supporting English, Spanish, and Turkish.

The app follows a local-first architecture: user progress, saved words, and content are cached on-device using MMKV and Zustand persistence, with Supabase handling backend data and authentication. TanStack Query manages all server state with offline-aware caching and automatic revalidation, ensuring the app remains responsive even with intermittent connectivity.

Built with Expo's managed workflow and delivered to both TestFlight and Google Play Console via EAS Build, the project covers the full lifecycle — from authentication (Apple Sign-In, Google OAuth, passwordless email OTP) and push notifications to OTA updates and gamification with an XP system.

## Tech Stack

| Category              | Technology                               | Purpose                                     |
| --------------------- | ---------------------------------------- | ------------------------------------------- |
| **Framework**         | React Native 0.81 + Expo SDK 54          | Cross-platform mobile (iOS & Android)       |
| **Language**          | TypeScript 5.9                           | Type safety across the entire codebase      |
| **Runtime**           | Hermes                                   | Optimized JS engine for React Native        |
| **Navigation**        | Expo Router 6                            | File-based routing with layout groups       |
| **State (Client)**    | Zustand 4                                | Client state with 11 persisted slices       |
| **State (Server)**    | TanStack React Query 5                   | Server state, caching & offline persistence |
| **Backend**           | Supabase                                 | Auth, database, real-time                   |
| **Auth**              | Apple Sign-In · Google OAuth · Email OTP | Three passwordless auth providers           |
| **Storage**           | MMKV · Expo SecureStore                  | Fast KV store + encrypted auth tokens       |
| **Localization**      | i18next + react-i18next                  | Multi-language UI (EN, ES, TR)              |
| **Animations**        | React Native Reanimated 4                | Gesture-driven & layout animations          |
| **Lists**             | Shopify FlashList                        | High-performance virtualized lists          |
| **Networking**        | NetInfo + custom retry                   | Offline detection & resilient fetching      |
| **Notifications**     | Expo Notifications                       | Push notifications with preferences         |
| **Push Delivery**     | Firebase FCM + APNs                      | Remote push notifications (iOS & Android)   |
| **OTA Updates**       | EAS Update                               | Over-the-air JS bundle updates              |
| **Haptics**           | Expo Haptics                             | Tactile feedback on interactions            |
| **Images**            | Expo Image                               | Performant image loading with caching       |
| **UI Polish**         | Expo Blur · Linear Gradient · SVG        | Visual effects and vector graphics          |
| **Edge Functions**    | Supabase Edge Functions                  | Serverless business logic                   |
| **Database Security** | Supabase RLS                             | Row Level Security on all tables            |
| **Server Logic**      | Supabase RPC + pg_cron                   | Server-side functions & scheduled jobs      |
| **Admin Panel**       | Next.js + Vercel                         | Content management & notification control   |
| **Testing**           | Jest + React Native Testing Library      | Unit and component tests                    |
| **Build & Deploy**    | EAS Build                                | Cloud builds for TestFlight & Play Console  |

## Screenshots

<!-- Add 3-4 app screenshots here -->

## Architecture

### Project Structure

```
app/                          # Expo Router — file-based routing
├── (auth)/                   # Auth screens (login, register, privacy)
├── (onboarding)/             # Onboarding flow
├── (tabs)/                   # Main tab navigator (home, explore, profile)
├── article/                  # Article detail & reading screens
├── course/                   # Course detail & unit screens
├── learn/lesson/             # Lesson & quiz screens
└── word/                     # Word detail screens

src/
├── components/               # UI components by feature
│   ├── auth/                 #   Auth-specific (rate limit countdown)
│   ├── cards/                #   Content cards
│   ├── completion/           #   XP completion modals
│   ├── forms/                #   Language picker, input forms
│   ├── loading/              #   Skeleton loaders
│   ├── notifications/        #   Swipeable notification items
│   ├── study/                #   Study session components
│   ├── ui/                   #   Shared UI (toast, bottom sheet, flash list)
│   ├── word/                 #   Word display & review
│   ├── xp-animation/         #   XP animation provider & overlays
│   └── xp-display/           #   XP badge, progress bar, streak display
├── constants/                # Environment config, app constants
├── hooks/                    # Custom hooks
│   └── queries/              #   TanStack Query hooks (articles, courses, words, gamification)
├── lib/                      # Core infrastructure
│   ├── storage/              #   MMKV + SecureStore adapters
│   └── supabase/             #   Client, auth providers, network-aware fetch
├── localization/             # i18next translations
│   ├── en/                   #   English (10 namespace files)
│   ├── es/                   #   Spanish
│   └── tr/                   #   Turkish
├── services/                 # Business logic (rate limiter, OTA updates, word progress)
├── store/                    # Zustand store
│   └── slices/               #   11 slices (auth, settings, study, articles, content, activity, cache, lessons, network, progress, toast)
├── styles/                   # Shared styles
├── types/                    # TypeScript type definitions
└── utils/                    # Utility functions & animation helpers
```

### Architectural Approach

The app follows a **feature-first, local-first** architecture with clear state separation. Client state (auth, settings, study progress, UI) lives in Zustand with MMKV persistence, while server state (articles, courses, word lists, gamification) is managed by TanStack React Query with offline-first caching and MMKV-backed persistence. This separation ensures the app remains fully functional offline while syncing seamlessly when connectivity returns.

### Data Flow

User actions flow from UI components through custom hooks, which dispatch to either Zustand slices (for client state) or TanStack Query mutations (for server state). Mutations hit Supabase via a network-aware fetch wrapper that checks connectivity, applies exponential backoff retry, and falls back to cached data when offline. Optimistic updates in React Query provide instant UI feedback while the backend sync happens asynchronously.

## Key Features

### Authentication

Three passwordless authentication providers are implemented, each with its own dedicated service module and built-in rate limiting:

- **Apple Sign-In** — Native iOS authentication via `expo-apple-authentication` integrated with Supabase Auth. Platform-gated to iOS only.
- **Google OAuth** — Browser-based OAuth flow using `expo-auth-session` and `expo-web-browser`, with token extraction from callback URLs supporting both hash and query parameter formats.
- **Email OTP** — Passwordless magic code authentication through Supabase's built-in OTP system. Supports send, verify, and resend flows with per-email and global rate limiting.

All three providers share a unified rate limiting system with sliding window algorithm, persistent attempt tracking via MMKV, and configurable block durations (e.g., 5 email OTP sends/hour, 10 OTP verifications/15 min, 35 global auth attempts/hour). Auth tokens are stored in iOS Keychain / Android Keystore via `expo-secure-store`, with automatic fallback to MMKV for oversized tokens (>2KB).

Account deletion is available in settings — it clears all local data (Zustand store, React Query cache, content cache, notification state, MMKV storage), signs out from Supabase, and redirects to login.

### Push Notifications

The notification system is built on Expo Notifications with Firebase FCM (Android) and APNs (iOS) for remote delivery:

- **Token Registration** — On app launch, the device's Expo push token is retrieved and stored in Supabase alongside device metadata (model, brand, platform). An Android notification channel is configured with max importance, vibration, and custom light color.
- **Notification Handling** — Foreground notifications display as alerts with sound and badge. Tapped notifications support deep linking via validated URL paths from notification data. Cold-start taps (app opened from killed state) are handled with a delayed response check.
- **Notification History** — Up to 10 recent notifications are fetched from Supabase, with 8 notification types mapped to distinct icons and colors (achievement, streak, reminder, daily reminder, word review, new content, course completion, default). Each notification displays a relative timestamp and an unread indicator.
- **User Preferences** — A per-user `notifications_enabled` toggle is synced to Supabase with optimistic UI updates and rollback on failure. Preferences auto-refresh when the app returns to foreground.
- **Swipe to Delete** — Notification items support left-swipe gesture deletion with animated card translation and fade-in delete button, powered by React Native Gesture Handler and Reanimated.

### Gamification

A full XP and progression system drives engagement across all learning activities:

- **XP Economy** — Nine distinct XP sources with defined values: word learned (4 XP), word reviewed (1 XP), article completed (20 XP), lesson completed (20 XP), daily activity (4 XP), with streak bonuses at 7-day (+12 XP) and 28-day (+32 XP) milestones.
- **Level System** — Users progress through levels based on accumulated XP. Level-up events trigger animated celebration modals with trophy icons, count-up XP animations, and haptic feedback.
- **Streak Tracking** — Daily activity streaks are tracked server-side via a `check_daily_activity_and_streak` RPC call. The UI displays current streak with contextual emojis and highlights 7-day and 28-day milestones with distinct styling.
- **XP Animations** — A queue-based animation system renders three animation types (XP earned, level up, streak bonus) sequentially via a React Context provider. Each animation has configurable duration, easing, and haptic feedback, with accessibility support for reduced motion preferences.
- **Activity Tracking** — Daily activity is recorded across four dimensions (words learned, articles read, reviews done, lessons completed) with weighted scoring for weekly activity summaries.

All XP mutations use TanStack Query with optimistic updates for instant UI feedback, and the entire gamification UI is localized across all three supported languages.

### Learning Engine

The core learning engine combines spaced repetition vocabulary practice with structured course progression:

- **Spaced Repetition (SRS)** — A 7-level fixed-interval schedule governs word review timing: 1 day, 3 days, 7 days, 1 month, 3 months, 6 months, 1 year. Words at mastery level 6+ are considered "mastered" and automatically removed from the hard words list. A daily cap of 2 level-ups per word prevents gaming the system.
- **Word Progress** — Each word tracks mastery level (0–7), next review date, and daily level-up count. Correct answers advance the level and schedule the next review; incorrect answers decrement the level and reset to a 1-day interval. Review queues are sorted by oldest review date first, then lowest mastery level.
- **Course & Lesson Structure** — Content is organized as Courses > Units > Lessons > Sentences + Quizzes. Lessons support four quiz types: multiple choice, true/false, fill in the blank, and sentence ordering. Lessons unlock sequentially — each requires completing the previous one.
- **Progress Resumption** — Lesson progress tracks sentence index, scroll position percentage, time spent, quiz scores, and attempt count, enabling users to resume exactly where they left off.
- **Dual-State Architecture** — A local `StudySlice` handles immediate UI state while a server-synced `ProgressSlice` serves as the source of truth. Sync uses a merge (union) strategy to prevent data loss, with an explicit replace option for full server resets.

### Localization

The app supports three languages (English, Spanish, Turkish) with full UI localization:

- **i18next Integration** — Translation files are dynamically loaded across 10 namespaces (common, navigation, onboarding, settings, study, profile, auth, premium, legal, gamification) with `@formatjs/intl-pluralrules` polyfills for Hermes compatibility.
- **System Language Detection** — On first launch, `expo-localization` detects the device language and falls back to English for unsupported languages. Users can override with a manual "system" or explicit language setting.
- **Language Switching** — Changing the learning or translation language validates against duplicates (auto-swaps if the same language is selected for both), clears the content cache, and triggers a full content reload to fetch language-appropriate material.

### Offline Support

The app is designed to work offline-first with multi-layer caching and graceful degradation:

- **Network Detection** — Real-time connectivity monitoring via `@react-native-community/netinfo` feeds a Zustand network slice. The app defaults to "online" optimistically until the listener reports otherwise.
- **MMKV Storage** — Two dedicated MMKV instances (main state + content cache) provide fast, synchronous key-value storage. Instances are lazily initialized on first access to reduce boot time. Auth tokens use a tiered strategy: `expo-secure-store` (Keychain/Keystore) as primary, with automatic MMKV fallback for oversized tokens.
- **React Query Persistence** — The query client is configured with `networkMode: 'offlineFirst'`, 5-minute stale time, 24-hour garbage collection, and MMKV-backed persistence. Only successful queries are persisted. `refetchOnReconnect: 'always'` ensures data freshness when connectivity returns.
- **Network-Aware Fetch** — A centralized `networkAwareFetch` wrapper checks online status before every API call, supports fallback data sources for offline scenarios, and offers silent-fail or throw-error strategies per call site.
- **Content Cache** — Large content arrays (articles, word lists, courses) are stored in a separate MMKV instance with a 24-hour TTL, avoiding bloat in the main Zustand persistence layer.

## Technical Challenges

### Zustand Hydration Race Condition

On iOS, the `onRehydrateStorage` callback in Zustand's persist middleware is asynchronous, meaning `index.tsx` can render before hydration completes. This caused the app to briefly show the onboarding screen to already-authenticated users. The fix ensures `_hasHydrated` is set to `true` in every code path — successful rehydration, null state, and error — so the root layout always waits for hydration before routing decisions.

### Data Migration Without Data Loss

As the state architecture evolved from a single study slice to a dedicated progress slice, persisted data needed migration. The rehydration logic merges old and new storage using Set unions (`[...new Set([...oldData, ...newData])]`), with the progress slice as the source of truth and old data as fallback. Both layers are kept in sync to prevent inconsistency during the transition.

### Multi-Layer Cache Architecture

Content caching required separating concerns across three layers: MMKV for fast synchronous access (dual instances for state vs. content), React Query for server state with offline-first network mode and selective persistence, and Zustand for client state with custom `partialize` to control what gets persisted. The content cache uses a 24-hour TTL with explicit validation on rehydration, and expired caches trigger a background refetch rather than blocking the UI.

### Auth Token Storage with Tiered Fallback

`expo-secure-store` has a ~2KB size limit per item, which some JWT tokens exceed. The auth storage adapter tries SecureStore first, falls back to MMKV for oversized tokens, and automatically cleans up duplicates when tokens migrate between stores. This ensures secure storage is used whenever possible without breaking authentication for edge cases.

### Production Splash Screen Delay (Android)

Content initialization was blocking the splash screen dismissal, causing 15–30 second waits on Android in production. The fix defers content loading to `InteractionManager.runAfterInteractions`, decoupling it from the splash screen lifecycle. An `AbortController` pattern prevents stale state updates if the initialization effect is re-triggered before completion.

### Client-Side Rate Limiting

To protect auth endpoints from abuse without relying solely on server-side limits, a sliding window rate limiter persists attempt timestamps in MMKV. Each auth action has granular limits (e.g., 5 email OTP sends/hour with a 30-minute block, 35 global auth attempts/hour). The UI displays countdown timers when rate-limited, and all limits can be cleared on successful authentication.

---

_This is a showcase repository. Source code is private._
_For inquiries, reach out via [LinkedIn](https://www.linkedin.com/in/anil-burcu/)._
