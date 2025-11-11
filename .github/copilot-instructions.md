## Copilot / AI agent instructions for this repo

Goal: help an AI coding agent become productive quickly by describing the architecture, developer workflows, conventions, and important gotchas discovered in the codebase.

- Project entry points and components
  - `src/app.py` — FastAPI application object named `app`. Routes are defined here and the app mounts static assets from `src/static`.
  - `src/static/` — single-page front-end served at `/static/index.html`. Root (`/`) redirects to `/static/index.html` in `app.py`.
  - `src/README.md` — short developer notes about endpoints and intended run steps.

- Big-picture architecture
  - Single-process FastAPI app with an in-memory datastore: the `activities` dict in `src/app.py` is the canonical data model. Data resets when the server restarts.
  - No database, no auth, and no background workers. All state changes are performed in-process by mutating the `activities` dict.
  - Static UI is served from the same app under `/static` (mounted using `StaticFiles`).

- Important developer workflows (how to run and debug)
  - Install dependencies: `pip install -r requirements.txt` (file lists `fastapi` and `uvicorn`).
  - Preferred local run (stable):

    uvicorn src.app:app --reload --host 0.0.0.0 --port 8000

    Notes: The top-level README suggests `python app.py`, but `src/app.py` only defines the FastAPI `app` object (no `if __name__ == '__main__'` runner). Use `uvicorn` to serve the app.
  - API docs and quick debug UI:
    - Open http://localhost:8000/docs (Swagger UI)
    - Or http://localhost:8000/redoc
  - Static UI access: http://localhost:8000/static/index.html (root `/` redirects to this page).

- API surface and contract (quick reference)
  - GET /activities
    - Returns the whole `activities` dict (keyed by activity name).
  - POST /activities/{activity_name}/signup?email=student@mergington.edu
    - Signs up the provided email for the named activity.
    - Example (note URL-encoding for spaces in activity names):

      curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=alice@mergington.edu"

    - Possible errors: 404 if activity not found; 400 if student already signed up.
  - DELETE /activities/{activity_name}/unregister?email=student@mergington.edu
    - Removes the email from participants. Errors: 404 activity not found; 400 if student not signed up.

- Project-specific conventions and gotchas
  - Activity identifier: activity names (e.g., "Chess Club") are used as keys. They are case-sensitive and may contain spaces — always URL-encode them in path segments.
  - Emails are used as the primary identifier for students across the data model.
  - Data persistence: data is in-memory only. Any change you make will be lost on server restart — useful for transient demos, not for production data migration work.
  - Concurrency: the `activities` dict is mutated directly without locks. For concurrent load or multi-worker uvicorn setups, behavior is not safe or deterministic.

- Where to look for examples
  - `src/app.py` — canonical examples of request handling, error responses (raises `HTTPException` with 400/404), and how data is structured.
  - `src/README.md` — quick start and API table that mirrors the implemented endpoints.
  - `src/static/index.html` — front-end example that interacts with the API (useful for UI changes or integration tests).

- Guidance for change proposals an AI agent might produce
  - Keep changes minimal and focused. When updating behavior that touches the in-memory store, include tests or a short local-run verification (start server + call endpoint) to show the effect.
  - If adding persistence (DB) or concurrency controls, document migration steps and the temporary compatibility layer (the project currently assumes everything is in-memory).

- Quick checklist for the next steps an agent should run locally
  1. pip install -r requirements.txt
  2. uvicorn src.app:app --reload --port 8000
  3. Visit /docs and try POST/DELETE flows using the example above

If any section is unclear or you want examples expanded (for instance, add sample tests or a minimal `uvicorn` runner in `app.py`), tell me which part to expand and I will update this file.
