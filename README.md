# Thai Drama Watchlist — Frontend

Vue 3 (Single File Component, `<script setup>`) + axios, connected to the Express backend.

## Run
1. Start the backend first (http://localhost:3000)
2. `cp .env.example .env` (`VITE_BASE_URL=http://localhost:3000`)
3. `pnpm i` then `pnpm run dev` → http://localhost:5173

## What it does (CRUD)
- **Create**: form → `POST /dramas`
- **Read**: list + filter (All / To watch / Watched) → `GET /dramas?watched=...`
- **Update**: Edit button → `PUT /dramas/:id`, "Mark watched" → `PATCH /dramas/:id`
- **Delete**: Delete button → `DELETE /dramas/:id`
