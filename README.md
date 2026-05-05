# share

Share any terminal command to the public internet in one command — no account, no config.

Built on [ttyd](https://github.com/tsl0922/ttyd) (browser-based terminal) and [Cloudflare Quick Tunnels](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/do-more-with-tunnels/trycloudflare/) (zero-config HTTPS tunnel).

```
$ share claude --dangerously-skip-permissions

⏳ Starting ttyd...
⏳ Creating tunnel on port 58291...

✓ Tunnel is up!
  Job ID : 17468321041234
  Command: claude --dangerously-skip-permissions
  Port   : 58291 (local)
  URL    : https://abc-example.trycloudflare.com

  Stop with: share stop 17468321041234
```

---

## How it works

```
  your machine                  Cloudflare edge          anyone on the internet
┌─────────────────┐            ┌──────────────┐         ┌────────────────────┐
│                 │            │              │         │                    │
│  your command   │            │  trycloudfl  │  HTTPS  │  browser opens     │
│       │         │  WebSocket │  are.com     │◄───────►│  terminal in tab   │
│       ▼         │◄──────────►│  (random     │         │                    │
│     ttyd        │            │   subdomain) │         └────────────────────┘
│  (localhost:N)  │            │              │
│       │         │            └──────────────┘
│  cloudflared   ─┘
│                 │
└─────────────────┘
```

1. **ttyd** wraps your command in a web terminal and listens on a random local port (`-p 0`)
2. **cloudflared** creates an outbound-only tunnel from that port to a random `*.trycloudflare.com` subdomain — no firewall rules, no port forwarding needed
3. The URL is valid as long as the `share` job is running; it disappears the moment you stop it

---

## Install

### One-liner

```bash
curl -fsSL https://raw.githubusercontent.com/qiaodeli111/share2net/main/share | bash -s -- install
```

This will:
- Install `ttyd` and `cloudflared` (if not already present)
- Copy the script to `~/bin/share`
- Add `~/bin` to your `PATH` in your shell rc file

### Platform support

| Platform | ttyd install method | cloudflared install method |
|---|---|---|
| macOS | `brew install` | `brew install` |
| Linux | Static binary from GitHub releases | Static binary from GitHub releases |

> Homebrew on macOS is required. Install it at [brew.sh](https://brew.sh) if you don't have it.

### Manual install

```bash
# Download the script
curl -fsSL https://raw.githubusercontent.com/qiaodeli111/share2net/main/share -o ~/bin/share
chmod +x ~/bin/share

# Install dependencies
share install
```

---

## Usage

### Share any command

```bash
# Share a tmux session
share tmux

# Share a bash shell
share bash

# Share claude (AI coding agent)
share claude --dangerously-skip-permissions

# Any command with arguments works
share htop
share python3 app.py
```

### Manage jobs

```bash
# List all running jobs
share list

# Stop a specific job
share stop <job_id>
```

### Example session

```
$ share tmux

⏳ Starting ttyd...
⏳ Creating tunnel on port 61847...

✓ Tunnel is up!
  Job ID : 17468321045678
  Command: tmux
  Port   : 61847 (local)
  URL    : https://xyz-tunnel.trycloudflare.com

  Stop with: share stop 17468321045678

$ share list

  17468321045678  :61847  cmd=tmux  url=https://xyz-tunnel.trycloudflare.com  [running]

$ share stop 17468321045678

  ✓ Killed ttyd (pid=45231)
  ✓ Killed cloudflared (pid=45232)
  Job 17468321045678 stopped
```

---

## Notes

**Security**
- The URL is publicly accessible to anyone who has it. Do not share sensitive sessions without additional authentication.
- Use `share stop` as soon as the session is no longer needed.
- Cloudflare Quick Tunnels are intended for development and personal use, not production.

**Limitations (Cloudflare Quick Tunnels)**
- Maximum 200 concurrent in-flight requests
- No SLA or uptime guarantee — Cloudflare uses Quick Tunnels to test new features
- URL changes every time you start a new job
- Does not support Server-Sent Events (SSE)

**URL lifetime**
The tunnel URL is only valid while the `share` job is running. There is no time limit — the URL stays alive until you run `share stop <job_id>` or the process is killed.

**Job state**
Job metadata (PID files, URLs) is stored in `~/.local/share/ttyd-share/`. If a process crashes without `share stop`, you can clean up with:

```bash
share list        # see dead jobs
rm -rf ~/.local/share/ttyd-share/<job_id>
```

---

## Requirements

- macOS or Linux
- [Homebrew](https://brew.sh) (macOS only)
- `curl`, `sudo` (Linux only, for binary install)
- Bash 4+

---

## License

MIT
