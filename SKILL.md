name: workstation.md
description: Spin up a cloud Linux server in seconds — SSH in, run code, and host a public website at https://<name>.workstation.md
emoji: 🖥️
requires: ssh, ssh-keygen
homepage: https://workstation.md

# Workstation.md

Create cloud Linux workstations on demand. Each workstation gives you full root SSH access and a public HTTPS URL. Serve anything on port 80 and it's instantly live at `https://<name>.workstation.md`.

## Setup

```bash
npm install -g workstation.md
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519   # skip if you already have a key
```

## Workflow

```bash
# 1. Create a workstation (pick a name, e.g. "my-site")
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name my-site

# Response:
# {"id":"my-site","host":"47.236.144.174","port":10000,"web":"https://my-site.workstation.md","expires":1711320967}

# 2. Run commands via SSH
ssh -p <port> root@<host> "<command>"

# 3. Read the in-container guide
ssh -p <port> root@<host> "cat /root/README.md"

# 4. Deploy your website (see examples below)
# Anything on port 80 is live at https://my-site.workstation.md

# 5. Manage lifecycle
workstation my-site extend    # renew for another 24h
workstation my-site destroy   # clean up when done
workstation list              # see all active workstations
```

## SSH Patterns

```bash
# Single command
ssh -p <port> root@<host> "apt-get update && apt-get install -y <package>"

# Multi-line script
ssh -p <port> root@<host> 'bash -s' << 'EOF'
echo "running multiple commands"
apt-get update
apt-get install -y python3-pip
pip3 install flask
EOF

# Transfer files to workstation
scp -P <port> ./file root@<host>:/var/www/html/
scp -P <port> -r ./project root@<host>:/root/
```

## Creating Websites

nginx is running on port 80 by default, serving /var/www/html/.
The public URL https://<name>.workstation.md routes to port 80.

### Method 1: Static files (simplest)

Write files to /var/www/html/ — changes are instant, no restart needed.

```bash
ssh -p <port> root@<host> 'bash -s' << 'EOF'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>My Site</title></head>
<body><h1>Hello World</h1></body>
</html>
HTML
EOF
```

### Method 2: Static site with CSS/JS and API data

```bash
ssh -p <port> root@<host> 'bash -s' << 'EOF'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Weather</title>
<style>
body { font-family: system-ui; max-width: 600px; margin: 40px auto; padding: 0 20px; }
.card { background: #fff; border-radius: 12px; padding: 24px; margin: 16px 0; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.big { font-size: 3em; font-weight: bold; }
</style>
</head>
<body>
<h1>Weather</h1>
<div class="card"><div class="big" id="temp">--</div></div>
<div class="card" id="detail">Loading...</div>
<script>
fetch('https://wttr.in/San+Francisco?format=j1')
  .then(r => r.json())
  .then(d => {
    const c = d.current_condition[0];
    document.getElementById('temp').textContent = c.temp_F + '°F';
    document.getElementById('detail').textContent = c.weatherDesc[0].value + ' — Wind: ' + c.windspeedMiles + ' mph';
  });
</script>
</body>
</html>
HTML
EOF
```

### Method 3: Python app (Flask)

```bash
ssh -p <port> root@<host> 'bash -s' << 'EOF'
pip3 install flask

cat > /root/app.py << 'PY'
from flask import Flask, jsonify
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello from Flask</h1>'

@app.route('/api/status')
def status():
    return jsonify({"status": "ok"})
PY

nohup python3 /root/app.py &

# Proxy nginx to Flask
cat > /etc/nginx/sites-enabled/default << 'CONF'
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
    }
}
CONF
nginx -s reload
EOF
```

### Method 4: Node.js app (Express)

```bash
ssh -p <port> root@<host> 'bash -s' << 'EOF'
mkdir -p /root/app && cd /root/app
npm init -y && npm install express

cat > /root/app/server.js << 'JS'
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('<h1>Hello from Express</h1>'));
app.get('/api/time', (req, res) => res.json({ time: new Date().toISOString() }));
app.listen(3000);
JS

nohup node /root/app/server.js &

cat > /etc/nginx/sites-enabled/default << 'CONF'
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
CONF
nginx -s reload
EOF
```

## Environment

- Ubuntu 22.04, root access
- Python 3, Node.js 20, git, curl, wget, jq, vim, nginx
- /root/README.md — full environment guide
- /var/www/html/ — nginx web root (port 80)
- Install anything: apt-get install -y <package>

## Tips

- Read /root/README.md after SSHing in — it has the full guide
- Static files in /var/www/html/ are served instantly, no restart needed
- For dynamic apps, run on any port and proxy through nginx
- nginx is safe to restart or stop — the container stays alive
- Use nohup or & for background processes so they survive SSH disconnect
- Workstations expire in 24 hours — run `workstation <id> extend` to renew
- Transfer files: scp -P <port> ./file root@<host>:/var/www/html/

## Full Example

User asks: "Create a website showing LA weather"

```bash
# Setup
npm install -g workstation.md
test -f ~/.ssh/id_ed25519.pub || ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519

# Create workstation
workstation create --pubkey "$(cat ~/.ssh/id_ed25519.pub)" --name la-weather
# {"id":"la-weather","host":"47.236.144.174","port":10001,"web":"https://la-weather.workstation.md",...}

# Deploy
ssh -p 10001 root@47.236.144.174 'bash -s' << 'EOF'
cat > /var/www/html/index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>LA Weather</title>
<style>
body { font-family: system-ui; max-width: 600px; margin: 40px auto; padding: 0 20px; background: #f5f5f5; }
.card { background: #fff; border-radius: 12px; padding: 24px; margin: 16px 0; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
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
<script>
fetch('https://wttr.in/Los+Angeles?format=j1')
  .then(r => r.json())
  .then(d => {
    const c = d.current_condition[0];
    document.getElementById('temp').textContent = c.temp_F + '°F';
    document.getElementById('cond').textContent = c.weatherDesc[0].value;
  });
</script>
</body>
</html>
HTML
EOF

# Done! Live at https://la-weather.workstation.md
```
