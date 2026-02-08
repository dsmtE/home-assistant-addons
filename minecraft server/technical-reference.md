# Minecraft Home Assistant Addon - Technical Reference

Key technical decisions and considerations for building this Minecraft Java Edition addon for Home Assistant.

## Project Stack

- **Base Image**: `itzg/minecraft-server:latest`
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
6. **Image Tags**: Use proper Docker tags, not addon version numbers
7. **Environment Variables**: itzg expects UPPERCASE (TYPE, VERSION, MEMORY)
8. **Optional Config**: Don't export undefined env vars, let base image use defaults
9. **Data Mapping**: Use `addon_config:rw` in config.yaml maps to `/data` in container

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

**config.yaml**: `map: [addon_config:rw]`

**Path mapping**:
- Host: `/addon_config/minecraft-server-java/`
- Container: `/data/`

**Directory structure**: world/, mods/, config/, server.properties, ops.json, whitelist.json, logs/

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

## Credits

- Base: [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)
- Inspired by: [williamcorsel/hamc-server-java](https://github.com/williamcorsel/hassio-addons), [cyclemat/minecraft_vanilla_server](https://github.com/cyclemat/Home-Assistant-Gameservers-ADDONS)

---

*Updated: February 8, 2026*
