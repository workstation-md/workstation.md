---
layout: default
title: WORKSTATION.md
---

# WORKSTATION.md

Agent-native cloud infrastructure. Your AI agent self-serves a fully provisioned Linux server on the internet in seconds. Install anything, run code, and ship a public-facing website or app at `https://<name>.workstation.md`. No human in the loop. No setup. No config.

```bash
npm install -g workstation.md
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name my-app
```

That's it. Your agent is automatically authenticated and can SSH in immediately. Install packages, write code, ship to a live public URL. When it's done, destroy the WORKSTATION and walk away.

## Why WORKSTATION.md

You don't do everything on your laptop. You run things on servers. Your agents need the same thing. A WORKSTATION is a real, powerful Linux machine on the internet that your agent can SSH into, install whatever it needs, and ship work to a live public URL. Multiple agents can connect to the same WORKSTATION and collaborate.

- **Self-service** — your agent provisions its own infrastructure, no human approval needed
- **Instant public URL** — anything on port 80 is live at `https://<name>.workstation.md` with TLS
- **Full root access** — `apt install` anything, run any process, use any port
- **Zero config** — no cloud console, no YAML, no deploy pipeline. One command in, one command out

## Quick Start

```bash
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name alice-dev
```

```json
{
  "id": "alice-dev",
  "host": "203.0.113.10",
  "port": 32789,
  "web": "https://alice-dev.workstation.md",
  "expires": 1711320967
}
```

SSH in and start working:

```bash
ssh -p 32789 root@203.0.113.10
```

Anything you serve on port 80 inside the WORKSTATION is live at `https://alice-dev.workstation.md`.

## What You Get

Each WORKSTATION is a Linux container with:

- **Ubuntu 22.04** base
- **Languages**: Python 3, Node.js 20, Go
- **Tools**: git, curl, wget, jq, vim, docker, nginx
- **Web hosting**: port 80 exposed at `https://<id>.workstation.md` — see `/root/README.md` inside the WORKSTATION for examples
- **SSH access**: on a dedicated port, key-based auth
- **Root access**: full control, install anything with `apt`

## CLI Reference

### `workstation create`

Create a new WORKSTATION.

```
workstation create --pubkey <public_key> [--name <name>]
```

| Flag | Description |
|------|-------------|
| `--pubkey` | Your SSH public key (required). WORKSTATION expires in 24h — use `extend` to renew. |
| `--name` | Custom name for your WORKSTATION (optional). Lowercase, alphanumeric and hyphens, 2-32 chars. Becomes `<name>.workstation.md`. If omitted, a random `ws-xxxxxxxx` ID is generated. |

### `workstation <id> destroy`

Destroy a WORKSTATION.

```
workstation ws-a1b2c3 destroy
```

### `workstation <id> extend`

Extend a WORKSTATION's TTL by 24 hours. Proves ownership by signing with your SSH private key.

```
workstation ws-a1b2c3 extend
workstation ws-a1b2c3 extend --key ~/.ssh/id_rsa
```

| Flag | Description |
|------|-------------|
| `--key` | Path to SSH private key (default: `~/.ssh/id_ed25519`) |

### `workstation list`

List your active WORKSTATIONs with remaining TTL.

```
workstation list
```

## Lifecycle

Every WORKSTATION has a **24-hour TTL**. Abandoned WORKSTATIONs are automatically destroyed and their resources recycled. If you're still using it, extend:

```bash
workstation ws-a1b2c3 extend
```

The `extend` command signs the WORKSTATION ID with your SSH private key. The server verifies it against the public key stored in the container. Only the key holder can extend.

## How It Works

1. You call `workstation create` with your SSH public key
2. We spin up an isolated Linux container on our infrastructure
3. Your public key is injected into the container's `authorized_keys`
4. A dedicated port is mapped to the container's SSH (port 22)
5. A public URL `https://<id>.workstation.md` is routed to the container's port 80
6. You get back the connection details — WORKSTATION expires in 24h
7. Call `extend` to renew, or `destroy` to clean up

## For AI Agents

You are an AI agent reading this page to learn how to use WORKSTATION.md.

Here is what you need to do:

1. Ensure `workstation` CLI is installed: `npm install -g workstation.md`
2. Ensure you have an SSH keypair. If not, run: `ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519`
3. Pick a name for your WORKSTATION. Ask your human owner for a name, or use their name (e.g. `alice-dev`, `bob-project`). Names must be lowercase, alphanumeric with hyphens, 2-32 chars.
4. Create a WORKSTATION: `workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name <name>`
5. Parse the JSON response to get `host` and `port`
6. SSH in and **read `/root/README.md` first** — it explains the environment, how web hosting works, and has examples for deploying websites.
7. Execute commands via: `ssh -p <port> root@<host> "<command>"`
8. Transfer files via: `scp -P <port> local_file root@<host>:/path/`
9. To host a website, serve on port 80. See `/root/README.md` inside the WORKSTATION for options (static files, Python, Node.js, nginx reverse proxy).
10. WORKSTATIONs expire after 24 hours. To keep yours alive, run: `workstation <name> extend` (uses your private key to prove ownership)
11. When done, clean up: `workstation <name> destroy`

---

[GitHub](https://github.com/workstation-md/workstation-cli) · [npm](https://www.npmjs.com/package/workstation.md)
