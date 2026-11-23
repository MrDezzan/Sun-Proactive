<h1 align="center">Sun Proactive ☀️</h1>
<p align="center"><b>AI-powered platform for social volunteering.</b><br>Connects project curators with the right volunteers, using vector matching and automated verification.</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Next.js-15-000000?style=flat-square&logo=nextdotjs&logoColor=white" alt="Next.js">
  <img src="https://img.shields.io/badge/Fastify-000000?style=flat-square&logo=fastify&logoColor=white" alt="Fastify">
  <img src="https://img.shields.io/badge/PostgreSQL-pgvector-316192?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Tailwind-4-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white" alt="Tailwind CSS">
</p>

---

## What it does

A curator posts a volunteering task. Volunteers sign up with their skills. The platform ranks applicants for the curator, explains each recommendation, answers volunteers' questions about the task, and checks the photo report once the work is done.

## AI features

| Feature | How it works |
|---|---|
| **AI matching** | Task descriptions and volunteer skills are turned into embeddings (`text-embedding-3-small`) and ranked by cosine similarity. |
| **Explainable AI** | Every recommendation comes with a short natural-language reason. |
| **AI consultant (RAG)** | A chatbot on each task page answers only from the task description, which avoids made-up answers and saves curators from repeating themselves. |
| **Vision AI** | Photo reports are checked with **Google Gemini 1.5 Flash**: does the photo match the task, and is anyone trying a prompt injection. |
| **Autonomous AI manager** | A cron job finds tasks with approaching deadlines and sends push notifications to the volunteers who fit best. |

## Roles

1. **Volunteer (12+)**: picks skills at sign-up, browses open tasks, gets AI help on task pages and submits photo reports that are verified by AI.
2. **Curator**: creates tasks through an AI-guided dialogue, reviews AI-ranked applications with explanations and approves participants. Needs admin approval to activate.
3. **Administrator**: manages users, approves or rejects curator applications and moderates the platform.

## Tech stack

| Layer | Technology |
|---|---|
| Backend | Fastify 5, TypeScript, Drizzle ORM, PostgreSQL |
| Frontend | Next.js 15 (App Router), Tailwind CSS 4, a central API client with automatic JWT handling |
| AI | OpenAI-compatible API (embeddings, matching, RAG), Google Gemini (vision) |
| Security | bcrypt (12 rounds), stateless JWT, role-based access guards, Helmet, rate limiting |

The frontend and backend are separate services. The old monolithic Next.js API routes were removed.

## Security

- Passwords are hashed with bcrypt, never stored as plain text.
- Stateless JWT sessions. `authGuard` and `adminGuard` check access on every protected route.
- Secure headers through `@fastify/helmet`.
- Rate limit of 100 requests per minute per IP.
- Background AI jobs authenticate with a secret header, so nobody can trigger them from outside.

## Project structure

```
sun-proactive-backend/
  src/
    routes/        auth, tasks, chat, verify, notifications, admin, cron, teams, friends
    services/      ai.ts (LLM, RAG), vector.ts (embeddings, matching)
    middleware/    JWT and role guards
    db/            Drizzle schema and connection
sun-proactive/     Next.js app (volunteer, curator and admin areas)
```

## Run locally

You need a recent Node.js LTS, Docker and an OpenAI-compatible API key. A Gemini key is needed only for photo verification.

```bash
# 1. Database (PostgreSQL + pgvector)
cd sun-proactive && docker compose up -d

# 2. Backend
cd ../sun-proactive-backend
cp .env.example .env          # fill in the keys
npm install
npm run db:push               # create tables
npx tsx src/scripts/create-admin.ts
npm run dev

# 3. Frontend (new terminal)
cd ../sun-proactive
npm install
NEXT_PUBLIC_API_URL=http://localhost:8080 npm run dev
```

## My role

Sole developer. I designed and built the system end to end: the AI matching pipeline (embeddings, cosine ranking, explanations), the RAG chatbot, Vision AI verification, the Fastify backend with its JWT and role-based security, and the Next.js frontend.
