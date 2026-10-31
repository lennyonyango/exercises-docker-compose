# Exercise 2.11: Containerised development environment

SvelteKit project with a containerised dev environment. Only Docker and Docker Compose are needed on the host.

## What I did

- Wrote a `Dockerfile` that installs dependencies with `npm ci` (package files copied first for layer caching) and starts the Vite dev server with `--host`.
- Added a `.dockerignore` (`node_modules`, `.svelte-kit`, `build`, `.git`, `.env*`).
- Added a `compose.yaml` that builds the image, publishes port 5173 and mounts the source for hot reload.

## Running

```sh
docker compose up --build
```

The app is at http://localhost:5173. Editing a file on the host updates the browser immediately.

## Key decisions

- `--host` is needed because Vite otherwise listens only on `localhost`, which is unreachable from outside the container.
- The project folder is bind mounted for live edits, and a named volume is mounted over `node_modules` so the host folder does not hide the container's dependencies:

  ```yaml
  volumes:
    - .:/app
    - node_modules:/app/node_modules
  ```

  After adding a dependency, run `docker compose down -v` before rebuilding.
- The source is not copied into the image, because the bind mount provides it. This keeps rebuilds fast.
- The project lives in the Linux filesystem (WSL), so file watching works without polling.

## Without Docker

```sh
npm install
npm run dev
```