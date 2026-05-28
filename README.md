# ZENTRA — AI Travel Optimization Platform 

ZENTRA is a full-stack travel planning web app that finds every possible route between any two cities — direct flights, direct trains, connecting flights via hubs, connecting trains, and mixed flight+train options. It compares them by cost and time, optionally plans your hotel stay and food budget, and generates AI-powered travel insights.

---

## What It Does

- **Route Search** — Direct flights, direct trains, connecting flights via 6 major hubs, connecting trains, mixed flight+train combos
- **Smart Filtering** — Sort all results by cheapest, fastest, or best balanced score
- **Live Data** — Real flights from AviationStack, real trains from IRCTC via RapidAPI — falls back to accurate Haversine distance estimates when APIs are unavailable or quota is exhausted
- **Hotel & Food Planning** — Enter a budget and get hotel recommendations with per-night cost and a daily food breakdown
- **AI Insights** — Gemini-powered travel tips, best season to visit, and must-do highlights
- **Interactive Map** — Animated ant-path routes drawn on a dark CartoDB map, colour-coded by transport mode
- **User Accounts** — Register, log in, and save trips (JWT authentication)
- **Smart Caching** — Redis cache with automatic in-memory fallback so repeated searches are instant and don't burn API quota

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, Vite, Tailwind CSS, Framer Motion |
| Map | React Leaflet + leaflet-ant-path |
| Charts | Recharts (cost vs time scatter plot) |
| Backend | Node.js 18+, Express 4 |
| Database | MongoDB Atlas (cloud, free tier) |
| Cache | Redis (optional) with in-memory fallback |
| Auth | JWT + bcryptjs |
| AI | Google Gemini (`gemini-1.5-flash`) |
| Flights | AviationStack API |
| Trains | IRCTC via RapidAPI (`irctc1.p.rapidapi.com`) |
| Hotels | Booking.com via RapidAPI (`booking-com-api3.p.rapidapi.com`) |
| Routing | Dijkstra algorithm with MinHeap priority queue |

---

## Prerequisites

Install these before you begin:

| Tool | Version | Download |
|------|---------|----------|
| Node.js | v18 or higher | https://nodejs.org |
| npm | v9 or higher (bundled with Node) | — |
| Git | any | https://git-scm.com |

Verify they are installed:
```bash
node --version    # v18.x.x or higher
npm --version     # 9.x.x or higher
git --version
```

---

## Step 1 — Get Your API Keys

You need **4 free API keys**. Get them before starting setup.

### AviationStack — Live flight data
1. Go to https://aviationstack.com and click **Sign Up Free**
2. After signing up, your **API Access Key** is on the dashboard
3. Free plan: 100 requests/month (app uses distance estimates after this)

### RapidAPI — IRCTC trains + Booking.com hotels
Both services use the **same RapidAPI key**:
1. Create a free account at https://rapidapi.com
2. Search **"IRCTC"** → find `irctc1.p.rapidapi.com` → click **Subscribe to Test**
3. Search **"Booking.com"** → find `booking-com-api3.p.rapidapi.com` → click **Subscribe to Test**
4. Your **RapidAPI Key** appears in any API's Code Snippet panel under `"X-RapidAPI-Key"`

### Google Gemini — AI travel insights
1. Go to https://aistudio.google.com/app/apikey
2. Click **Create API Key** → choose any Google Cloud project (or create one)
3. Copy the key — free tier has generous limits for personal use

---

## Step 2 — Set Up MongoDB Atlas

1. Go to https://www.mongodb.com/atlas and create a **free account**
2. Create a new **free M0 cluster** (pick any region)
3. **Database Access** → Add new database user:
   - Username: anything (e.g. `zentra_user`)
   - Password: something secure — **save this, you'll need it**
   - Role: **Read and write to any database**
4. **Network Access** → Add IP Address → click **Allow Access From Anywhere** (0.0.0.0/0)
   - For production, restrict this to your server IP
5. **Clusters** → Connect → Drivers → Node.js → copy the connection string:
   ```
   mongodb+srv://zentra_user:<password>@cluster0.xxxxx.mongodb.net/?appName=Cluster0
   ```
6. Replace `<password>` with your actual password and add `/zentra` as the database name:
   ```
   mongodb+srv://zentra_user:yourpassword@cluster0.xxxxx.mongodb.net/zentra?retryWrites=true&w=majority&appName=Cluster0
   ```
   This is your `MONGODB_URI`.

---

## Step 3 — Clone and Install

```bash
git clone https://github.com/YOUR_USERNAME/zentra.git
cd zentra
```

Install backend dependencies:
```bash
cd server
npm install
```

Install frontend dependencies:
```bash
cd ../client
npm install
```

---

## Step 4 — Create Environment Files

### Backend — `server/.env`

Create a file called `.env` inside the `server/` folder:

```bash
# Windows (PowerShell)
cd server
New-Item .env

# Mac / Linux
cd server
touch .env
```

Open it in any text editor and paste this, replacing the placeholder values:

```env
# MongoDB Atlas connection string (from Step 2)
MONGODB_URI=mongodb+srv://YOUR_USERNAME:YOUR_PASSWORD@cluster0.xxxxx.mongodb.net/zentra?retryWrites=true&w=majority&appName=Cluster0

# AviationStack (from Step 1)
AVIATIONSTACK_API_KEY=your_aviationstack_key_here

# RapidAPI — used for both IRCTC trains and Booking.com hotels (from Step 1)
RAPIDAPI_KEY=your_rapidapi_key_here

# Google Gemini (from Step 1)
GEMINI_API_KEY=your_gemini_key_here

# JWT secret — any long random string, keep it private
JWT_SECRET=some_very_long_random_string_change_in_production

# Redis — if you set up Redis (Step 6), keep this. Otherwise the app uses memory cache.
REDIS_URL=redis://127.0.0.1:6379

# Server port
PORT=3001
```

### Frontend — `client/.env`

Create a file called `.env` inside the `client/` folder:

```env
VITE_API_URL=http://localhost:3001
```

---

## Step 5 — Seed the Database

This loads 80+ Indian cities (with coordinates, IATA codes, and railway station codes) into MongoDB:

```bash
cd server
npm run seed
```

Expected output:
```
Connected to MongoDB
Seeded 79 cities
```

You only need to run this once. If you change nothing, you don't need to run it again.

---

## Step 6 — Start the App

Open **two terminals** side by side.

**Terminal 1 — Backend:**
```bash
cd server
npm run dev
```

Expected output:
```
🚀 ZENTRA server on port 3001
✅ MongoDB connected
⚠️  Redis unavailable — running without cache   ← normal if Redis not set up
```

**Terminal 2 — Frontend:**
```bash
cd client
npm run dev
```

Expected output:
```
  VITE v5.x.x  ready in ~300ms
  ➜  Local:   http://localhost:5173/
```

Open **http://localhost:5173** in your browser. You're done.

---

## Step 7 — Redis Setup (Optional)

Redis caches API responses so repeated searches are instant and don't use quota. The app works without it — it uses an in-memory cache automatically. Set up Redis if you want the cache to survive server restarts.

### Windows (using WSL)

1. Open PowerShell as Administrator and install WSL if you haven't:
   ```powershell
   wsl --install
   ```
   Restart your computer when prompted.

2. Open the **Ubuntu** app from the Start Menu, then run:
   ```bash
   sudo apt-get update
   sudo apt-get install -y redis-server
   redis-server --daemonize yes
   redis-cli ping
   ```
   You should see `PONG`. Redis is now running.

3. To start Redis after a reboot, open Ubuntu and run:
   ```bash
   redis-server --daemonize yes
   ```

### Mac

```bash
brew install redis
brew services start redis
redis-cli ping    # should print PONG
```

### Linux (Ubuntu/Debian)

```bash
sudo apt-get install -y redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
redis-cli ping    # should print PONG
```

Once Redis is running, restart the backend server. You'll see `✅ Redis connected` instead of the warning.

---

## How Route Search Works

```
User enters: Delhi → Vijayawada  (modes: flight, train)

Backend:
  1. Resolve cities → IATA codes (DEL, VGA) + coordinates
  
  2. Fetch in parallel:
       AviationStack  → live DEL→VGA flights  (or distance estimate)
       IRCTC RapidAPI → NDLS→BZA trains       (or distance estimate)
  
  3. Enumerate connecting routes through 6 hub airports:
       flight→flight  DEL→HYD + HYD→VGA  (via Hyderabad)
       flight→flight  DEL→BOM + BOM→VGA  (via Mumbai)
       flight→flight  DEL→BLR + BLR→VGA  (via Bangalore)
       ...and via Chennai, Kolkata
       train→train    Delhi→Hyderabad + Hyderabad→Vijayawada
       flight→train   DEL→HYD flight + HYD→VGA train
       ...etc.
  
  4. Run Dijkstra on the edge graph → optimal paths
  
  5. Guarantee one result per transport mode (so train always appears
     alongside flight even when flight is cheaper on all metrics)
  
  6. Sort by balanced score (60% cost + 40% time)
  
  7. Return all paths → frontend shows them all with mode badges
```

---

## API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/search` | No | Search routes + hotel plan |
| GET | `/api/cities/autocomplete?q=del` | No | City name suggestions |
| POST | `/api/auth/register` | No | Create account |
| POST | `/api/auth/login` | No | Login, returns JWT |
| GET | `/api/trips` | Yes | List saved trips |
| POST | `/api/trips` | Yes | Save a trip |
| DELETE | `/api/trips/:id` | Yes | Delete a saved trip |
| GET | `/api/ai/insights` | No | AI travel tips |

### Search request body

```json
{
  "from": "Delhi",
  "to": "Vijayawada",
  "modes": ["flight", "train", "bus", "ship"],
  "budget": 15000,
  "nights": 3,
  "checkIn": "2025-06-01",
  "checkOut": "2025-06-04"
}
```

`budget`, `nights`, `checkIn`, `checkOut` are optional — omit them to skip hotel planning.

---

## Transport Mode Colours

| Mode | Map Colour | Badge Colour |
|------|-----------|-------------|
| ✈️ Flight | Blue | Blue |
| 🚆 Train | Green | Green |
| 🚌 Bus | Orange | Orange |
| 🚢 Ship | Cyan | Cyan |
| 🚕 Cab | Yellow | Yellow |
| 🚇 Metro | Pink | Pink |

---

## Project Structure

```
ZENTRA/
├── client/                          Frontend (React + Vite)
│   ├── src/
│   │   ├── App.jsx                  Root: 3-panel layout + state
│   │   ├── context/AuthContext.jsx  Login state, JWT storage
│   │   └── components/
│   │       ├── layout/
│   │       │   ├── LeftPanel.jsx    Search form, mode toggles, budget
│   │       │   └── RightPanel.jsx   Route cards, filters, tabs
│   │       ├── map/CenterMap.jsx    Leaflet map + animated paths
│   │       ├── AuthModal.jsx        Login / register popup
│   │       ├── CityAutocomplete.jsx Debounced city search input
│   │       ├── TravelTimeline.jsx   Vertical journey timeline
│   │       ├── HotelPlan.jsx        Budget breakdown + hotel cards
│   │       ├── AiInsights.jsx       Gemini AI card
│   │       └── charts/RouteGraph.jsx Cost vs time chart
│   ├── .env                         VITE_API_URL=http://localhost:3001
│   └── package.json
│
└── server/                          Backend (Node.js + Express)
    ├── src/
    │   ├── app.js                   Express setup + rate limiting
    │   ├── controllers/
    │   │   ├── search.controller.js City resolution + route search
    │   │   ├── auth.controller.js   Register / login
    │   │   ├── trip.controller.js   Save / fetch trips
    │   │   ├── ai.controller.js     AI insights
    │   │   └── city.controller.js   Autocomplete
    │   ├── services/
    │   │   ├── path.service.js      Builds ALL route options (core logic)
    │   │   ├── dijkstra.service.js  MinHeap Dijkstra pathfinding
    │   │   ├── aviation.service.js  AviationStack + flight estimates
    │   │   ├── irctc.service.js     IRCTC RapidAPI + train estimates
    │   │   ├── flightNormalizer.service.js
    │   │   ├── hotel.service.js     Budget planner
    │   │   ├── bookingcom.service.js Booking.com RapidAPI
    │   │   ├── ai.service.js        Google Gemini
    │   │   ├── cache.service.js     Redis + memory fallback cache
    │   │   └── redis.service.js     ioredis connection
    │   ├── models/
    │   │   ├── City.model.js
    │   │   ├── User.model.js
    │   │   └── SavedTrip.model.js
    │   ├── middleware/auth.middleware.js  JWT verification
    │   └── scripts/seed.js          Seeds 79 cities into MongoDB
    ├── .env                         All API keys (never commit this)
    └── package.json
```

---

## Supported Cities

The app has built-in coordinate and station data for **80+ Indian cities** so routes work even without a live database.

**With airports (flights available):**
Delhi, Mumbai, Bangalore, Hyderabad, Chennai, Kolkata, Pune, Ahmedabad, Kochi, Goa, Jaipur, Lucknow, Varanasi, Amritsar, Guwahati, Nagpur, Thiruvananthapuram, Coimbatore, Madurai, Visakhapatnam, Vijayawada, Patna, Chandigarh, Indore, Bhopal, Srinagar, Leh, Agra, Mysore, Udaipur, Jodhpur, Ranchi, Raipur, Aurangabad, Trichy, Bhubaneswar, and more.

**International:**
Dubai, Doha, London, Singapore, Bangkok, Paris, New York, Tokyo, Kuala Lumpur, Sydney.

**Train-only (no airport):**
Guntur, Haridwar, Rishikesh, Puri, Darjeeling, Jamshedpur, Solapur, Ajmer, Nashik, Kolhapur, Dhanbad, and more.

---

## Troubleshooting

### "0 routes found" for a city pair
- Check server terminal — look for "no station code" or "no IATA" warnings
- Try spelling variants: "Bangalore" or "Bengaluru", "Mumbai" or "Bombay"
- The app has estimates for 80+ cities — if your city isn't listed, try the nearest major city

### MongoDB connection fails on startup
- Check your `MONGODB_URI` — password should not contain `@` or special URL characters (encode them if so)
- In Atlas → **Network Access**, make sure your IP or 0.0.0.0/0 is whitelisted
- Re-run `npm run seed` after fixing the connection

### "IRCTC 403" in server logs
- You need to subscribe to the IRCTC API on RapidAPI (free)
- Go to https://rapidapi.com → search `irctc1` → Subscribe to Test
- Train distance estimates are used automatically until then

### "AviationStack: 0 live flights"
- AviationStack's free plan only returns currently airborne flights
- The app automatically uses distance-based estimates — routes still appear

### Redis shows "unavailable" in logs
- This is not an error — in-memory cache takes over automatically
- To set up Redis, follow Step 7 above

### Frontend can't reach the backend (network error)
- Make sure the server is running (`npm run dev` in `server/`)
- Check `client/.env` has `VITE_API_URL=http://localhost:3001` (no trailing slash)
- If you changed the server port, update both `server/.env` and `client/.env`

### Port 3001 already in use
```powershell
# Windows PowerShell
Get-Process -Id (Get-NetTCPConnection -LocalPort 3001 -ErrorAction SilentlyContinue).OwningProcess | Stop-Process -Force
```
```bash
# Mac / Linux
kill -9 $(lsof -ti:3001)
```

---

## License

Private
