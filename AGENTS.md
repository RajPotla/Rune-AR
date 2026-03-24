## Cursor Cloud specific instructions

### Project structure

All source code lives under `Rune-AR/`:
- **Frontend** (React + Vite + TypeScript): `Rune-AR/front-end/helix/`
- **Backend** (Flask + SocketIO + SQLAlchemy): `Rune-AR/back-end/server/`
- **Docker Compose**: `Rune-AR/docker-compose.yml` (for containerized dev — not needed for local dev)

### Running services locally (without Docker Compose)

| Service | Command | Port | Working directory |
|---------|---------|------|-------------------|
| PostgreSQL | `docker start rune-postgres` (or create with `docker run -d --name rune-postgres -e POSTGRES_USER=rune -e POSTGRES_PASSWORD=runepass -e POSTGRES_DB=runedb -p 5432:5432 postgres:15-alpine`) | 5432 | N/A |
| Backend | `DATABASE_URL=postgresql://rune:runepass@localhost:5432/runedb OPENAI_API_KEY=$OPENAI_API_KEY python3 run.py` | 5001 | `Rune-AR/back-end/server/` |
| Frontend | `npm run dev -- --host` | 5173 | `Rune-AR/front-end/helix/` |

Start services in order: PostgreSQL -> Backend -> Frontend.

### Key gotchas

- **Vite proxy**: `vite.config.ts` proxy target must be `http://localhost:5001` for local dev (it defaults to `http://backend:5001` which is the Docker Compose service name).
- **dotenv path**: `app/config.py` uses `load_dotenv(dotenv_path="../../../.env")` which resolves relative to CWD. When running from `back-end/server/`, this resolves to `/workspace/.env` which may not exist. Set `DATABASE_URL` and `OPENAI_API_KEY` as environment variables directly instead of relying on the `.env` file.
- **Database migrations**: Run `FLASK_APP=run.py python3 -m flask db upgrade` from `Rune-AR/back-end/server/` after PostgreSQL is running.
- **No automated test suite**: The project has no pytest or Jest tests configured. Lint is available via `npm run lint` in the frontend directory.
- **OPENAI_API_KEY**: Required for AI chat features. Without a valid key, the app loads and the chat UI works but AI responses will fail. Set the `OPENAI_API_KEY` secret in the Cursor Cloud environment.
- **Frontend build**: `npm run build` (in `Rune-AR/front-end/helix/`) runs TypeScript checking + Vite build.
- **Docker must be running** for PostgreSQL container. Start dockerd with `sudo dockerd &>/tmp/dockerd.log &` if needed, then ensure socket permissions with `sudo chmod 666 /var/run/docker.sock`.
