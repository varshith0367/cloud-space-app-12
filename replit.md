# Cloud Space — Replit Project Guide

## Overview

Cloud Space is a premium, AI-powered digital workspace mobile app built for students, developers, and startup founders. It lets users collaborate on projects, manage tasks, store files, and access AI tools — all in one clean, investor-ready interface.

The app is built with **Expo (React Native)** using **Expo Router** for file-based navigation. It has a companion **Express.js backend** server and uses **PostgreSQL via Drizzle ORM** for persistent data storage. The UI targets Android and iOS primarily, with web as a secondary target.

Key features:
- Authentication (email/password + Google Sign-In simulation)
- Home dashboard with activity feed and AI suggestions
- Workspace management with task boards
- AI Tools panel (Code Helper, Content Generator, Summarizer, Resume Reviewer)
- User profile with subscription plan management
- Light and dark mode support

---

## User Preferences

Preferred communication style: Simple, everyday language.

---

## System Architecture

### Frontend (Expo / React Native)

The app uses **Expo Router** (file-based routing) to organize screens into logical groups:

- `app/(auth)/` — Login, Signup, Forgot Password screens
- `app/(tabs)/` — Main tabbed interface: Home, Workspace, AI Tools, Profile
- `app/workspace/[id].tsx` — Dynamic route for individual workspace detail views
- `app/subscription.tsx` — Subscription/plan upgrade modal

**Navigation flow:**
1. App starts, checks `AuthContext` for a stored user session
2. If no session → redirect to `/(auth)/login`
3. If session exists → redirect to `/(tabs)` (home dashboard)

**UI design system:**
- Color palette defined centrally in `constants/colors.ts` (navy `#0F172A`, electric blue `#3B82F6`, white)
- Glassmorphism and blur effects via `expo-blur` and `expo-glass-effect`
- Animations with `react-native-reanimated`
- Gradients via `expo-linear-gradient`
- Icons via `@expo/vector-icons` (Feather and Ionicons sets)
- Typography: Inter font loaded via `@expo-google-fonts/inter`
- Platform-adaptive tabs: Uses `expo-glass-effect`'s Liquid Glass tabs on supported iOS, falls back to classic `expo-router` Tabs on Android/web

**State management:**
- `AuthContext` (`context/AuthContext.tsx`) handles user session globally using React Context + `useState`
- `@tanstack/react-query` handles server data fetching and caching
- `AsyncStorage` persists user session and workspace data locally on the device

### Backend (Express.js)

Located in `server/`, the backend is a Node.js Express server:

- `server/index.ts` — Entry point, sets up CORS (Replit-domain-aware), middleware, and registers routes
- `server/routes.ts` — Route registration (currently scaffold, routes go under `/api/` prefix)
- `server/storage.ts` — Storage abstraction layer with `IStorage` interface; currently uses `MemStorage` (in-memory), designed to be swapped for a database-backed implementation

The server serves both the API and (in production) the pre-built static Expo web bundle.

**CORS** is configured to allow requests from Replit dev/production domains and localhost for development.

### Database

- **PostgreSQL** is used as the database (configured via `DATABASE_URL` environment variable)
- **Drizzle ORM** manages schema definition and migrations
- Schema lives in `shared/schema.ts` — currently defines a `users` table with `id`, `username`, and `password`
- `drizzle-zod` generates Zod validation schemas from the Drizzle table definitions
- Migrations output to `./migrations/`
- Run `npm run db:push` to push schema changes

The `shared/` directory is imported by both the frontend and backend, enabling shared types.

### Authentication

Currently **client-side mock authentication** using `AsyncStorage`:
- `login()` creates a mock user object and saves it to AsyncStorage
- `signup()` creates a new mock user with a role and plan
- `logout()` clears the stored user
- `updateUser()` merges partial updates into the stored user

The `server/storage.ts` `IStorage` interface is prepared for real server-side user management. JWT-based authentication and real password hashing need to be wired up to replace the mock implementation.

### Data Flow (Client ↔ Server)

- `lib/query-client.ts` sets up a TanStack Query client with a custom `apiRequest()` function
- API requests use `expo/fetch` and target the Express server at the domain defined by `EXPO_PUBLIC_DOMAIN`
- The base URL is derived from the `EXPO_PUBLIC_DOMAIN` environment variable (set automatically in Replit dev scripts)

---

## External Dependencies

| Dependency | Purpose |
|---|---|
| `expo` (~54) | Core Expo SDK and build tooling |
| `expo-router` (~6) | File-based navigation for React Native |
| `react-native-reanimated` | Smooth animations |
| `expo-linear-gradient` | Gradient backgrounds |
| `expo-blur` | Blur/glassmorphism effects |
| `expo-glass-effect` | Liquid Glass tab bar (iOS) |
| `expo-haptics` | Haptic feedback |
| `expo-image-picker` | File/image selection |
| `@expo-google-fonts/inter` | Inter font family |
| `@expo/vector-icons` | Feather, Ionicons icon sets |
| `@tanstack/react-query` | Server state management and caching |
| `@react-native-async-storage/async-storage` | Local persistent storage |
| `drizzle-orm` | ORM for PostgreSQL |
| `drizzle-zod` | Schema validation from Drizzle models |
| `drizzle-kit` | Drizzle migration CLI |
| `pg` | PostgreSQL Node.js client |
| `express` (v5) | Backend API server |
| `http-proxy-middleware` | Dev proxy support |
| `react-native-gesture-handler` | Gesture support |
| `react-native-keyboard-controller` | Keyboard-aware scroll handling |
| `react-native-safe-area-context` | Safe area insets |
| `react-native-screens` | Native screen navigation performance |
| `react-native-svg` | SVG rendering |
| `expo-location` | Location access (for future features) |

**Environment Variables Required:**
- `DATABASE_URL` — PostgreSQL connection string (required by server and Drizzle)
- `EXPO_PUBLIC_DOMAIN` — The domain the Express server is reachable at (auto-set by Replit dev scripts)
- `REPLIT_DEV_DOMAIN` — Set by Replit, used for CORS and dev URL configuration
- `REPLIT_DOMAINS` — Set by Replit for production domain CORS