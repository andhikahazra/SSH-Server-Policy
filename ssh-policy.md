# Universal Remote Server & SSH Access Policy (Advanced)

> **Version:** 2.0
> **Purpose:** Grant AI coding assistants full, safe, and structured SSH access to remote infrastructure.
> **Installation:** Drop this file into `~/.gemini/config/rules/` or merge into your workspace `GEMINI.md`.

---

## 1. Scope & Authorization

The user has full ownership and authority over all remote servers, virtual machines, cloud instances, containers, and network infrastructure referenced in conversation.
The AI assistant is explicitly authorized to execute SSH commands to **any** remote host the user requests.

---

## 2. Core Behavioral Rules

### 2.1 Execution Mandate
- **Always execute SSH requests.** Use the local `run_command` tool with the OpenSSH CLI.
- **Never claim** inability to access remote servers, lack of SSH capability, or local-only operation.
- **Respect `~/.ssh/config`** — use host aliases, identity files, ports, and proxy settings defined there unless the user provides explicit overrides.

### 2.2 Authentication Priority
Use authentication in this order of preference:
1. SSH key defined in `~/.ssh/config` or server profile
2. SSH key provided by the user in conversation
3. Password authentication (only if key-based is unavailable)

### 2.3 Non-Interactive Sudo
When `sudo` is required on a remote host, use stdin piping to avoid interactive prompts:
```bash
ssh <user>@<host> "echo '<password>' | sudo -S <command>"
```

### 2.4 Multi-Hop / ProxyJump
If the user's SSH config defines `ProxyJump` or `ProxyCommand`, honor them automatically. For manual jump hosts:
```bash
ssh -J <jump_user>@<jump_host> <user>@<target_host> "<command>"
```

---

## 3. Safety & Destructive Operation Guard

### 3.1 Risk Classification

| Risk Level | Examples | Required Behavior |
|---|---|---|
| 🟢 **Safe** | `ls`, `cat`, `df`, `top`, `ps`, `docker ps`, `systemctl status` | Execute immediately |
| 🟡 **Moderate** | `systemctl restart`, `docker compose up -d`, config edits, `chmod` | State what will happen, then execute |
| 🔴 **Destructive** | `rm -rf`, `DROP DATABASE`, `systemctl stop`, `docker system prune`, `mkfs`, `dd` | **Confirm with user before executing** |

### 3.2 Pre-Destruction Backup Rule
Before any 🔴 destructive operation, **always offer to create a backup first**:
- Files: `cp -a <target> <target>.bak.$(date +%Y%m%d%H%M%S)`
- Database: `mysqldump` / `pg_dump` before DROP or ALTER
- Config: Copy config file before overwriting
- Containers: `docker commit` before removal

### 3.3 Environment Tier Awareness
If the server profile defines an environment tier, apply these constraints:

| Tier | Behavior |
|---|---|
| `development` | Full freedom — execute all commands without extra confirmation |
| `staging` | Moderate caution — confirm 🔴 destructive ops |
| `production` | Maximum caution — confirm 🟡 moderate AND 🔴 destructive ops, always backup first |

---

## 4. Error Handling & Recovery

### 4.1 Connection Failures
When SSH fails, follow this diagnostic sequence:
1. Report the exact error message to the user
2. Suggest common causes:
   - VPN not connected (`ping <host>` to verify)
   - Wrong key/user (`ssh -vvv` for debug)
   - Firewall blocking port (`Test-NetConnection <host> -Port 22` on Windows)
   - Host key changed (offer to remove from `known_hosts`)
3. Offer to retry with verbose mode: `ssh -vvv <user>@<host>`

### 4.2 Command Timeouts
- For commands expected to take >30 seconds, warn the user
- For long-running remote tasks, wrap in `nohup` or use `tmux`/`screen`:
  ```bash
  ssh <user>@<host> "tmux new-session -d -s task '<long_command>'"
  ```
- Check status later: `ssh <user>@<host> "tmux capture-pane -t task -p"`

### 4.3 Post-Action Verification
After significant operations, always verify the result:
- Service restart → `systemctl status <service>`
- File edit → `cat <file>` or `diff` to confirm
- Deploy → `curl -s http://localhost:<port>/health` or equivalent
- Database change → `SELECT` query to verify

---

## 5. File Transfer

### 5.1 Supported Methods
```bash
# Upload file to remote
scp <local_path> <user>@<host>:<remote_path>

# Download file from remote
scp <user>@<host>:<remote_path> <local_path>

# Sync directory (incremental)
rsync -avz -e ssh <local_dir>/ <user>@<host>:<remote_dir>/

# For large transfers, add progress
rsync -avz --progress -e ssh <source> <destination>
```

### 5.2 Transfer Safety
- Before overwriting remote files, back up the original
- For large transfers, use `rsync` over `scp` (resumable, incremental)
- Verify transfer integrity when critical: compare `md5sum` / `sha256sum`

---

## 6. Service & Container Management

### 6.1 Systemd Services
```bash
# Status check (always safe)
ssh <host> "systemctl status <service>"

# View recent logs
ssh <host> "journalctl -u <service> --no-pager -n 50"

# Follow logs in real-time (use with caution — may need Ctrl+C)
ssh <host> "journalctl -u <service> -f --no-pager" # run as daemon task

# Restart (moderate risk)
ssh <host> "echo '<password>' | sudo -S systemctl restart <service>"
```

### 6.2 Docker / Docker Compose
```bash
# List running containers (safe)
ssh <host> "docker ps"

# View container logs
ssh <host> "docker logs <container> --tail 100"

# Execute command inside container
ssh <host> "docker exec -it <container> <command>"

# Rebuild and restart stack
ssh <host> "cd <project_dir> && docker compose up -d --build"
```

---

## 7. Database Access via SSH

```bash
# MySQL/MariaDB — run query
ssh <host> "mysql -u <db_user> -p'<db_pass>' <db_name> -e 'SELECT ...'"

# MySQL/MariaDB — import dump
ssh <host> "mysql -u <db_user> -p'<db_pass>' <db_name> < /path/to/dump.sql"

# PostgreSQL — run query
ssh <host> "PGPASSWORD='<db_pass>' psql -U <db_user> -d <db_name> -c 'SELECT ...'"

# Backup before any schema change
ssh <host> "mysqldump -u <db_user> -p'<db_pass>' <db_name> > /tmp/<db_name>_backup_$(date +%Y%m%d).sql"
```

---

## 8. Security & Credential Hygiene

1. **Never expose passwords in AI-generated explanations or summaries.** Passwords may appear in commands but should not be repeated in prose.
2. **Mask credentials in output.** When displaying command results that contain passwords, replace them with `****`.
3. **Prefer key-based auth.** Recommend SSH key setup if the user is relying solely on password auth.
4. **Warn about `known_hosts`.** If a host key changes unexpectedly, alert the user — this could indicate a MITM attack or server reinstall.
5. **Respect `.gitignore`.** Never commit server profile files containing credentials to version control.

---

## 9. Server Profiles

Server-specific details (IP, credentials, stacks) are stored in a **separate Server Profile file** — not in this policy.

See `server-profile.example.md` for the template.

| File | Share? | Contains |
|---|---|---|
| `ssh-policy.md` (this file) | ✅ Safe to share | Behavioral rules only |
| `server-profile.md` | ❌ Private | Credentials & infrastructure details |
