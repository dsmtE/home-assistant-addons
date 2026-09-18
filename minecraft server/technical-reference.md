# Minecraft Home Assistant Addon - Technical Reference

Key technical decisions and considerations for building this Minecraft Java Edition addon for Home Assistant.

## Project Stack

- **Base Image**: `itzg/minecraft-server:2026.9.1-java21` (pinned; `latest` currently also == Java 25 but moves without notice)
- **Process Supervisor**: S6-Overlay
- **Config Library**: Bashio

## S6-Overlay Configuration

**Critical**: Must install **all 4 components**, not just 2:
1. `s6-overlay-noarch.tar.xz`
2. `s6-overlay-${S6_ARCH}.tar.xz` (x86_64 for amd64, aarch64 for aarch64)
3. `s6-overlay-symlinks-noarch.tar.xz` ⚠️
4. `s6-overlay-symlinks-arch.tar.xz` ⚠️

**Why symlinks matter**: Without them, `with-contenv` and other utilities don't work, causing silent failures.

**Required environment variables**:
- `S6_BEHAVIOUR_IF_STAGE2_FAILS=2`
- `S6_CMD_WAIT_FOR_SERVICES=1`
- `S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0`

**Prerequisites**: Install `xz-utils` before extracting `.tar.xz` files.

## Service Management: `/etc/services.d/` vs `s6-rc.d`

### Legacy Pattern: `/etc/services.d/` (Chosen)

**Structure**:
```
/etc/services.d/minecraft/
├── run        # Startup script
└── finish     # Shutdown handler
```

**How it works**:
- s6 scans `/etc/services.d/` at startup
- Executes `run` script for each service
- On exit, calls `finish` script with exit code
- Simple one-to-one script mapping

**Advantages**:
- Simpler for single-service addons
- Minimal boilerplate
- Well-documented in Home Assistant examples
- Direct shell script execution

**Disadvantages**:
- Less sophisticated dependency management
- No atomic service state changes
- Limited service definitions

**When to use**: Single-service addons (like this Minecraft addon)

### Modern Pattern: `s6-rc.d` (Alternative)

**Structure**:
```
/etc/s6-rc.d/
├── minecraft/
│   ├── run          # Service executable
│   ├── type         # File with "longrun" or "oneshot"
│   ├── dependencies # Optional: list of dependencies
│   └── pipeline     # Optional: chained services
└── .s6-rc-compile/
    └── ...          # Compiled service configuration
```

**How it works**:
- Configuration files describe what each service does
- `s6-rc-compile` compiles configuration to optimized format
- Supports atomic state changes and dependencies
- Complex graphs of related services

**Advantages**:
- Better for multi-service addons
- Atomic service state changes
- Explicit dependencies between services
- Service groups and pipelines

**Disadvantages**:
- More boilerplate required
- Requires compilation step
- Overkill for simple addons
- More complex debugging

**When to use**: Multi-service addons (e.g., database + API + worker), complex initialization sequences

### Addon Specifics

This addon uses `/etc/services.d/` because:
1. Only one service (Minecraft server)
2. No inter-service dependencies
3. Home Assistant addon examples use this pattern
4. Simpler to maintain and debug

**Run script essentials**:
- Use `set -e` to exit on errors
- End with `exec /start` to replace shell process
- Export uppercase variables for itzg compatibility

## Bashio Integration

**Key functions**:
- `bashio::config 'option'` - Read config
- `bashio::log.{info|error|warning}` - Logging
- `bashio::var.has_value` - Check if set
- `bashio::exit.nok` - Exit with error

**Important**: Always use bashio logging, never raw `echo`

## Critical

1. **S6-Overlay Symlinks**: Install all 4 components (most common failure point)
2. **xz-utils**: Required before extracting s6-overlay
3. **Script Shebang**: Use `#!/usr/bin/with-contenv bashio` not `#!/bin/bash`
4. **Script Permissions**: `chmod +x` on run and finish scripts
5. **Exit Codes**: finish script expects 0 or 256 for normal exits
6. **Image Tags**: Use proper Docker tags (e.g. `2026.9.1-java21`), not addon version numbers; pin the base image in `build.yaml` for reproducible builds
7. **Environment Variables**: itzg expects UPPERCASE (TYPE, VERSION, MEMORY)
8. **Optional Config**: Don't export undefined env vars, let base image use defaults
9. **Data Mapping**: Dual mounts (as in hamc-server-java): `addon_config:rw` → `/data` (browseable at `addons_config/minecraft-server-java/` on host) and `data` → `/hassio_data`
10. **IPv6**: Export `PREFER_IPV6=true` to get a dual-stack `[::]` listener (IPv6 + IPv4); without it the JVM wildcard bind is IPv4-only

## Configuration Patterns

**Optional fields**:
```yaml
schema:
  modrinth_modpack: str?
  modrinth_version_type: list(release|beta|alpha)?
```

**Note**: Home Assistant doesn't support conditional field visibility.

## Dockerfile Best Practices

- **Shell**: `SHELL ["/bin/bash", "-o", "pipefail", "-c"]` for safer builds
- **Packages**: Single RUN layer with `--no-install-recommends`
- **Architecture**: Map `amd64` → `x86_64`, `aarch64` → `aarch64` for s6-overlay
- **Cleanup**: Remove `/tmp/*`, `/var/cache/*`, `/var/log/*`, `/var/lib/apt/lists/*`

## AppArmor Essentials

Required permissions:
- `/data/**` - Server data persistence
- `/usr/lib/jvm/**` - Java runtime
- `/etc/services.d/**` - S6 service scripts
- `/run/**` - Runtime files
- `capability`, `file` - Basic capabilities
- `signal (send) set=(kill,term,int,hup,cont)` - Process signals

## Data Persistence

**config.yaml** (same dual mapping as [hamc-server-java](https://github.com/williamcorsel/hassio-addons/tree/main/hamc-server-java)):
```yaml
map:
  - type: data
    read_only: false
    path: /hassio_data   # data share → extra data volume
  - type: addon_config
    read_only: false
    path: /data          # addon_config share → itzg working dir (browseable from HA)
```
## Modrinth Integration

**Environment variables**:
- `MODRINTH_MODPACK` - Required when TYPE=MODRINTH
- `MODRINTH_MODPACK_VERSION_TYPE` - Optional (release/beta/alpha)
- `MODRINTH_LOADER` - Optional (forge/fabric/quilt)

**Pattern**: Only export if user provides value, let itzg use defaults otherwise.

## Server Types

| Type | Description |
|------|-------------|
| VANILLA | Mojang official |
| PAPER | High-performance (recommended) |
| FORGE/FABRIC/QUILT | Mod loaders |
| MODRINTH | Complete modpacks |

## Build Configuration

Centralize versions in `build.yaml`:
```yaml
args:
  BASHIO_VERSION: v0.17.5
  S6_OVERLAY_VERSION: 3.2.2.0
  BUILD_FROM: "itzg/minecraft-server:latest"
```

**Pinning the base image**: `latest` moves on every itzg release (and currently == Java 25). Pin a dated tag for reproducible builds: `itzg/minecraft-server:2026.9.0-java25` (amd64 + arm64). **Java 25** is the default for modern servers/modpacks; if you ever run an older Minecraft version (< 1.20.5) or an old Forge build that complains, fall back to `2026.9.0-java21`. Note: the Minecraft server *version* is selected at runtime via `minecraft_version`, independent of the image tag.

## IPv6 & Remote Access (CGNAT workaround)

### Why IPv6

Behind carrier-grade NAT (CGNAT) there is no public IPv4 and no port forwarding, so IPv4-based hosting is impossible. IPv6 gives every device its own public address and removes the NAT — inbound traffic is **routed**, not forwarded. The remaining gatekeeper is a box **firewall rule** (blocked inbound by default).

### Address anatomy

```
2a02:842a:dca:1701 : 7a22:6831:644d:1624
\_________________/   \_________________/
    PREFIX (64 bits)       INTERFACE-ID (64 bits)
    given by the ISP      chosen by the device
```

### The three "stabilities"

| # | Thing | Who controls it | Fix |
|---|-------|-----------------|-----|
| 1 | Interface-ID (host suffix) | Box / device | DHCPv6 **static lease** on the box (SFR: "Baux statiques"), or `ipv6.addr-gen-mode=eui64` on HAOS; keep HA's IPv6 on **automatic** so the suffix survives prefix rotation |
| 2 | Prefix | ISP | Accept it may rotate → use **dynamic DNS** with AAAA so friends use a hostname |
| 3 | Firewall rule | Box | Ensure the rule matches the device (by static lease), so it survives prefix changes |

### SFR Box 8 (Fibre) example

1. **LAN → Baux statiques**: enable **DHCPv6** on the box, then add an IPv6 static lease for the Home Assistant device (stable suffix).
2. **Sécurité → Accès → Réseau v6** → **Créer une règle**:
   - name the rule, select the HA device's static IPv6 address,
   - protocol **TCP**, internal & external port **25565**,
   - source access (e.g. "Tous"), enable.
3. Reboot the box so the HA device picks the static lease; HA keeps IPv6 on **automatic** (a manual/static entry in HA breaks if the prefix rotates).

### Home Assistant side

- Keep `host_network: true` — the addon shares the host's real IPv6 address; the supervisor's published-`ports` mechanism is IPv4-only, so it's not usable for IPv6 hosting (and is ignored under host_network anyway).
- `PREFER_IPV6=true` (addon option `prefer_ipv6`) → Java binds a dual-stack `[::]` socket, so LAN IPv4 keeps working. Without it, the JVM wildcard bind is IPv4-only and the server cannot be reached over IPv6.

### Dynamic DNS (hostname for friends)

DuckDNS maps a hostname (e.g. `yourserver.duckdns.org`) to the HA host's **global** IPv6
address (AAAA record) so friends never have to chase a rotating prefix.

**⚠️ Leading/trailing spaces break everything.** The DuckDNS addon (and the manual update
URL) fail silently if any config field contains a stray space — the update URL becomes
malformed, the record is never updated, and the addon still shows as "running". The usual
culprits are `token` and `ipv6` right after a copy/paste. Trim spaces (and newlines) out
of **every** field, and after editing options click **Restart** — saving options is not
applying them.

**What to think about:**
- "Running" ≠ updating. Verify the record at duckdns.org (DNS Records → AAAA) or with
  `nslookup -type=AAAA yourserver.duckdns.org`; don't trust the addon log for DNS state.
- Manual update is the cleanest test/debug tool:

  ```
  curl -sL "https://duckdns.org/update?domains=<yourdomain>&token=<yourtoken>&ipv6=<your-ipv6>"
  ```

  - Reply `OK` → record updated.
  - Reply `KO` → token/domain mismatch — re-check for stray spaces in those fields.
  - **Gotcha**: the `www.` host redirects (301) — use `-L` and no `www` or you get "Moved Permanently".
- Detect the host's current global address with `curl -6 -s https://api6.ipify.org`.
- If the addon ever fails to update: fall back to manual `curl` runs, or a tiny loop in a
  `host_network` addon that fetches `api6.ipify.org` and calls DuckDNS itself. DuckDNS's
  own auto-detect / `dnsip` integration can't be relied on as an updater.
- `lets_encrypt`/`accept_terms` only matters if you serve HTTPS; leave it off otherwise.
  Dynamic DNS only publishes a DNS record you control — respect the service ToS.

### End-to-end setup order

1. Give the host a stable interface-ID (DHCPv6 static lease on the box).
2. Enable `host_network` + `prefer_ipv6` in the addon (dual-stack `[::]` listener; LAN IPv4 keeps working).
3. Open inbound **TCP 25565** on the box, matched to the device's static IPv6 (so it survives prefix changes).
4. Point DuckDNS at the host's **global** IPv6 and verify the AAAA.
5. From an outside network: `ping6 yourserver.duckdns.org`, then join via the hostname (or `[IPv6]:25565`).
6. Lock the door: fill the addon `whitelist` now that the server is internet-exposed.
7. Keep ZeroTier/Tailscale as the IPv4 fallback for friends without native IPv6.

### Verification

- From a friend's network: `ping6 yourserver.duckdns.org` should answer.
- Join with `yourserver.duckdns.org` or `[2a02:...:1624]:25565`.
- Keep ZeroTier/Tailscale for friends without native IPv6 (this repo's fallback).

## Credits

- Base: [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)
- Inspired by: [williamcorsel/hamc-server-java](https://github.com/williamcorsel/hassio-addons), [cyclemat/minecraft_vanilla_server](https://github.com/cyclemat/Home-Assistant-Gameservers-ADDONS)

---

*Updated: September 18, 2026*
