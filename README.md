# AuraFit AI — Student Workout & Diet Planner

AuraFit AI is a full-stack web app that generates personalized, budget-friendly workout and diet plans for students. Users complete a short onboarding form, and the app produces a 7-day diet plan, a 7-day workout split, a categorized grocery list, a weekly budget summary, and an AI coach chatbot — powered by the Grok API, with a smart offline fallback when no API key is configured.

## Features

- **Onboarding wizard** — collects age, gender, weight, height, fitness goal, activity level, diet preference, budget, available equipment, and medical conditions/injuries.
- **AI-generated plan** — calls the Grok API (`grok-2-1212`) to produce a full week of meals and workouts tailored to the user's profile, culture, and budget.
- **Offline fallback generator** — if no Grok API key is set, the server calculates BMR/TDEE (Mifflin-St Jeor) and macro targets itself, then assembles a plan from built-in exercise and meal libraries — no internet/API dependency required to demo the app.
- **Dashboard** with six tabs:
  - **Overview** — quick summary and macro targets
  - **Workouts** — 7-day exercise split with sets, reps, rest, and form cues
  - **Meals & Diet** — daily meals with ingredients, macros, and cost
  - **Grocery List** — items grouped by category with estimated cost and shelf life
  - **AI Coach Chat** — conversational assistant for swaps, injuries, and budget tips
  - **Progress Tracker** — weight and water-intake logging
- **Persistent local state** — profile and plan are saved to `localStorage` so users don't have to regenerate on every visit; a "Reset" action clears everything.

## Tech Stack

| Layer | Stack |
|---|---|
| Client | React 19 + Vite, `lucide-react` icons |
| Server | Node.js (ESM) + Express |
| AI | x.ai Grok API (`grok-2-1212`), with offline fallback logic |
| Storage | Browser `localStorage` (no database) |

## Project Structure

```
Diet/
├── client/                 # React + Vite frontend
│   └── src/
│       ├── components/
│       │   ├── Onboarding.jsx       # Multi-step profile form
│       │   ├── Dashboard.jsx        # Tabbed app shell
│       │   ├── WorkoutPlan.jsx
│       │   ├── DietPlan.jsx
│       │   ├── GroceryList.jsx
│       │   ├── AICoach.jsx
│       │   └── ProgressTracker.jsx
│       └── utils/macroCalculator.js
├── server/                 # Express backend
│   ├── server.js           # API routes
│   ├── services/aiService.js  # Grok API calls + offline fallback generator
│   └── .env.example
└── package.json            # Root scripts to run client + server together
```

## Prerequisites

- Node.js 18+ and npm
- A [Grok (x.ai) API key](https://x.ai/) — optional. Without it, the app still works using the offline plan generator.

## Setup

1. **Install dependencies** (root, client, and server):
   ```bash
   npm run install-all
   ```
   This runs `install-client`, `install-server`, and the root install in sequence. (Each uses `--legacy-peer-deps`.)

2. **Configure environment variables**

   Copy the example file and add your key:
   ```bash
   cp server/.env.example server/.env
   ```
   ```env
   PORT=5000
   GROK_API_KEY=YOUR_GROK_API_KEY_HERE
   ```
   Leave `GROK_API_KEY` as the placeholder (or empty) to use the offline fallback generator instead of calling the live API.

3. **Run the app** (client + server concurrently):
   ```bash
   npm run dev
   ```
   - Backend: `http://localhost:5000`
   - Frontend: `http://localhost:5173` (default Vite port)

   Or run them separately:
   ```bash
   npm run server   # backend only, with nodemon
   npm run client   # frontend only, with vite
   ```

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Health check |
| `POST` | `/api/generate-plan` | Generates a workout/diet plan from a submitted profile |
| `POST` | `/api/chat` | AI Coach chat — expects `{ messages: [], userProfile: {} }` |

### `POST /api/generate-plan` request body

```json
{
  "age": "20",
  "gender": "male",
  "weight": "70",
  "height": "175",
  "goal": "loss | gain | strength | general",
  "activityLevel": "sedentary | light | moderate | active",
  "dietPreference": "none | vegetarian | vegan | halal | ...",
  "budget": "low | medium | high",
  "equipment": "none | dumbbells | gym",
  "medicalConditions": "optional free text"
}
```

The response includes `macros`, `dietPlan` (7 days of meals), `workoutPlan` (7 days of exercises), `groceryList`, and `budgetSummary`. An `offline: true/false` flag indicates whether the plan came from the live Grok API or the local fallback generator.

## Notes

- This is a client-side-only persistence app — there is no database; all user data lives in the browser's `localStorage` and is cleared on reset.
- CORS is enabled on the server for local frontend development.
- The `GROK_API_KEY` is read server-side only and never exposed to the client.
