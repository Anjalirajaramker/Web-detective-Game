# WebDetective - Web Inspector Challenge Game

A multiplayer real-time web detective game where participants solve challenges by inspecting webpage source code, finding hidden elements, and answering web development trivia. Built for Round-1 of our our event for QUEST-2026 which can  supported  upto 100+ concurrent participations.

## 🎮 Features

- **5 Domain-Specific Challenges**
  - GitHub Repository Hunt (7 questions)
  - Documentation Deep Dive (6 questions)
  - Wikipedia Fact Finding (6 questions)
  - Neal.fun Visualization Explorer (6 questions)
  - HTML/CSS/JavaScript Inspection (6 questions)

- **Real-Time Multiplayer**
  - UUID-based session management
  - Live leaderboard and stats
  - Concurrent participant support (100+)
  - Attempt tracking and audit logs

- **Student Verification**
  - Google Sheets integration with XLSX parsing
  - Multi-field authentication (name, team, phone)
  - Duplicate attempt prevention

- **Admin Dashboard**
  - Real-time results and statistics
  - CSV export functionality
  - Question bank management (add/update/deactivate)
  - Runtime question updates without redeployment

- **Smart Question Distribution**
  - Weighted randomization algorithm
  - Prevents bias toward frequently-used challenges
  - Fair difficulty distribution across 100+ participants

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | HTML5, Vanilla JavaScript, CSS3 |
| **Backend** | Node.js, Express.js |
| **Database** | PostgreSQL (Supabase/Neon) or JSON (file-based) |
| **External APIs** | Google Sheets (XLSX parsing) |
| **Deployment** | Render, Railway, Fly.io |

## 📋 Prerequisites

- Node.js 16+
- npm or yarn
- PostgreSQL (optional; JSON fallback available)
- Google Sheets with student roster data

## 🚀 Quick Start

### 1. Installation

```bash
git clone <repo-url>
cd webdetective
npm install
```

### 2. Environment Setup

Create a `.env` file in the root directory:

```env
# Server Configuration
PORT=3000
BASE_URL=http://localhost:3000

# Admin Authentication
ADMIN_KEY=your-secure-admin-key-here

# Database (Optional - uses JSON if not set)
DATABASE_URL=postgresql://user:password@localhost:5432/webdetective

# Google Sheets Configuration
SHEET_ID=1V6VlldnTD7N5-QzKv1vo-7WXsXGcv5kT-7phLV53u3g
GID=240800619
```

### 3. Run Locally

```bash
npm start
```

- Participant game: `http://localhost:3000/quest2026-treasure-hunt.html`
- Admin dashboard: `http://localhost:3000/admin.html`

## 🗄️ Storage Modes

### Development (File-Based)
- Data stored in `data/store.json`
- Auto-initializes on first run
- No external dependencies

### Production (PostgreSQL)
- Set `DATABASE_URL` environment variable
- Automatic table creation on startup
- Connection pooling with SSL support
- Recommended for 100+ concurrent users

**To switch modes**: Simply set or unset `DATABASE_URL`

## 🔌 API Endpoints

### Participant Endpoints

```http
POST /api/verify-student
Content-Type: application/json

{
  "name": "John Doe",
  "team_name": "Team Alpha",
  "phone_num": "9876543210"
}
```

```http
POST /api/start
Content-Type: application/json

{
  "name": "John Doe",
  "teamName": "Team Alpha"
}
```

```http
POST /api/answer
Content-Type: application/json

{
  "participantId": "uuid-here",
  "clueNumber": 1,
  "answer": "octodex"
}
```

```http
POST /api/complete
Content-Type: application/json

{
  "participantId": "uuid-here",
  "elapsedSeconds": 1245,
  "hintsUsed": 2,
  "penaltySeconds": 120
}
```

### Admin Endpoints

```http
GET /api/admin/results?key=ADMIN_KEY
```

Returns leaderboard with participant stats, completion status, elapsed time, hints used, and final codes.

```http
GET /api/admin/export?key=ADMIN_KEY
```

Downloads CSV file with all participant results and metrics.

```http
GET /api/admin/questions?includeInactive=true&key=ADMIN_KEY
```

Lists all questions in the bank with domain, type, and status.

```http
POST /api/admin/questions?key=ADMIN_KEY
Content-Type: application/json

{
  "id": "gh-new-challenge",
  "domain": "github",
  "type": "GITHUB HUNT",
  "title": "Challenge Title",
  "task": "Task description",
  "clue": "Clue text",
  "link": "https://example.com",
  "hint": "Hint text",
  "answers": ["answer1", "answer2"]
}
```

```http
PUT /api/admin/questions/gh-octocat?key=ADMIN_KEY
Content-Type: application/json

Updates an existing question (any fields)
```

```http
DELETE /api/admin/questions/gh-octocat?key=ADMIN_KEY
```

Soft deactivates a question (remains in DB but marked inactive).

## 📊 Database Schema

### PostgreSQL Tables

```sql
CREATE TABLE participants (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  team_name TEXT NOT NULL,
  assigned_question_ids JSONB NOT NULL,
  current_clue INTEGER NOT NULL DEFAULT 1,
  status TEXT NOT NULL DEFAULT 'active',
  start_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  elapsed_seconds INTEGER,
  hints_used INTEGER NOT NULL DEFAULT 0,
  penalty_seconds INTEGER NOT NULL DEFAULT 0,
  final_code TEXT
);

CREATE TABLE attempts (
  id BIGSERIAL PRIMARY KEY,
  participant_id TEXT NOT NULL REFERENCES participants(id),
  clue_number INTEGER NOT NULL,
  answer TEXT NOT NULL,
  correct BOOLEAN NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE question_usage (
  question_id TEXT PRIMARY KEY,
  usage_count INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE questions (
  id TEXT PRIMARY KEY,
  domain TEXT NOT NULL,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  task TEXT NOT NULL,
  clue TEXT NOT NULL,
  link TEXT,
  hint TEXT NOT NULL,
  answers JSONB NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

## 🎯 Challenge Domains

### GitHub Hunt (7 Questions)
Navigate GitHub repositories, find key repository information, understand structure and metadata.

### Documentation Dive (6 Questions)
Explore official docs for Python, HTML, CSS, and JavaScript from MDN and official sources.

### Wikipedia Fact Find (6 Questions)
Research computing history, AI milestones, technology origins, and programming language creators.

### Neal.fun Explorer (6 Questions)
Interact with interactive visualizations (Space scale, Deep Sea) to extract data and facts.

### Inspect Challenge (6 Questions)
Use browser developer tools to find hidden HTML comments, meta tags, CSS properties, data attributes, and JS variables.

## 🔐 Security

- **Admin Authentication**: API key-based via `x-admin-key` header or `key` query parameter
- **Input Validation**: All inputs trimmed, sanitized, and validated
- **SQL Injection Prevention**: Parameterized queries throughout
- **HTTPS**: Enforced in production (Render/Railway handle automatically)
- **Environment Variables**: Sensitive data stored in `.env`, never committed to git

## 🚢 Deployment

### Deploy to Render

1. Connect GitHub repository to Render
2. Set build command: `npm install`
3. Set start command: `npm start`
4. Add environment variables:
   - `ADMIN_KEY=<your-secret-key>`
   - `DATABASE_URL=<supabase-or-neon-connection-string>`
5. Deploy!

### Deploy to Railway

1. Connect GitHub repository
2. Railway auto-detects Node.js
3. Add PostgreSQL add-on (or use external Supabase)
4. Set environment variables
5. Deploy!

### Using Supabase (Free PostgreSQL)

```env
DATABASE_URL=postgresql://postgres:password@db.supabase.co:5432/postgres
```

Supabase provides 500MB free database perfect for this application.

## 📈 Performance Optimization

- **Connection Pooling**: PostgreSQL pool size optimized for concurrent connections
- **Question Caching**: Questions loaded once and cached in memory
- **DOM Efficiency**: Minimal frontend re-renders using vanilla JS
- **JSON Limit**: Body parser limited to 1MB to prevent abuse

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Unauthorized admin key" | Verify `ADMIN_KEY` in `.env` matches entered key |
| "Failed to load students from Google Sheets" | Check `SHEET_ID` and `GID` are correct and sheet is public |
| "Cannot connect to database" | Verify `DATABASE_URL` format; system falls back to JSON if empty |
| Duplicate participant error | This prevents replay attempts; clear `data/store.json` to reset if needed |
| Question not appearing | Check `is_active` status in questions table; deactivated questions are hidden |

