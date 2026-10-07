# Server Profile — Template
#
# Instructions:
#   1. Copy this file to `server-profile.md` (same directory or your GEMINI.md).
#   2. Replace all `<PLACEHOLDER>` values with your actual server details.
#   3. Add `server-profile.md` to .gitignore — it contains credentials.
#   4. Duplicate the "Server Profile" section for each server you manage.
#   5. Remove the example block at the bottom after you're done.

---

## Server Profile: <SERVER_NICKNAME>

### Connection Details

| Key | Value |
|---|---|
| **Host** | `<IP_ADDRESS_OR_HOSTNAME>` |
| **Connection** | `<Direct / OpenVPN / WireGuard / Tailscale / SSH Tunnel>` |
| **Environment Tier** | `<development / staging / production>` |
| **SSH User** | `<SSH_USERNAME>` |
| **SSH Key** | `<PATH_TO_SSH_PRIVATE_KEY>` (e.g. `~/.ssh/id_ed25519`) |
| **SSH Port** | `<PORT>` (default: `22`) |
| **Sudo Password** | `<SUDO_PASSWORD>` |
| **OS / Distro** | `<Ubuntu 22.04 / Debian 12 / Rocky 9 / etc.>` |

### Jump Host (optional, delete if not needed)

If this server is only reachable via a jump/bastion host:

| Key | Value |
|---|---|
| **Jump Host** | `<JUMP_HOST_IP_OR_ALIAS>` |
| **Jump User** | `<JUMP_USER>` |
| **Jump Key** | `<PATH_TO_JUMP_KEY>` |

### Active Stacks / Services

<!-- List all services so the AI has operational context -->

| Service Name | Port(s) | Type | Database | Health Check | Notes |
|---|---|---|---|---|---|
| `<SERVICE_1>` | `<PORT>` | `<systemd / docker / pm2 / manual>` | `<DB_NAME or —>` | `<curl URL or —>` | `<NOTES>` |
| `<SERVICE_2>` | `<PORT>` | `<systemd / docker / pm2 / manual>` | `<DB_NAME or —>` | `<curl URL or —>` | `<NOTES>` |

### Database Credentials (optional)

| Engine | Host | Port | User | Password | Default DB |
|---|---|---|---|---|---|
| `<MySQL / MariaDB / PostgreSQL / MongoDB>` | `localhost` | `<PORT>` | `<DB_USER>` | `<DB_PASSWORD>` | `<DB_NAME>` |

### Tunnel / Reverse Proxy (optional)

| Key | Value |
|---|---|
| **Type** | `<cloudflared / nginx / caddy / traefik / none>` |
| **Domain Pattern** | `<*.example.com>` |
| **SSL** | `<Let's Encrypt / Cloudflare Origin / self-signed / none>` |
| **Systemd Service** | `<SERVICE_NAME>` |
| **Config Path** | `<PATH_TO_CONFIG>` |

### Cron Jobs / Scheduled Tasks (optional)

| Schedule | Command | Purpose |
|---|---|---|
| `<CRON_EXPRESSION>` | `<COMMAND>` | `<DESCRIPTION>` |

### Backup Policy (optional)

| What | Method | Schedule | Destination |
|---|---|---|---|
| `<Database>` | `<mysqldump / pg_dump>` | `<Daily 2AM / etc.>` | `<PATH_OR_REMOTE>` |
| `<Files>` | `<rsync / tar>` | `<Weekly / etc.>` | `<PATH_OR_REMOTE>` |

### Notes / Special Instructions

<!--
  Add any server-specific instructions the AI should know about:
  - "Always restart nginx after editing /etc/nginx/sites-enabled/*"
  - "This server shares MariaDB with Server B, do not restart MariaDB without checking Server B first"
  - "Deploy via git pull on /var/www/app, then run php artisan migrate"
-->

- `<NOTE_1>`
- `<NOTE_2>`

---

<!--
  ╔══════════════════════════════════════════════════════════════╗
  ║  EXAMPLE — Delete this block after filling in your own      ║
  ╚══════════════════════════════════════════════════════════════╝

  ## Server Profile: Diskominfo Dev Server

  ### Connection Details

  | Key | Value |
  |---|---|
  | **Host** | `192.168.100.64` |
  | **Connection** | OpenVPN on Windows |
  | **Environment Tier** | `development` |
  | **SSH User** | `devface` |
  | **SSH Key** | `C:/Users/hazra/.ssh/id_ed25519` |
  | **SSH Port** | `22` |
  | **Sudo Password** | `prabsky##` |
  | **OS / Distro** | Ubuntu 22.04 LTS |

  ### Active Stacks / Services

  | Service Name             | Port(s)        | Type    | Database         | Health Check                          | Notes                    |
  |--------------------------|----------------|---------|------------------|---------------------------------------|--------------------------|
  | Face Recognition Gateway | 80, 443, 5050  | docker  | —                | `curl -s http://localhost:5050/health` | Main recognition service |
  | Temanku Unik             | 8080, 8081     | docker  | MariaDB 3306     | `curl -s http://localhost:8080`        | Community app            |
  | Matriks BaseFW           | 8082           | docker  | `basefw_matriks` | `curl -s http://localhost:8082`        | Framework base           |

  ### Database Credentials

  | Engine  | Host      | Port | User   | Password    | Default DB       |
  |---------|-----------|------|--------|-------------|------------------|
  | MariaDB | localhost | 3306 | root   | secret123   | basefw_matriks   |

  ### Tunnel / Reverse Proxy

  | Key | Value |
  |---|---|
  | **Type** | `cloudflared` |
  | **Domain Pattern** | `*.hazra.biz.id` |
  | **SSL** | Cloudflare Origin |
  | **Systemd Service** | `cloudflared.service` |
  | **Config Path** | `/etc/cloudflared/config.yml` |

  ### Notes / Special Instructions

  - Always check `docker ps` before restarting any container
  - Cloudflared tunnel config is at `/etc/cloudflared/config.yml`
  - MariaDB is shared between Temanku Unik and Matriks BaseFW

-->
