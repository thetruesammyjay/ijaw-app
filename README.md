# Ijaw Language Learning App — React Native

> A cross-platform mobile application for learning the Ijaw language through structured lessons, audio pronunciation, flashcards, and interactive quizzes. Built with React Native for Android and iOS from a single codebase.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [How the App Works](#how-the-app-works)
3. [Architecture](#architecture)
4. [Tech Stack](#tech-stack)
5. [File Structure](#file-structure)
6. [Database Schema](#database-schema)
7. [API Design](#api-design)
8. [Key Features Explained](#key-features-explained)
9. [Admin Dashboard](#admin-dashboard)
10. [Setup & Installation](#setup--installation)
11. [Environment Variables](#environment-variables)
12. [Deployment](#deployment)

---

## Project Overview

The Ijaw Language Learning App helps users learn the Kolokuma dialect of the Ijaw language through:

- Structured lesson modules organised by level
- Native speaker audio for every word and phrase
- Flashcard review sessions
- Multiple quiz formats
- Progress tracking and learning streaks
- Offline lesson access via downloaded content

The app targets Ijaw youths reconnecting with their heritage, non-Ijaw learners, and schools. Phase 1 focuses on the Kolokuma dialect; additional dialects are planned for Phase 2.

---

## How the App Works

### User Journey

```mermaid
flowchart TD
    A([App Launch]) --> B{Returning User?}
    B -- Yes --> C[Login Screen]
    B -- No --> D[Onboarding / Register]
    D --> E[Email Registration]
    E --> F[Email Verification]
    F --> G[Profile Setup]
    C --> H[Dashboard]
    G --> H

    H --> I[Browse Levels]
    I --> J[Select Level]
    J --> K[Lesson Screen]

    K --> L[Vocabulary Card]
    L --> M[Play Audio]
    M --> N[Next Word]
    N --> L
    N --> O{Lesson Complete?}
    O -- No --> L
    O -- Yes --> P[Quiz]

    P --> Q{Quiz Type}
    Q --> R[Multiple Choice]
    Q --> S[Match Word / Meaning]
    Q --> T[Listen and Choose]
    Q --> U[Fill in the Blank]

    R & S & T & U --> V[Quiz Result]
    V --> W[Update Progress + Streak]
    W --> X[Unlock Next Lesson]
    X --> H
```

### Authentication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant App as React Native App
    participant API as Backend API
    participant DB as Database

    U->>App: Enter email + password
    App->>API: POST /auth/login
    API->>DB: Validate credentials
    DB-->>API: User record
    API-->>App: JWT access token + refresh token
    App->>App: Store tokens in SecureStore
    App->>U: Redirect to Dashboard

    Note over App,API: Token refresh on expiry
    App->>API: POST /auth/refresh (refresh token)
    API-->>App: New access token
```

### Lesson Delivery Flow

```mermaid
sequenceDiagram
    participant U as User
    participant App as React Native App
    participant Cache as Local SQLite Cache
    participant API as Backend API
    participant S3 as Audio Storage (S3/CDN)

    U->>App: Open Lesson
    App->>Cache: Check if lesson is downloaded
    alt Lesson cached offline
        Cache-->>App: Return cached lesson data + audio
    else Not cached
        App->>API: GET /lessons/:id
        API-->>App: Lesson JSON (words, translations, audio URLs)
        App->>S3: Download audio files
        S3-->>App: Audio binary
        App->>Cache: Store lesson + audio locally
    end
    App->>U: Render lesson cards with audio
```

### Progress Tracking Flow

```mermaid
flowchart LR
    A[Lesson Completed] --> B[Calculate Score]
    B --> C[Update lesson_progress table]
    C --> D{First completion?}
    D -- Yes --> E[Unlock next lesson]
    D -- No --> F[Update best score]
    E & F --> G[Check streak]
    G --> H{Studied today?}
    H -- No --> I[Increment streak counter]
    H -- Yes --> J[Streak already updated today]
    I & J --> K[Sync to backend]
    K --> L[Update dashboard stats]
```

---

## Architecture

### High-Level System Architecture

```mermaid
graph TB
    subgraph Mobile["Mobile App (React Native)"]
        UI[UI Layer / Screens]
        RQ[React Query / State]
        SQ[SQLite Offline Cache]
        AS[Audio Player (expo-av)]
        SS[SecureStore (JWT)]
    end

    subgraph Backend["Backend (Node.js / Express)"]
        AUTH[Auth Service]
        LESSON[Lesson Service]
        QUIZ[Quiz Service]
        PROGRESS[Progress Service]
        NOTIF[Notification Service]
    end

    subgraph Admin["Admin Dashboard (React Web)"]
        ADM[Admin UI]
        CMS[Content Management]
    end

    subgraph Storage["Storage"]
        PG[(PostgreSQL)]
        S3[S3 / Cloudflare R2\nAudio Files]
        REDIS[(Redis Cache)]
    end

    UI --> RQ
    RQ --> AUTH
    RQ --> LESSON
    RQ --> QUIZ
    RQ --> PROGRESS
    AS --> S3
    RQ <--> SQ

    AUTH --> PG
    LESSON --> PG
    LESSON --> REDIS
    QUIZ --> PG
    PROGRESS --> PG
    NOTIF --> PG

    ADM --> CMS
    CMS --> LESSON
    CMS --> S3
    CMS --> PG
```

### Component Architecture

```mermaid
graph TD
    subgraph Navigation
        Stack[Stack Navigator]
        Tab[Bottom Tab Navigator]
        Drawer[Drawer Navigator]
    end

    subgraph Screens
        Home[HomeScreen]
        Lessons[LessonsScreen]
        LessonDetail[LessonDetailScreen]
        Flashcard[FlashcardScreen]
        Quiz[QuizScreen]
        Profile[ProfileScreen]
        Progress[ProgressScreen]
    end

    subgraph Components
        AudioCard[AudioCard]
        WordCard[WordCard]
        QuizOption[QuizOption]
        ProgressBar[ProgressBar]
        StreakBadge[StreakBadge]
        LevelBadge[LevelBadge]
    end

    subgraph State
        AuthStore[AuthStore - Zustand]
        LessonStore[LessonStore - Zustand]
        ProgressStore[ProgressStore - Zustand]
    end

    Stack --> Tab
    Tab --> Home
    Tab --> Lessons
    Tab --> Profile
    Lessons --> LessonDetail
    LessonDetail --> Flashcard
    LessonDetail --> Quiz

    LessonDetail --> AudioCard
    LessonDetail --> WordCard
    Quiz --> QuizOption
    Home --> ProgressBar
    Home --> StreakBadge
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile Framework | React Native (Expo managed workflow) |
| Navigation | React Navigation v6 |
| State Management | Zustand + React Query |
| Offline Storage | expo-sqlite (SQLite) |
| Audio Playback | expo-av |
| Authentication | JWT (access + refresh tokens) |
| Secure Storage | expo-secure-store |
| Backend | Node.js + Express |
| Database | PostgreSQL |
| Cache | Redis |
| Audio/File Storage | AWS S3 or Cloudflare R2 |
| Admin Dashboard | React + Vite + TailwindCSS |
| Push Notifications | Expo Notifications (FCM / APNs) |
| ORM | Prisma |

---

## File Structure

```
ijaw-language-app/
│
├── 📄 README.md
├── 📄 .gitignore
├── 📄 .env.example
│
├── 📁 mobile/                              # React Native App (Expo)
│   │
│   ├── 📄 app.json                         # Expo config
│   ├── 📄 App.tsx                          # Root component
│   ├── 📄 babel.config.js
│   ├── 📄 tsconfig.json
│   ├── 📄 package.json
│   ├── 📄 metro.config.js
│   │
│   ├── 📁 src/
│   │   │
│   │   ├── 📁 navigation/                  # React Navigation setup
│   │   │   ├── 📄 RootNavigator.tsx        # Root stack (Auth vs App)
│   │   │   ├── 📄 AppNavigator.tsx         # Bottom tab navigator
│   │   │   ├── 📄 AuthNavigator.tsx        # Auth stack (Login, Register)
│   │   │   ├── 📄 LessonNavigator.tsx      # Lesson stack
│   │   │   └── 📄 types.ts                 # Navigation param types
│   │   │
│   │   ├── 📁 screens/                     # App screens
│   │   │   │
│   │   │   ├── 📁 auth/
│   │   │   │   ├── 📄 LoginScreen.tsx
│   │   │   │   ├── 📄 RegisterScreen.tsx
│   │   │   │   ├── 📄 ForgotPasswordScreen.tsx
│   │   │   │   ├── 📄 ResetPasswordScreen.tsx
│   │   │   │   └── 📄 VerifyEmailScreen.tsx
│   │   │   │
│   │   │   ├── 📁 onboarding/
│   │   │   │   ├── 📄 WelcomeScreen.tsx
│   │   │   │   ├── 📄 OnboardingSlide1.tsx  # Why learn Ijaw
│   │   │   │   ├── 📄 OnboardingSlide2.tsx  # How it works
│   │   │   │   └── 📄 ProfileSetupScreen.tsx
│   │   │   │
│   │   │   ├── 📁 home/
│   │   │   │   ├── 📄 HomeScreen.tsx        # Dashboard, streak, progress summary
│   │   │   │   └── 📄 DailyGoalScreen.tsx
│   │   │   │
│   │   │   ├── 📁 lessons/
│   │   │   │   ├── 📄 LevelsScreen.tsx      # Level 1, 2, 3 overview
│   │   │   │   ├── 📄 LessonListScreen.tsx  # Lessons within a level
│   │   │   │   ├── 📄 LessonScreen.tsx      # Main lesson view (word cards)
│   │   │   │   ├── 📄 LessonCompleteScreen.tsx
│   │   │   │   └── 📄 DownloadedScreen.tsx  # Offline lessons
│   │   │   │
│   │   │   ├── 📁 flashcards/
│   │   │   │   ├── 📄 FlashcardDeckScreen.tsx
│   │   │   │   └── 📄 FlashcardReviewScreen.tsx
│   │   │   │
│   │   │   ├── 📁 quiz/
│   │   │   │   ├── 📄 QuizScreen.tsx        # Quiz container/controller
│   │   │   │   ├── 📄 MultipleChoiceScreen.tsx
│   │   │   │   ├── 📄 MatchWordScreen.tsx
│   │   │   │   ├── 📄 ListenChooseScreen.tsx
│   │   │   │   ├── 📄 FillBlankScreen.tsx
│   │   │   │   └── 📄 QuizResultScreen.tsx
│   │   │   │
│   │   │   ├── 📁 progress/
│   │   │   │   ├── 📄 ProgressScreen.tsx    # Overall progress view
│   │   │   │   └── 📄 AchievementsScreen.tsx
│   │   │   │
│   │   │   ├── 📁 profile/
│   │   │   │   ├── 📄 ProfileScreen.tsx
│   │   │   │   ├── 📄 EditProfileScreen.tsx
│   │   │   │   ├── 📄 SettingsScreen.tsx
│   │   │   │   └── 📄 ChangePasswordScreen.tsx
│   │   │   │
│   │   │   └── 📁 voice/                    # Phase 2 - voice practice
│   │   │       ├── 📄 VoicePracticeScreen.tsx
│   │   │       └── 📄 RecordingReviewScreen.tsx
│   │   │
│   │   ├── 📁 components/                   # Reusable UI components
│   │   │   │
│   │   │   ├── 📁 common/
│   │   │   │   ├── 📄 Button.tsx
│   │   │   │   ├── 📄 Input.tsx
│   │   │   │   ├── 📄 Card.tsx
│   │   │   │   ├── 📄 Modal.tsx
│   │   │   │   ├── 📄 LoadingSpinner.tsx
│   │   │   │   ├── 📄 ErrorMessage.tsx
│   │   │   │   ├── 📄 Badge.tsx
│   │   │   │   └── 📄 Avatar.tsx
│   │   │   │
│   │   │   ├── 📁 lesson/
│   │   │   │   ├── 📄 WordCard.tsx          # Word + translation + audio button
│   │   │   │   ├── 📄 AudioPlayer.tsx       # Play/pause/repeat controls
│   │   │   │   ├── 📄 LessonProgressBar.tsx
│   │   │   │   ├── 📄 LevelCard.tsx         # Level overview card
│   │   │   │   └── 📄 DownloadButton.tsx    # Offline download trigger
│   │   │   │
│   │   │   ├── 📁 flashcard/
│   │   │   │   ├── 📄 Flashcard.tsx         # Flip card animation
│   │   │   │   └── 📄 FlashcardProgress.tsx
│   │   │   │
│   │   │   ├── 📁 quiz/
│   │   │   │   ├── 📄 QuizOption.tsx        # Single answer choice
│   │   │   │   ├── 📄 QuizHeader.tsx        # Question number + timer
│   │   │   │   ├── 📄 AnswerFeedback.tsx    # Correct / incorrect feedback
│   │   │   │   └── 📄 ScoreCard.tsx
│   │   │   │
│   │   │   └── 📁 home/
│   │   │       ├── 📄 StreakCounter.tsx
│   │   │       ├── 📄 DailyGoalRing.tsx
│   │   │       └── 📄 RecentLessonCard.tsx
│   │   │
│   │   ├── 📁 store/                        # Zustand global state
│   │   │   ├── 📄 authStore.ts
│   │   │   ├── 📄 lessonStore.ts
│   │   │   ├── 📄 progressStore.ts
│   │   │   └── 📄 settingsStore.ts
│   │   │
│   │   ├── 📁 api/                          # API layer (React Query + axios)
│   │   │   ├── 📄 client.ts                 # Axios instance + interceptors
│   │   │   ├── 📄 auth.api.ts
│   │   │   ├── 📄 lessons.api.ts
│   │   │   ├── 📄 quiz.api.ts
│   │   │   ├── 📄 progress.api.ts
│   │   │   └── 📄 user.api.ts
│   │   │
│   │   ├── 📁 hooks/                        # Custom React hooks
│   │   │   ├── 📄 useAudioPlayer.ts         # expo-av wrapper
│   │   │   ├── 📄 useOfflineSync.ts         # Sync progress when back online
│   │   │   ├── 📄 useStreak.ts
│   │   │   ├── 📄 useDownload.ts            # Lesson download manager
│   │   │   └── 📄 useVoiceRecord.ts         # Phase 2
│   │   │
│   │   ├── 📁 db/                           # SQLite offline database
│   │   │   ├── 📄 database.ts               # DB init + migrations
│   │   │   ├── 📄 schema.ts                 # Table definitions
│   │   │   ├── 📄 lessons.db.ts             # Lesson CRUD ops
│   │   │   ├── 📄 progress.db.ts            # Progress CRUD ops
│   │   │   └── 📄 audio.db.ts               # Audio file path storage
│   │   │
│   │   ├── 📁 utils/
│   │   │   ├── 📄 formatters.ts             # Date, score formatters
│   │   │   ├── 📄 validators.ts
│   │   │   ├── 📄 storage.ts                # SecureStore wrappers
│   │   │   └── 📄 permissions.ts            # Mic/storage permissions
│   │   │
│   │   ├── 📁 theme/
│   │   │   ├── 📄 colors.ts                 # Brand colors
│   │   │   ├── 📄 typography.ts
│   │   │   ├── 📄 spacing.ts
│   │   │   └── 📄 index.ts
│   │   │
│   │   └── 📁 types/
│   │       ├── 📄 user.types.ts
│   │       ├── 📄 lesson.types.ts
│   │       ├── 📄 quiz.types.ts
│   │       └── 📄 progress.types.ts
│   │
│   └── 📁 assets/
│       ├── 📁 images/
│       │   ├── 🖼️ splash.png
│       │   ├── 🖼️ icon.png
│       │   ├── 🖼️ adaptive-icon.png
│       │   ├── 🖼️ onboarding-1.png
│       │   ├── 🖼️ onboarding-2.png
│       │   └── 🖼️ empty-state.png
│       └── 📁 fonts/
│           ├── 📄 Poppins-Regular.ttf
│           ├── 📄 Poppins-Bold.ttf
│           └── 📄 Poppins-Medium.ttf
│
├── 📁 backend/                             # Node.js + Express API
│   │
│   ├── 📄 package.json
│   ├── 📄 tsconfig.json
│   ├── 📄 .env.example
│   ├── 📄 Dockerfile
│   │
│   ├── 📁 src/
│   │   │
│   │   ├── 📄 app.ts                        # Express app setup
│   │   ├── 📄 server.ts                     # Entry point
│   │   │
│   │   ├── 📁 config/
│   │   │   ├── 📄 database.ts               # Prisma client
│   │   │   ├── 📄 redis.ts
│   │   │   ├── 📄 s3.ts                     # S3 / R2 client
│   │   │   └── 📄 env.ts                    # Validated env vars
│   │   │
│   │   ├── 📁 routes/
│   │   │   ├── 📄 auth.routes.ts
│   │   │   ├── 📄 users.routes.ts
│   │   │   ├── 📄 levels.routes.ts
│   │   │   ├── 📄 lessons.routes.ts
│   │   │   ├── 📄 flashcards.routes.ts
│   │   │   ├── 📄 quizzes.routes.ts
│   │   │   ├── 📄 progress.routes.ts
│   │   │   └── 📄 admin.routes.ts
│   │   │
│   │   ├── 📁 controllers/
│   │   │   ├── 📄 auth.controller.ts
│   │   │   ├── 📄 users.controller.ts
│   │   │   ├── 📄 lessons.controller.ts
│   │   │   ├── 📄 quizzes.controller.ts
│   │   │   ├── 📄 progress.controller.ts
│   │   │   └── 📄 admin.controller.ts
│   │   │
│   │   ├── 📁 services/
│   │   │   ├── 📄 auth.service.ts
│   │   │   ├── 📄 lesson.service.ts
│   │   │   ├── 📄 quiz.service.ts
│   │   │   ├── 📄 progress.service.ts
│   │   │   ├── 📄 audio.service.ts          # Upload + CDN URL generation
│   │   │   ├── 📄 email.service.ts
│   │   │   └── 📄 notification.service.ts
│   │   │
│   │   ├── 📁 middleware/
│   │   │   ├── 📄 auth.middleware.ts         # JWT verification
│   │   │   ├── 📄 admin.middleware.ts        # Admin role check
│   │   │   ├── 📄 rateLimit.middleware.ts
│   │   │   ├── 📄 upload.middleware.ts       # Multer config
│   │   │   └── 📄 validate.middleware.ts     # Zod schema validation
│   │   │
│   │   ├── 📁 validators/
│   │   │   ├── 📄 auth.schema.ts
│   │   │   ├── 📄 lesson.schema.ts
│   │   │   └── 📄 quiz.schema.ts
│   │   │
│   │   └── 📁 utils/
│   │       ├── 📄 jwt.ts
│   │       ├── 📄 hash.ts
│   │       ├── 📄 response.ts               # Standardised API response helpers
│   │       └── 📄 logger.ts
│   │
│   └── 📁 prisma/
│       ├── 📄 schema.prisma                 # Database schema
│       └── 📁 migrations/
│
└── 📁 admin/                               # Admin Dashboard (React + Vite)
    │
    ├── 📄 package.json
    ├── 📄 vite.config.ts
    ├── 📄 tailwind.config.js
    │
    └── 📁 src/
        ├── 📄 App.tsx
        │
        ├── 📁 pages/
        │   ├── 📄 Dashboard.tsx             # Stats overview
        │   ├── 📄 Levels.tsx               # Manage levels
        │   ├── 📄 Lessons.tsx              # Manage lessons
        │   ├── 📄 Words.tsx                # Add/edit words + audio upload
        │   ├── 📄 Quizzes.tsx              # Manage quiz questions
        │   ├── 📄 Users.tsx                # User management
        │   └── 📄 Settings.tsx
        │
        ├── 📁 components/
        │   ├── 📄 AudioUploader.tsx        # Drag and drop audio upload
        │   ├── 📄 WordForm.tsx
        │   ├── 📄 LessonForm.tsx
        │   ├── 📄 QuizBuilder.tsx          # Visual quiz question builder
        │   └── 📄 DataTable.tsx
        │
        └── 📁 api/
            └── 📄 admin.api.ts
```

---

## Database Schema

```mermaid
erDiagram
    users {
        uuid id PK
        string email UK
        string password_hash
        string full_name
        string avatar_url
        int streak_count
        date last_studied_at
        boolean email_verified
        timestamp created_at
    }

    levels {
        int id PK
        string title
        string description
        int order_index
        boolean is_published
    }

    lessons {
        uuid id PK
        int level_id FK
        string title
        string description
        int order_index
        boolean is_published
        timestamp created_at
    }

    words {
        uuid id PK
        uuid lesson_id FK
        string ijaw_word
        string english_translation
        string audio_url
        string phonetic
        int order_index
    }

    flashcard_decks {
        uuid id PK
        uuid lesson_id FK
        string title
    }

    quiz_questions {
        uuid id PK
        uuid lesson_id FK
        string question_type
        string question_text
        string audio_url
        string correct_answer
        json options
        int order_index
    }

    lesson_progress {
        uuid id PK
        uuid user_id FK
        uuid lesson_id FK
        boolean is_completed
        int best_quiz_score
        int times_completed
        timestamp completed_at
        timestamp updated_at
    }

    user_streaks {
        uuid id PK
        uuid user_id FK
        int current_streak
        int longest_streak
        date last_activity_date
    }

    downloaded_lessons {
        uuid id PK
        uuid user_id FK
        uuid lesson_id FK
        timestamp downloaded_at
    }

    users ||--o{ lesson_progress : "tracks"
    users ||--|| user_streaks : "has"
    users ||--o{ downloaded_lessons : "downloads"
    levels ||--o{ lessons : "contains"
    lessons ||--o{ words : "contains"
    lessons ||--o{ flashcard_decks : "has"
    lessons ||--o{ quiz_questions : "has"
    lessons ||--o{ lesson_progress : "tracked by"
    lessons ||--o{ downloaded_lessons : "downloaded in"
```

---

## API Design

All endpoints return JSON. Protected routes require `Authorization: Bearer <token>`.

### Auth
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `POST /api/auth/refresh`
- `POST /api/auth/forgot-password`
- `POST /api/auth/reset-password`
- `GET /api/auth/verify-email/:token`

### Lessons
- `GET /api/levels` — list all levels
- `GET /api/levels/:id/lessons` — lessons in a level
- `GET /api/lessons/:id` — lesson detail (words, audio URLs)
- `GET /api/lessons/:id/flashcards` — flashcard deck
- `GET /api/lessons/:id/quiz` — quiz questions

### Progress
- `POST /api/progress/lesson/:id/complete` — mark lesson complete + score
- `GET /api/progress/me` — user's full progress
- `GET /api/progress/streak` — current streak info

### Admin (role-protected)
- `POST /api/admin/words` — add word + upload audio
- `PUT /api/admin/words/:id`
- `DELETE /api/admin/words/:id`
- `POST /api/admin/lessons`
- `PUT /api/admin/lessons/:id`
- `POST /api/admin/quiz-questions`
- `GET /api/admin/users`
- `GET /api/admin/stats`

---

## Key Features Explained

### Offline Lessons

When a user downloads a lesson, the mobile app:

1. Fetches the full lesson JSON from the API
2. Downloads all audio files to the device's file system via `expo-file-system`
3. Stores the lesson data and local audio paths in SQLite
4. On subsequent opens, the app checks SQLite first before hitting the network

Progress made offline is queued locally and synced to the backend when connectivity is restored.

### Audio Pronunciation

Audio files are hosted on S3/Cloudflare R2 and served via CDN. Each `WordCard` component loads the audio URL from the lesson data and uses `expo-av` to play it. Users can replay as many times as they want. The admin panel allows uploading `.mp3` files directly, which are stored in object storage and the URL saved to the `words` table.

### Streak System

A `user_streaks` record tracks the current streak. After any lesson completion, the backend checks `last_activity_date`. If the last activity was yesterday, the streak increments. If it was today, nothing changes. If it was more than a day ago, the streak resets to 1.

### Quiz Engine

The `QuizScreen` acts as a controller that receives a list of questions from the API. Based on each question's `question_type` field, it renders the appropriate sub-screen (`MultipleChoiceScreen`, `ListenChooseScreen`, etc.). Answers are collected locally and submitted in one batch at the end for scoring.

---

## Admin Dashboard

The web admin dashboard at `admin.ijawapp.com` allows content managers to:

- Create and publish levels and lessons
- Add new Ijaw words with English translations, phonetics, and native audio files
- Build quiz questions visually with a drag-and-drop interface
- View user statistics and learning progress
- Manage user accounts

```mermaid
flowchart TD
    A[Admin Login] --> B[Dashboard - Stats Overview]
    B --> C[Levels Manager]
    B --> D[Lessons Manager]
    B --> E[Word + Audio Manager]
    B --> F[Quiz Builder]
    B --> G[User Management]

    C --> H[Create / Edit / Publish Level]
    D --> I[Create / Edit / Reorder Lessons]
    E --> J[Add Word + Upload .mp3]
    J --> K[S3 Upload + CDN URL saved to DB]
    F --> L[Add Multiple Choice / Match / Fill-in-Blank Questions]
    G --> M[View Progress / Reset Passwords / Suspend Accounts]
```

---

## Setup & Installation

### Prerequisites

- Node.js 18+
- PostgreSQL 15+
- Redis
- Expo CLI (`npm install -g expo-cli`)
- AWS account or Cloudflare R2 bucket

### Mobile App

```bash
cd mobile
npm install
cp .env.example .env
# Fill in API_URL in .env
npx expo start
```

### Backend

```bash
cd backend
npm install
cp .env.example .env
# Fill in DATABASE_URL, JWT_SECRET, S3 credentials
npx prisma migrate dev
npx prisma db seed
npm run dev
```

### Admin Dashboard

```bash
cd admin
npm install
npm run dev
```

---

## Environment Variables

### Backend `.env`

```
DATABASE_URL=postgresql://user:password@localhost:5432/ijaw_app
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_jwt_secret
JWT_REFRESH_SECRET=your_refresh_secret
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=30d

AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_BUCKET_NAME=ijaw-audio

SMTP_HOST=
SMTP_USER=
SMTP_PASS=

PORT=4000
```

### Mobile `.env`

```
EXPO_PUBLIC_API_URL=https://api.ijawapp.com
```

---

## Deployment

```mermaid
flowchart LR
    subgraph CI["CI/CD (GitHub Actions)"]
        Push[Git Push] --> Test[Run Tests]
        Test --> Build[Build Docker Image]
        Build --> Deploy[Deploy to VPS / Railway]
    end

    subgraph Production
        API[Backend API\nRailway / VPS]
        AdminUI[Admin Dashboard\nVercel]
        CDN[Audio Files\nCloudflare R2 + CDN]
        DB[(PostgreSQL\nNeon / Supabase)]
        Cache[(Redis\nUpstash)]
    end

    subgraph Stores
        Play[Google Play Store]
        AppStore[Apple App Store]
    end

    Deploy --> API
    API --> DB
    API --> Cache
    API --> CDN

    MobileApp[Expo EAS Build] --> Play
    MobileApp --> AppStore
```

### App Store Deployment

Use Expo EAS (Expo Application Services):

```bash
# Install EAS CLI
npm install -g eas-cli

# Configure builds
eas build:configure

# Build for both platforms
eas build --platform all

# Submit to stores
eas submit --platform android
eas submit --platform ios
```
