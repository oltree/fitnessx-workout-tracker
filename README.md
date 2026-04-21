# FitnessX - Workout Tracker

> [Russian version / Русская версия](README.ru.md)

**FitnessX** is a modern fitness web application for creating, scheduling, and tracking personal workouts - built entirely with a React + TypeScript frontend, without a backend server.

**Live demo:** https://oltree.github.io/fitnessx

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Architecture](#project-architecture)
- [State Management](#state-management)
- [Routing & Auth Guards](#routing--auth-guards)
- [API Integration](#api-integration)
- [Getting Started](#getting-started)
- [Deployment](#deployment)
- [Contact](#contact)

---

## Overview

FitnessX lets users register, log in, plan workouts on an interactive calendar, add individual exercises with custom parameters, and track completion progress - all stored in the browser's `localStorage` through Redux-Persist, requiring no backend.

The app also integrates with the **OpenWeatherMap API** to provide smart weather-based recommendations when scheduling outdoor leg workouts.

---

## Features

| Feature | Details |
|---|---|
| **Authentication** | Registration & login with email/password, validated against localStorage |
| **Home Dashboard** | BMI banner, workout creation form, mini calendar view |
| **Workout Builder** | Multi-step form: date picker → exercise list → workout naming |
| **Exercise Parameters** | Sets, reps, duration, weight - configurable per exercise |
| **Interactive Calendar** | Full calendar (month/week/day views) with drag-and-drop rescheduling |
| **Workout Detail Page** | View exercises, toggle completion status per exercise |
| **Notifications Feed** | Login events, registration confirmations, weather alerts |
| **Profile Page** | User info, full workout history with exercise breakdowns, logout |
| **Weather Integration** | Checks current weather (Minsk/Moscow) when scheduling leg workouts |

---

## Tech Stack

### Core
| Technology | Version | Purpose |
|---|---|---|
| React | 18.2.0 | UI framework |
| TypeScript | 4.9.5 | Static typing |
| React Router DOM | 6.11.1 | Client-side routing |

### State Management
| Technology | Version | Purpose |
|---|---|---|
| Redux Toolkit | 1.9.5 | State management with slices |
| Redux-Persist | 6.0.0 | Persisting state to localStorage |
| React-Redux | 8.0.5 | React bindings and hooks |

### Forms & UI
| Technology | Version | Purpose |
|---|---|---|
| React Hook Form | 7.43.9 | Form validation and state |
| FullCalendar | 6.1.6 | Interactive calendar with drag-and-drop |
| React DatePicker | 4.11.0 | Date selection UI |
| Classnames | 2.3.2 | Conditional CSS class composition |
| UUID | 9.0.0 | Unique ID generation for entities |

### Styling
| Technology | Version | Purpose |
|---|---|---|
| SCSS Modules | - | Scoped component styles |
| Sass | 1.62.1 | CSS preprocessor |

### HTTP & External APIs
| Technology | Version | Purpose |
|---|---|---|
| Axios | 1.4.0 | HTTP client for OpenWeatherMap API |

### Build & Tooling
| Technology | Version | Purpose |
|---|---|---|
| Craco | 7.1.0 | CRA config extension (webpack aliases) |
| React Scripts | 5.0.1 | Build toolchain |
| gh-pages | 5.0.0 | GitHub Pages deployment |
| Prettier | 2.8.8 | Code formatting |

---

## Project Architecture

```
src/
├── assets/
│   ├── icons/               # SVG exercise and navigation icons
│   ├── images/              # Banner image, 404 illustration
│   └── styles/              # Global SCSS variables, mixins, animations
│
├── components/
│   ├── common/              # Reusable UI components
│   │   ├── banner/          # BMI info banner
│   │   ├── buttons/         # Button, IconButton, RemoveButton
│   │   ├── calendar/        # FullCalendar wrapper component
│   │   ├── exercise-form/   # Form for a single exercise entry
│   │   ├── input/           # Controlled Input with error display
│   │   ├── loader/          # Loading spinner
│   │   ├── radio-group/     # Exercise type selection
│   │   └── workout-form/    # Multi-step workout creation flow
│   ├── layout/              # App shell: Layout, Header
│   └── screens/             # Page-level components
│       ├── Home.tsx
│       ├── Welcome.tsx
│       ├── SignIn.tsx / SignUp.tsx
│       ├── Workout.tsx      # Workout detail & exercise tracking
│       ├── Profile.tsx
│       ├── Notifications.tsx
│       └── NotFound.tsx
│
├── hooks/                   # Custom React hooks
│   ├── useAuth.ts           # Reads isAuth from Redux
│   ├── useUser.ts           # Current user info
│   ├── useWorkouts.ts       # All persisted workouts
│   ├── useWorkout.ts        # Currently viewed workout
│   ├── useExercises.ts      # Exercises in the creation form
│   ├── useNotifications.ts  # Notification list
│   ├── useRedirect.ts       # Delayed navigation helper
│   └── hooks.ts             # Typed useAppDispatch / useAppSelector
│
├── routes/
│   ├── routes.ts            # Route definitions with auth flags
│   └── Router.tsx           # Auth-guarded route rendering
│
├── services/
│   └── weather.service.ts   # OpenWeatherMap API singleton
│
├── store/
│   ├── store.ts             # Redux store + persistence config
│   ├── slices/              # Redux Toolkit slices (5 slices)
│   └── selectors/           # Memoized selectors per domain
│
├── types/                   # TypeScript interfaces and enums
│   ├── user.type.ts
│   ├── workout.type.ts
│   ├── exercise.type.ts
│   ├── notification.type.ts
│   ├── event.type.ts
│   ├── route.type.ts
│   └── enams/               # Enums: exercises, cities, days
│
└── utils/
    ├── constants/           # Exercise icon/name mappings, date format
    └── helpers/             # Pure utility functions
```

---

## State Management

The app uses **Redux Toolkit** with **Redux-Persist**. Five slices handle all application state:

| Slice | Persisted | Contents |
|---|---|---|
| `user` | Yes | `email`, `firstName`, `lastName`, `isAuth` |
| `workouts` | Yes | Full list of `Workout[]`, sorted by date |
| `exercises` | No | Temporary exercise list during workout creation |
| `notifications` | No | Feed of notification entries |
| `workout` (current) | No | Currently viewed workout for detail/tracking page |

Only `user` and `workouts` are persisted to `localStorage` - session-only state is kept in Redux without persistence.

Custom hooks (`useWorkouts`, `useUser`, etc.) wrap memoized selectors to keep components decoupled from the Redux API.

---

## Routing & Auth Guards

| Route | Component | Auth Required |
|---|---|---|
| `/` | Welcome | No |
| `/signup` | SignUp | No |
| `/signin` | SignIn | No |
| `/home` | Home | Yes |
| `/workout` | Workout | Yes |
| `/notifications` | Notifications | Yes |
| `/profile` | Profile | Yes |
| `*` | NotFound | - |

The `Router.tsx` component checks the `isAuth` flag from Redux state and conditionally renders protected or public routes. Unauthenticated users are redirected to `/`.

---

## API Integration

### Weather Service (`src/services/weather.service.ts`)

Singleton class integrating with **OpenWeatherMap API**:
- `getWeatherData(city)` - fetches current weather for Minsk or Moscow
- `isBadWeather(activity, data)` - returns `true` for leg workouts when: rain is present **or** temperature < 10°C
- `getWeatherMessage(city, activity)` - returns a user-facing recommendation string

The weather check triggers automatically when the user adds a **legs** exercise, creating a notification with the result.

### Authentication (localStorage)

No backend API. Users are stored as JSON in `localStorage['users']`. On login, the app looks up the entry by email and validates the password, then writes auth state to Redux (persisted via Redux-Persist).

---

## Getting Started

### Prerequisites
- Node.js 16+
- npm or yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/oltree/fitnessx.git
cd fitnessx

# Install dependencies
npm install

# Start development server
npm run dev
```

The app will be available at `http://localhost:3000`.

### Available Scripts

| Script | Description |
|---|---|
| `npm run dev` | Start development server via Craco |
| `npm run build` | Production build |
| `npm test` | Run tests |
| `npm run deploy` | Build and deploy to GitHub Pages |

---

## Deployment

Deployed to **GitHub Pages** using the `gh-pages` package.

- Homepage: https://oltree.github.io/fitnessx
- Build output: `/build` directory
- Base URL: `/fitnessx` (set via `PUBLIC_URL` in `.env`)

To deploy manually:
```bash
npm run deploy
```

---

## Contact

**Oleg Melekh** - Frontend Developer  
Email: oleg.melekh.frontend@gmail.com
