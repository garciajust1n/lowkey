# Frontend Setup

Static HTML/CSS/vanilla JS admin + applicant site for OEEMS. No build step, no
`package.json`, no npm dependencies.

## Prerequisites

- Any static file server (Node's `serve`, Python's `http.server`, or a Docker
  container — see below).
- A running instance of the [backend](../backend/SETUP.md) to talk to.

## Running locally

```
cd frontend
npx serve .
```

Or any other static server — the frontend is plain files, nothing to compile.

## Pointing at the backend

The frontend calls the API at `window.__ENV__.BACKEND_URL`, which is read at
page-load time from `frontend/env-config.js`:

```js
window.__ENV__ = { BACKEND_URL: "http://localhost:8000" };
```

For local development this file is checked in with a `http://localhost:8000`
default — edit it directly if your backend runs somewhere else. When run via
Docker, this file is regenerated at container start (see below), so don't rely
on manual edits surviving a container rebuild.

## Docker

```
cd frontend
docker build -t oeems-frontend .
docker run --rm -p 3000:3000 -e BACKEND_URL=http://localhost:8000 oeems-frontend
```

The image is `node:20-alpine` with the `serve` package installed globally (no
nginx). `entrypoint.sh` regenerates `env-config.js` from the `BACKEND_URL` env
var at container start, then runs `serve -s /app -l 3000`.

**`BACKEND_URL` is read by the end user's browser, not by the frontend
container.** It must be a URL the browser can reach — e.g. `http://localhost:8000`
or a public hostname. Never set it to the Docker-internal service name
(`http://backend:8000`); a browser on the host machine cannot resolve that.

To run both frontend and backend together, use `docker compose up --build` from
the repo root — see the root [README.md](../README.md#docker-compose).

## Structure

- Each HTML page pairs with a same-named or feature-named `.js` file under `js/`
  (ES modules, `type="module"` on applicant-facing pages that import shared
  modules like `session.js`; admin pages that don't need imports stay classic
  scripts).
- `js/apiService.js` — the only module that calls the backend (`fetch`, JWT
  attached from `js/session.js`).
- `js/session.js` — stores/reads the JWT (`localStorage`), decodes it for the
  current user's id/email.
- `env-config.js` — runtime backend URL, see above.
