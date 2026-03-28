# MyHealthFirstAI - System Architecture

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MyHealthFirstAI                                    │
│                    "AI-Powered Fitness & Nutrition"                         │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────┐
                              │   User Device   │
                              └────────┬────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                  │
                    ▼                  ▼                  ▼
            ┌───────────┐      ┌───────────┐      ┌───────────┐
            │    iOS    │      │  Android  │      │    Web    │
            │   (Expo)  │      │  (Expo)   │      │ (RN Web)  │
            └─────┬─────┘      └─────┬─────┘      └─────┬─────┘
                  │                  │                  │
                  └──────────────────┼──────────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │     React Native + Expo         │
                    │     (Single Codebase)           │
                    │                                 │
                    │  • 14 Screens                   │
                    │  • Expo Router Navigation       │
                    │  • Platform-adaptive UI         │
                    │  • Local SQLite Storage         │
                    └────────────────┬────────────────┘
                                     │
                                     │ HTTP/REST
                                     │
                    ┌────────────────┴────────────────┐
                    │         FastAPI Backend         │
                    │         (Python 3.11+)          │
                    │                                 │
                    │  • 9 API Routers                │
                    │  • Async SQLAlchemy             │
                    │  • Rate Limiting                │
                    │  • JWT Authentication           │
                    └────────────────┬────────────────┘
                                     │
            ┌────────────────────────┼────────────────────────┐
            │                        │                        │
            ▼                        ▼                        ▼
    ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
    │   SQLite DB   │       │  Google       │       │   Fallback    │
    │  (Async)      │       │  Gemini AI    │       │   Database    │
    │               │       │               │       │               │
    │  • Users      │       │  • Flash 2.0  │       │  • 30+ Foods  │
    │  • Food Logs  │       │  • Pro 1.5    │       │  • Offline    │
    │  • Health     │       │               │       │  • No Quota   │
    │  • Badges     │       │  Vision/Text  │       │               │
    │  • Streaks    │       │  /Audio AI    │       │               │
    └───────────────┘       └───────────────┘       └───────────────┘
```

---

## 📱 Frontend Architecture

### Expo Router - File-Based Navigation

```
app/
├── _layout.tsx          # Root layout with platform detection
├── index.tsx            # Dashboard (home)
├── food.tsx             # AI Food Scanner
├── voice.tsx            # Voice Logging
├── water.tsx            # Water Tracking
├── recipes.tsx          # Pantry Chef
├── planner.tsx          # Meal Planner
├── workout.tsx          # AI Workout Planner (800+ lines)
├── watch.tsx            # Smartwatch Sync (800+ lines)
├── form.tsx             # Form Corrector
├── coach.tsx            # AI Coach Chat
├── badges.tsx           # Achievements
├── premium.tsx          # Subscription
└── more.tsx             # Additional Options
```

### Responsive Navigation Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        RESPONSIVE NAVIGATION                                 │
└─────────────────────────────────────────────────────────────────────────────┘

MOBILE (iOS/Android)                    WEB (Desktop/Tablet)
┌──────────────────────┐                ┌──────────────────────────────────────┐
│                      │                │┌────────────┐                        │
│      Content         │                ││            │       Content          │
│      Area            │                ││  Sidebar   │       Area             │
│                      │                ││            │                        │
│                      │                ││  • Logo    │                        │
│                      │                ││  • Nav     │                        │
├──────────────────────┤                ││  • Streak  │                        │
│ 🏠 │ 🍎 │ 🤖 │ 💎 │ ☰ │                ││  • Pro     │                        │
│Home│Food│ AI │Pro│More│                │└────────────┘                        │
└──────────────────────┘                └──────────────────────────────────────┘
     Bottom Tabs (5)                         Left Sidebar (12 items)
```

### Component Architecture

```
components/
├── shared/                    # Cross-platform components
│   ├── GlassCard.tsx          # Glass morphism container
│   ├── ActionButton.tsx       # Primary/secondary buttons
│   ├── RingChart.tsx          # Animated circular progress
│   ├── Gamification.tsx       # Streak, XP, badges display
│   ├── UpgradeModal.tsx       # Pro upgrade prompt
│   └── index.ts               # Barrel export
│
└── web/
    ├── Sidebar.tsx            # Web navigation sidebar
    └── index.ts
```

### Context Providers

```typescript
// App wrapped in providers
<SubscriptionProvider>      // Freemium state, usage limits
  <UserProvider>            // User data, progress, streaks
    <App />
  </UserProvider>
</SubscriptionProvider>
```

#### SubscriptionContext
- `tier`: 'free' | 'pro'
- `isPro`: boolean
- `canUseVoiceLogging`: boolean
- `canUseFormCorrector`: boolean
- `dailyScansRemaining`: number
- `subscribe()`, `cancelSubscription()`

#### UserContext
- `profile`: User profile data
- `todayProgress`: Daily calories/macros
- `currentStreak`: Login streak count
- `badges`: Unlocked achievements
- `logFood()`, `logWater()`, `syncHealth()`

---

## ⚙️ Backend Architecture

### FastAPI Application Structure

```python
# main.py - Entry Point
app = FastAPI(
    title="MyHealthFirstAI API",
    version="1.0.0",
    lifespan=lifespan  # Async startup/shutdown
)

# Middleware Stack
app.add_middleware(CORSMiddleware, ...)
app.add_middleware(RateLimitMiddleware)

# Router Registration (9 routers)
app.include_router(auth.router,         prefix="/api/auth")
app.include_router(food.router,         prefix="/api/food")
app.include_router(voice.router,        prefix="/api/voice")
app.include_router(form.router,         prefix="/api/form")
app.include_router(recipes.router,      prefix="/api/recipes")
app.include_router(chat.router,         prefix="/api/chat")
app.include_router(workout.router,      prefix="/api/workout")
app.include_router(health.router,       prefix="/api/health")
app.include_router(subscription.router, prefix="/api/subscription")
```

### Router Responsibilities

| Router | Endpoints | Description |
|--------|-----------|-------------|
| `auth.py` | `/login`, `/register`, `/me` | JWT authentication |
| `food.py` | `/analyze`, `/diet-adjustment` | Food image AI analysis |
| `voice.py` | `/analyze`, `/parse-text`, `/transcribe` | Voice-to-food processing |
| `form.py` | `/analyze` | Exercise form video analysis |
| `recipes.py` | `/generate` | AI recipe generation |
| `chat.py` | `/message` | AI coach conversations |
| `workout.py` | `/generate-plan` | Personalized workout plans |
| `health.py` | `/sync`, `/today`, `/history`, `/summary` | Smartwatch data |
| `subscription.py` | `/check-limit`, `/check-badges` | Freemium management |

### Service Layer

```
services/
├── gemini_ai.py           # Google Gemini integration
│   ├── analyze_food_image()
│   ├── parse_food_from_text()
│   ├── fallback_parse_food()     # Offline fallback
│   ├── generate_recipe()
│   ├── analyze_form()
│   └── chat_with_coach()
│
├── vision_ai.py           # Multi-modal vision service
│   └── MultiModalVisionService
│
├── voice_processor.py     # Audio processing
│   ├── transcribe_audio()
│   └── voice_to_food_pipeline()
│
└── rate_limiter.py        # Usage tracking
    ├── check_daily_limit()
    └── increment_usage()
```

### Database Models (SQLAlchemy Async)

```python
# Core Models
class User(Base):
    id, email, password_hash, subscription_tier
    created_at, updated_at

class FoodLog(Base):
    id, user_id, food_name, calories, protein, carbs, fat
    meal_type, source, created_at

class HealthData(Base):
    id, user_id, sync_id, data_type
    steps, heart_rate, spo2, sleep_hours
    source, recorded_at, synced_at

class UserBadge(Base):
    id, user_id, badge_id, unlocked_at

class Streak(Base):
    id, user_id, streak_type, current_count
    longest_count, last_activity

class DailyUsage(Base):
    id, user_id, date, feature_type, count
```

---

## 🔄 Data Flow Examples

### Food Scanning Flow

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Camera  │───▶│ Base64  │───▶│ Backend │───▶│ Gemini  │
│ Capture │    │ Encode  │    │ /food/  │    │ Vision  │
└─────────┘    └─────────┘    │ analyze │    │ AI      │
                              └────┬────┘    └────┬────┘
                                   │              │
                              ┌────▼────┐    ┌────▼────┐
                              │ Rate    │    │ Parse   │
                              │ Limit   │    │ JSON    │
                              │ Check   │    │ Result  │
                              └────┬────┘    └────┬────┘
                                   │              │
                              ┌────▼──────────────▼────┐
                              │    Return Nutrition     │
                              │    {calories, protein,  │
                              │     carbs, fat, name}   │
                              └─────────────────────────┘
```

### Voice Logging Flow

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Record  │───▶│ Base64  │───▶│ Backend │───▶│ Gemini  │
│ Audio   │    │ Encode  │    │ /voice/ │    │ Audio   │
└─────────┘    └─────────┘    │ analyze │    │ Trans.  │
                              └────┬────┘    └────┬────┘
                                   │              │
                                   │         ┌────▼────┐
                                   │         │ Parse   │
                                   │         │ Food    │
                                   │         │ Text    │
                                   │         └────┬────┘
                                   │              │
                    ┌──────────────┴──────────────┘
                    │
              ┌─────▼─────┐
              │  Gemini   │ ◄──── If quota exceeded
              │  Available│       ────────────────▶
              └─────┬─────┘                       │
                    │                        ┌────▼────┐
              ┌─────▼─────┐                  │Fallback │
              │ AI Parse  │                  │Food DB  │
              │ Nutrition │                  │(30+)    │
              └───────────┘                  └─────────┘
```

### Workout Plan Generation Flow

```
User Input                    Processing                    Output
┌──────────────────┐    ┌──────────────────────┐    ┌──────────────────┐
│ • Height/Weight  │    │ Calculate BMI        │    │ Weekly Plan:     │
│ • Age/Gender     │───▶│ Determine calories   │───▶│ • Day 1: Upper   │
│ • Goal           │    │ Adjust for goal      │    │ • Day 2: Lower   │
│ • Target Weight  │    │ Select exercises     │    │ • Day 3: Core    │
│ • Timeframe      │    │ Create diet plan     │    │ • Day 4: Full    │
│ • Days/Week      │    │ Calculate progress   │    │ • Diet Plan      │
│ • Experience     │    │                      │    │ • Tips           │
└──────────────────┘    └──────────────────────┘    └──────────────────┘
```

---

## 🔐 Security Architecture

### Authentication Flow

```
┌─────────┐         ┌─────────┐         ┌─────────┐
│ Client  │         │ Backend │         │   DB    │
└────┬────┘         └────┬────┘         └────┬────┘
     │                   │                   │
     │  POST /login      │                   │
     │  {email, pass}    │                   │
     │──────────────────▶│                   │
     │                   │  Verify user      │
     │                   │──────────────────▶│
     │                   │◀──────────────────│
     │                   │                   │
     │                   │  Generate JWT     │
     │                   │  (24h expiry)     │
     │   {token}         │                   │
     │◀──────────────────│                   │
     │                   │                   │
     │  GET /api/food    │                   │
     │  Auth: Bearer JWT │                   │
     │──────────────────▶│                   │
     │                   │  Validate JWT     │
     │                   │  Extract user_id  │
     │   {data}          │                   │
     │◀──────────────────│                   │
```

### Rate Limiting Strategy

```python
# Limits by subscription tier
LIMITS = {
    'free': {
        'food_scans': 3,      # per day
        'voice_logs': 0,      # Pro only
        'form_checks': 0,     # Pro only
        'recipes': 2,         # per day
        'chat_messages': 10   # per day
    },
    'pro': {
        'food_scans': -1,     # unlimited
        'voice_logs': -1,
        'form_checks': -1,
        'recipes': -1,
        'chat_messages': -1
    }
}
```

---

## 📊 AI Model Usage

### Google Gemini Integration

| Model | Use Case | Speed | Cost |
|-------|----------|-------|------|
| `gemini-2.0-flash` | Food recognition, chat, voice | Fast | Low |
| `gemini-1.5-pro` | Recipes, form analysis | Slower | Higher |

### Fallback Strategy

When Gemini quota is exceeded:

```python
COMMON_FOODS = {
    "pizza": {"calories": 285, "protein": 12, "carbs": 36, "fat": 10},
    "burger": {"calories": 540, "protein": 25, "carbs": 40, "fat": 29},
    "salad": {"calories": 150, "protein": 5, "carbs": 12, "fat": 10},
    # ... 30+ common foods
}

def fallback_parse_food(text: str):
    # Keyword matching against local database
    for food, data in COMMON_FOODS.items():
        if food in text.lower():
            return FoodAnalysisResult(**data)
```

---

## 📦 Deployment Architecture

### Development

```
┌─────────────────────────────────────────────────────────────┐
│                    Local Development                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Frontend (Expo)              Backend (FastAPI)             │
│  ┌─────────────────┐          ┌─────────────────┐          │
│  │ localhost:19006 │◀────────▶│ localhost:8000  │          │
│  │ (Metro Bundler) │          │ (Uvicorn)       │          │
│  └─────────────────┘          └────────┬────────┘          │
│                                        │                    │
│                               ┌────────▼────────┐          │
│                               │ SQLite (file)   │          │
│                               │ myhealthfirstai │          │
│                               │ .db             │          │
│                               └─────────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### Production (Recommended)

```
┌─────────────────────────────────────────────────────────────┐
│                    Production Setup                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────┐    ┌───────────────┐    ┌────────────┐  │
│  │  CDN/Vercel   │    │   Cloud Run   │    │ PostgreSQL │  │
│  │  (Frontend)   │───▶│   (Backend)   │───▶│  (Supabase)│  │
│  │               │    │               │    │            │  │
│  └───────────────┘    └───────┬───────┘    └────────────┘  │
│                               │                             │
│                      ┌────────▼────────┐                   │
│                      │  Google Cloud   │                   │
│                      │  Gemini API     │                   │
│                      └─────────────────┘                   │
│                                                             │
│  Mobile Apps:                                               │
│  ┌───────────────┐    ┌───────────────┐                    │
│  │  App Store    │    │  Play Store   │                    │
│  │  (iOS)        │    │  (Android)    │                    │
│  └───────────────┘    └───────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧪 Testing Strategy

### Frontend
- Component testing with Jest
- E2E testing with Detox (mobile)
- Playwright for web

### Backend
- pytest for unit tests
- pytest-asyncio for async tests
- httpx for API testing

```bash
# Run backend tests
cd backend
pytest tests/ -v

# Run frontend tests
cd frontend
npm test
```

---

## 📈 Performance Considerations

### Frontend
- **Lazy loading**: Screens loaded on demand
- **Image optimization**: Compressed before upload
- **Memoization**: React.memo for expensive components
- **Local storage**: SQLite for offline-first experience

### Backend
- **Async everywhere**: asyncio, aiosqlite
- **Connection pooling**: SQLAlchemy async engine
- **Response caching**: For repeated queries
- **Rate limiting**: Prevent abuse

---

## 🔮 Future Architecture Considerations

1. **Microservices**: Split AI services into separate containers
2. **Message Queue**: Redis/RabbitMQ for async AI processing
3. **Real-time**: WebSocket for live coaching
4. **CDN**: CloudFlare for global edge caching
5. **Analytics**: PostHog or Mixpanel integration

---

<p align="center">
  <b>Architecture designed for scale and maintainability</b>
</p>
