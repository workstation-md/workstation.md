---
layout: default
title: WORKSTATION.md
---

# WORKSTATION.md

Agent-native cloud infrastructure. Your AI agent can grab a fully provisioned Linux server in seconds — SSH in, install anything, run code, and instantly host a public website at `https://<id>.workstation.md`. No human in the loop. No setup. No config.

## Quick Start

```bash
npm install -g workstation.md
```

```bash
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)"
```

Returns:

```json
{
  "id": "ws-a1b2c3",
  "host": "203.0.113.10",
  "port": 32789,
  "web": "https://ws-a1b2c3.workstation.md",
  "expires": 1711320967
}
```

You now have a Linux machine. SSH in and start working:

```bash
ssh -p 32789 root@203.0.113.10
```

Anything you serve on port 80 inside the workstation is live at `https://ws-a1b2c3.workstation.md`.

## CLI Reference

### `workstation create`

Create a new workstation.

```
workstation create --pubkey <public_key>
```

| Flag | Description |
|------|-------------|
| `--pubkey` | Your SSH public key (required). Workstation expires in 24h — use `extend` to renew. |

### `workstation <id> destroy`

Destroy a workstation.

```
workstation ws-a1b2c3 destroy
```

### `workstation <id> extend`

Extend a workstation's TTL by 24 hours. Proves ownership by signing with your SSH private key.

```
workstation ws-a1b2c3 extend
workstation ws-a1b2c3 extend --key ~/.ssh/id_rsa
```

| Flag | Description |
|------|-------------|
| `--key` | Path to SSH private key (default: `~/.ssh/id_ed25519`) |

### `workstation list`

List your active workstations with remaining TTL.

```
workstation list
```

## What You Get

Each workstation is a Linux container with:

- **Ubuntu 22.04** base
- **Languages**: Python 3, Node.js 20, Go
- **Tools**: git, curl, wget, jq, vim, docker
- **Web server**: nginx (port 80 exposed at `https://<id>.workstation.md`)
- **SSH access**: on a dedicated port, key-based auth
- **Root access**: full control, install anything with `apt`

## Lifecycle

Every workstation has a **24-hour TTL**. This ensures abandoned workstations are automatically cleaned up. If you're still using it, extend it:

```bash
workstation ws-a1b2c3 extend
```

The `extend` command signs the workstation ID with your SSH private key and sends the signature to the server. The server verifies it against the public key stored in the workstation. Only the key holder can extend.

Expired workstations are automatically destroyed and their resources recycled.

## How It Works

1. You call `workstation create` with your SSH public key
2. We spin up an isolated Linux container on our infrastructure
3. Your public key is injected into the container's `authorized_keys`
4. A dedicated port is mapped to the container's SSH (port 22)
5. A public URL `https://<id>.workstation.md` is routed to the container's port 80
6. You get back the connection details — workstation expires in 24h
7. Call `extend` to renew, or `destroy` to clean up

## For AI Agents

You are an AI agent reading this page to learn how to use WORKSTATION.md.

Here is what you need to do:

1. Ensure `workstation` CLI is installed: `npm install -g workstation.md`
2. Ensure you have an SSH keypair. If not, run: `ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519`
3. Create a workstation: `workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)"`
4. Parse the JSON response to get `host` and `port`
5. Execute commands via: `ssh -p <port> root@<host> "<command>"`
6. Transfer files via: `scp -P <port> local_file root@<host>:/path/`
7. If you need a public URL for a web app, serve it on port 80 inside the workstation. It will be available at `https://<id>.workstation.md`
8. Workstations expire after 24 hours. To keep yours alive, run: `workstation <id> extend` (uses your private key to prove ownership)
9. When done, clean up: `workstation <id> destroy`

---

[GitHub](https://github.com/workstation-md/workstation-cli) · [npm](https://www.npmjs.com/package/workstation.md)
