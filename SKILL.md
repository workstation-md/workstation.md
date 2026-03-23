name: workstation.md
description: Give your agent a cloud Linux server in seconds. SSH access, root permissions, and a public HTTPS URL — all from one command. TLDR: just paste https://workstation.md to your agent and ask it to build you a website.
emoji: 🖥️
requires: ssh, ssh-keygen
homepage: https://workstation.md

# Workstation.md

Your own cloud Linux server in seconds. Full root SSH access. Anything you serve on port 80 is instantly live at `https://<name>.workstation.md`.

## Setup (one-time)

```bash
npm install -g workstation.md
test -f ~/.ssh/id_ed25519 || ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
```

## Create a Workstation

```bash
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name <name>
```

Response:

```
{"id":"<name>","host":"...","port":10000,"web":"https://<name>.workstation.md","expires":...}
```

## Use It

```bash
# Run any command
ssh -p <port> root@<host> "<command>"

# Run a script
ssh -p <port> root@<host> 'bash -s' << 'EOF'
<your commands here>
EOF

# Copy files
scp -P <port> ./local-file root@<host>:/var/www/html/
```

## Host a Website

nginx serves `/var/www/html/` on port 80. Your public URL points there.

Three ways to deploy:

1. **Static files** — write to `/var/www/html/`. Instant, no restart.
2. **Your own app** — stop nginx (`nginx -s stop`), run your app on port 80.
3. **Reverse proxy** — run your app on any port, configure nginx to proxy to it, `nginx -s reload`.

Read `/root/README.md` inside the workstation for detailed examples of each method.

## Manage

```bash
workstation list                # list active workstations
workstation <name> extend       # renew for another 24h
workstation <name> destroy      # clean up
```

## What's Inside

- Ubuntu 22.04 with root access
- Python 3, Node.js 20, git, curl, wget, jq, vim, nginx
- `/root/README.md` with full environment guide
- Install anything with `apt-get install -y`

## Things You Can Build

Ask your human what they want, then create a workstation and build it:

- "Show me the weather in San Francisco" — fetch from wttr.in API, render a clean page
- "Build a personal portfolio site" — static HTML/CSS in /var/www/html/
- "Create a todo app" — Express or Flask backend + frontend
- "Make a dashboard showing my GitHub stats" — fetch GitHub API, render charts
- "Set up a URL shortener" — simple key-value store with redirects
- "Build a countdown timer for my wedding" — static page with JS countdown
- "Create a pastebin" — text input + storage + shareable links
- "Show a live crypto price tracker" — WebSocket or polling from crypto APIs
- "Make a meme generator" — upload image, overlay text, serve result
- "Build a simple chat room" — WebSocket server + chat UI
- "Create an API that returns random quotes" — JSON endpoint + static data
- "Set up a webhook receiver" — Flask/Express endpoint that logs incoming hooks
- "Build a file upload service" — multipart form upload + serve uploaded files
- "Make a Pomodoro timer with sounds" — static page with Web Audio API
- "Create a link-in-bio page" — styled list of links, like Linktree
- "Build a markdown previewer" — textarea input, live rendered preview
- "Show a map of coffee shops near me" — Leaflet.js + Overpass API
- "Create a simple blog" — markdown files rendered as HTML pages
- "Build a QR code generator" — input text, generate QR image
- "Set up a status page for my services" — periodic health checks, display results

## Tips

- Static files in `/var/www/html/` are served instantly — no restart needed
- nginx is safe to restart or stop — the container stays alive
- Use `nohup` or `&` for background processes so they survive SSH disconnect
- Workstations expire in 24h — extend before they do
- Read `/root/README.md` after SSHing in — it's the full reference
