# AGENTS.md — Trade-Fat

## What this repo is

- The entire app is one self-contained static page: `Metatrade-Pro-24-Terminal.html` (~200 KB). React and Tailwind are already bundled **inline** inside it, and the page is a client-side-only trading-terminal simulation (French UI, virtual funds).
- There is no framework source, no `package.json`, no build step, no backend, no database and no server-side API. The page makes no network calls of its own (its only external request is Google Fonts).
- The file came from the repo branch `abdelaboykine-byte-patch-1` ("Add files via upload"). The Base44 working branch had only a README, so the page was checked onto the working branch during environment setup — it is the project.
- It is configured as nginx's directory index, so the site's home page is `/` (the raw file is also reachable at `/Metatrade-Pro-24-Terminal.html`).

## Running it (Base44 sandbox)

```bash
docker compose -f docker-compose.base44.yml up -d --build
curl -sI http://localhost:3000/                 # expect 200
docker compose ps                               # web should be healthy
docker compose -f docker-compose.base44.yml logs -f web
docker compose -f docker-compose.base44.yml down
```

- One service: `web` = `nginx:alpine`. Repo root is bind-mounted at `/usr/share/nginx/html`; server config lives in `.base44/nginx.conf`. Host port 3000 → container port 80.
- No environment variables or secrets are required, so nothing is read from `/run/base44/app.env`.

## Sandbox quirks (non-obvious, easy to break)

- The platform clones the repo into a root-only-readable directory (mode `0700`), which nginx's unprivileged worker user cannot traverse: the stock `nginx:alpine` serving that mount answers **403 Forbidden** (not 404), and the container restarts if the fix itself fails. The compose `command` therefore runs `chmod o+rx /usr/share/nginx/html` before starting nginx — which is also why that mount is deliberately *not* `:ro` (chmod on a read-only mount fails with "Read-only file system"). Keep the command and the mount mode together.
- Verify by checking the body, not just the status code — a directory listing or an nginx error page would also return 200: `curl -s http://localhost:3000/ | grep -c 'id="root"'`.

## Editing notes

- No watcher or HMR: nothing is compiled, nginx reads the bind-mounted files on each request, so an edit shows on the next browser refresh. Call `reload_preview` after changing the HTML.
- There is no bundler to install packages with — anything the page needs at runtime must stay inline in the file (or be loaded from the public internet).
- Keep the app bootable: the only requirement for a working preview is that nginx serves `/` with HTTP 200 and a non-empty page.
