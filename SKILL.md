---
name: workstation.md
description: Spin up a cloud Linux server in seconds — SSH in, run code, and host a public website at https://<name>.workstation.md
metadata: {"clawdbot":{"emoji":"🖥️","requires":{"commands":["ssh","ssh-keygen"]},"homepage":"https://workstation.md"}}
---

# Workstation.md Skill

Create cloud Linux workstations on demand. Each workstation gives you full root SSH access and a public HTTPS URL. Serve anything on port 80 and it's instantly live at `https://<name>.workstation.md`.

## Setup

```bash
# Install the CLI
npm install -g workstation.md

# Ensure you have an SSH key (skip if you already have one)
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
```

## Core Workflow

```bash
# 1. Create a workstation
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name my-site

# 2. Parse the JSON response for host and port
# {"id":"my-site","host":"47.236.144.174","port":10000,"web":"https://my-site.workstation.md","expires":1711320967}

# 3. SSH in and read the guide
ssh -p <port> root@<host> "cat /root/README.md"

# 4. Build your website (see examples below)

# 5. It's live at https://my-site.workstation.md

# 6. Extend before 24h expires, or destroy when done
workstation my-site extend
workstation my-site destroy
```

## CLI Commands

```bash
workstation create --pubkey "<key>" [--name <name>]   # Create workstation
workstation list                                       # List active workstations
workstation <id> extend [--key ~/.ssh/id_rsa]          # Extend TTL by 24h
workstation <id> destroy                               # Destroy workstation
```

## Running Commands via SSH

```bash
# Single command
ssh -p <port> root@<host> "ls -la"

# Multi-line script
ssh -p <port> root@<host> 'bash -s' << 'EOF'
apt-get update
apt-get install -y <package>
EOF

# Transfer files
scp -P <port> local_file root@<host>:/path/
scp -P <port> -r ./my-project root@<host>:/root/
```

## Creating Websites

Each workstation has nginx running on port 80 by default, serving `/var/www/html/`. The public URL `https://<name>.workstation.md` routes to this port.

### Static HTML

Write HTML directly to the web root:

```bash
ssh -p <port> root@<host> 'bash -s' << 'SCRIPT'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head><title>My Site</title></head>
<body><h1>Hello from my workstation!</h1></body>
</html>
HTML
SCRIPT
```

### Static site with CSS and JS

```bash
ssh -p <port> root@<host> 'bash -s' << 'SCRIPT'
mkdir -p /var/www/html/css /var/www/html/js

cat > /var/www/html/css/style.css << 'CSS'
body { font-family: system-ui; max-width: 800px; margin: 0 auto; padding: 20px; }
CSS

cat > /var/www/html/js/app.js << 'JS'
document.addEventListener('DOMContentLoaded', () => {
  fetch('https://api.example.com/data')
    .then(r => r.json())
    .then(data => document.getElementById('content').textContent = JSON.stringify(data));
});
JS

cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <h1>My App</h1>
  <div id="content">Loading...</div>
  <script src="/js/app.js"></script>
</body>
</html>
HTML
SCRIPT
```

### Python web app (Flask)

```bash
ssh -p <port> root@<host> 'bash -s' << 'SCRIPT'
pip3 install flask

cat > /root/app.py << 'PYTHON'
from flask import Flask, jsonify
import subprocess

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello from Flask!</h1>'

@app.route('/api/info')
def info():
    return jsonify({"status": "running", "server": "workstation"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
PYTHON

# Run Flask on port 3000
nohup python3 /root/app.py &

# Point nginx to Flask
cat > /etc/nginx/sites-enabled/default << 'CONF'
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
CONF
nginx -s reload
SCRIPT
```

### Node.js web app (Express)

```bash
ssh -p <port> root@<host> 'bash -s' << 'SCRIPT'
mkdir -p /root/app && cd /root/app
npm init -y
npm install express

cat > /root/app/server.js << 'JS'
const express = require('express');
const app = express();

app.get('/', (req, res) => res.send('<h1>Hello from Express!</h1>'));
app.get('/api/time', (req, res) => res.json({ time: new Date().toISOString() }));

app.listen(3000, () => console.log('Running on port 3000'));
JS

# Run Express on port 3000
nohup node /root/app/server.js &

# Point nginx to Express
cat > /etc/nginx/sites-enabled/default << 'CONF'
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
CONF
nginx -s reload
SCRIPT
```

### Single-page app with live API data

```bash
ssh -p <port> root@<host> 'bash -s' << 'SCRIPT'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>SF Weather</title>
<style>
  body { font-family: system-ui; max-width: 600px; margin: 40px auto; padding: 0 20px; }
  .weather { font-size: 2em; margin: 20px 0; }
  .detail { color: #666; margin: 8px 0; }
</style>
</head>
<body>
<h1>San Francisco Weather</h1>
<div class="weather" id="temp">Loading...</div>
<div class="detail" id="condition"></div>
<div class="detail" id="wind"></div>
<script>
fetch('https://wttr.in/San+Francisco?format=j1')
  .then(r => r.json())
  .then(data => {
    const current = data.current_condition[0];
    document.getElementById('temp').textContent = current.temp_F + '°F';
    document.getElementById('condition').textContent = current.weatherDesc[0].value;
    document.getElementById('wind').textContent = 'Wind: ' + current.windspeedMiles + ' mph ' + current.winddir16Point;
  });
</script>
</body>
</html>
HTML
SCRIPT
```

## What's Inside the Workstation

- **OS**: Ubuntu 22.04 with root access
- **Languages**: Python 3, Node.js 20
- **Tools**: git, curl, wget, jq, vim, nginx
- **Web server**: nginx on port 80, serving `/var/www/html/`
- **Guide**: `/root/README.md` — full reference for web hosting options
- **Install anything**: `apt-get install -y <package>`

## Tips

1. **Read `/root/README.md` first** after SSHing in — it has the full environment guide
2. **Static files go to `/var/www/html/`** — no restart needed, changes are instant
3. **For dynamic apps**, run on any port and proxy through nginx (see examples above)
4. **nginx is safe to restart** — the container won't die. Run `nginx` to start it again if you stop it
5. **Workstations expire in 24 hours** — run `workstation <id> extend` to renew
6. **Use `nohup` or `&`** for background processes so they survive after SSH disconnects
7. **Transfer files with `scp`** — `scp -P <port> ./file root@<host>:/var/www/html/`

## Example: Full Workflow

```bash
# User asks: "Create a website showing LA weather"

# 1. Install CLI and check SSH key
npm install -g workstation.md
test -f ~/.ssh/id_ed25519.pub || ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519

# 2. Create workstation
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name la-weather
# → {"id":"la-weather","host":"47.236.144.174","port":10001,"web":"https://la-weather.workstation.md","expires":...}

# 3. Deploy weather site
ssh -p 10001 root@47.236.144.174 'bash -s' << 'SITE'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>LA Weather</title>
<style>
  body { font-family: system-ui; max-width: 600px; margin: 40px auto; padding: 0 20px; background: #f5f5f5; }
  h1 { color: #333; }
  .card { background: white; border-radius: 12px; padding: 24px; margin: 16px 0; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
  .temp { font-size: 3em; font-weight: bold; }
  .label { color: #888; font-size: 0.9em; }
</style>
</head>
<body>
<h1>Los Angeles Weather</h1>
<div class="card">
  <div class="label">Temperature</div>
  <div class="temp" id="temp">--</div>
</div>
<div class="card">
  <div class="label">Conditions</div>
  <div id="cond">Loading...</div>
</div>
<div class="card">
  <div class="label">Wind</div>
  <div id="wind">--</div>
</div>
<script>
fetch('https://wttr.in/Los+Angeles?format=j1')
  .then(r => r.json())
  .then(d => {
    const c = d.current_condition[0];
    document.getElementById('temp').textContent = c.temp_F + '°F';
    document.getElementById('cond').textContent = c.weatherDesc[0].value;
    document.getElementById('wind').textContent = c.windspeedMiles + ' mph ' + c.winddir16Point;
  });
</script>
</body>
</html>
HTML
SITE

# 4. Done! Live at https://la-weather.workstation.md
```

## Credits

[workstation.md](https://workstation.md) · [GitHub](https://github.com/workstation-md/workstation-cli) · [npm](https://www.npmjs.com/package/workstation.md)
