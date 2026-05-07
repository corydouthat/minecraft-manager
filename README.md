# Minecraft Server Manager

Bash scripts for managing a Minecraft **Bedrock** server and a Minecraft **Java** server on Linux. Both scripts use GNU `screen` for session management and provide auto-restart, update checking, and monitoring.

## Scripts

| Script | Purpose |
|--------|---------|
| `minecraft-bedrock-manager.sh` | Bedrock Dedicated Server (BDS) |
| `minecraft-java-manager.sh` | Java Edition server |

---

## Requirements

- `screen` — session management
- `curl` or `wget` — downloads
- `unzip` — Bedrock archive extraction
- `java` (17+) — Java server only
- `jq` or `python3` — recommended for reliable JSON parsing (required for Java updates)

Install on Fedora/RHEL: `sudo dnf install screen curl unzip jq java-21-openjdk-headless`

---

## Quick Start

### Bedrock

```bash
chmod +x minecraft-bedrock-manager.sh

# First run creates config at ./minecraft-bedrock-manager.conf
# Edit it, then download the server:
./minecraft-bedrock-manager.sh update

# Start monitoring (auto-restart + daily updates at 03:00)
./minecraft-bedrock-manager.sh monitor
```

### Java

```bash
chmod +x minecraft-java-manager.sh

# First run creates config at ./minecraft-java-manager.conf
# Set AUTO_ACCEPT_EULA=true after reading https://www.minecraft.net/en-us/eula
# Then download the server JAR:
./minecraft-java-manager.sh update

# Start monitoring (auto-restart; auto-update is OFF by default)
./minecraft-java-manager.sh monitor
```

---

## Commands

Both scripts share the same core commands:

```
start        - Start the server
stop         - Stop the server gracefully
restart      - Stop then start
update       - Stop, update to latest version, start
check-update - Check if a newer version exists (no action taken)
status       - Show running/stopped status and version
monitor      - Start with auto-restart loop (default action)
version      - Show manager script version
```

### Java: whitelist management

```bash
./minecraft-java-manager.sh whitelist list
./minecraft-java-manager.sh whitelist add PlayerName
./minecraft-java-manager.sh whitelist remove PlayerName
./minecraft-java-manager.sh whitelist on
./minecraft-java-manager.sh whitelist off
```

`add` and `remove` work whether the server is running or not. When the server is offline they call the Mojang API to look up the player's UUID.

---

## Configuration

Copy the `.conf.example` file for your platform and edit it:

```bash
cp minecraft-java-manager.conf.example minecraft-java-manager.conf
# or
cp minecraft-bedrock-manager.conf.example minecraft-bedrock-manager.conf
```

The scripts auto-generate a config with defaults on first run if one isn't present.

### Key Java options

| Setting | Default | Notes |
|---------|---------|-------|
| `SERVER_JAR` | `server.jar` | JAR filename in `SERVER_DIR` |
| `MIN_RAM` / `MAX_RAM` | `2G` / `8G` | Java heap size |
| `AUTO_ACCEPT_EULA` | `false` | Set true after reading the EULA |
| `AUTO_UPDATE` | `false` | See version mismatch note below |

### Key Bedrock options

| Setting | Default | Notes |
|---------|---------|-------|
| `SERVER_BINARY` | `bedrock_server` | Binary name in `SERVER_DIR` |
| `GAMERULES_STRING` | see example | Applied on every start |
| `UPDATE_CHECK_TIME` | `03:00` | Daily update window (24h) |

---

## Java version mismatch

Java Edition clients must run the **exact same version** as the server. When you run `update`, the manager prints a clear warning:

```
*** ALL CLIENTS MUST UPDATE TO VERSION X.X.X ***
```

`AUTO_UPDATE` in the Java config defaults to `false` so you control when updates happen. Use `check-update` to see if a newer version is available without touching the running server.

---

## Security notes

- **Never commit `server.properties`** — it contains `rcon.password` and `management-server-secret`.
- **Never commit `*.conf` files** — they contain local paths and may be extended with secrets.
- **Never commit `whitelist.json` / `ops.json`** — player UUIDs are personal data.
- All of the above are already covered by `.gitignore`.
- Config files are created with `640` permissions (`chmod o-w` enforced at load time).

---

## Running as a service

To survive reboots, add a crontab entry:

```bash
@reboot /path/to/minecraft-java-manager.sh monitor >> /path/to/minecraft-java-manager.log 2>&1 &
```

Or create a systemd unit (recommended for production):

```ini
[Unit]
Description=Minecraft Java Server
After=network.target

[Service]
User=minecraft
ExecStart=/path/to/minecraft-java-manager.sh monitor
Restart=no

[Install]
WantedBy=multi-user.target
```
