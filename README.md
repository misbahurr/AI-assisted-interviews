# AI-Assisted Interviews

A real-time AI-powered voice interview platform. Enter your GitHub profile, get interviewed by an AI on your technical background, and receive a score with detailed feedback.

## Demo Flow

1. **Submit** your GitHub URL
2. **Interview** — AI conducts a 2–3 question voice interview personalized from your repos
3. **Results** — receive a score out of 10 and written feedback

## Tech Stack

| Layer | Tech |
|---|---|
| Frontend | React 19, TypeScript, React Router v7, Tailwind CSS v4, shadcn/ui |
| Backend | Express.js v5, Prisma v7, PostgreSQL |
| AI — Voice | OpenAI Realtime API (WebRTC) |
| AI — Scoring | Google Gemini 3.5 Flash |
| Transcription | Deepgram WebSocket SDK |
| Monorepo | Turborepo + Bun |

## Architecture

```
Browser
  ├─ MediaRecorder → Deepgram WebSocket → transcription → POST /api/v1/session/user/response/:id
  ├─ RTCPeerConnection ──SDP offer──▶ backend ──▶ OpenAI Realtime API (WebSocket sideband)
  └─ GET /api/v1/result/:id → Gemini 3.5 Flash → score + feedback
```

## Project Structure

```
apps/
  backend/    # Express.js API + WebSocket handlers (port 3001)
  frontend/   # React SPA (port 3000)
packages/
  ui/               # Shared component library
  eslint-config/
  typescript-config/
```

## Getting Started

### Prerequisites

- [Bun](https://bun.sh) >= 1.3.11
- PostgreSQL database

### Environment Variables

Create `apps/backend/.env`:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/ai_interviews
OPENAI_KEY=your_openai_api_key
GEMINI_API_KEY=your_gemini_api_key
PROXY_URL=your_proxy_url_for_github_scraping
```

Create `apps/frontend/.env`:

```env
BACKEND_URL=http://localhost:3001
```

### Install & Run

```bash
# Install dependencies
bun install

# Run database migrations
cd apps/backend && bunx prisma migrate deploy

# Start all services
bun dev
```

Frontend: http://localhost:3000  
Backend: http://localhost:3001

## API Routes

| Method | Route | Description |
|---|---|---|
| `POST` | `/api/v1/pre-interview` | Scrape GitHub, create interview session |
| `POST` | `/api/v1/session/:id` | SDP handshake, start OpenAI Realtime connection |
| `POST` | `/api/v1/session/user/response/:id` | Save user transcription |
| `GET` | `/api/v1/result/:id` | Get score, feedback, and full transcript |

## Database Schema

```prisma
model Interview {
  id             String          @id @default(uuid())
  githubMetadata Json
  status         InterviewStatus  # Pre | InProgress | Done
  score          Int
  feedback       String?
  conversations  Message[]
}

model Message {
  id          String      @id @default(uuid())
  message     String
  type        MessageType  # User | Assistant
  interviewId String
  createdAt   DateTime    @default(now())
}
```
