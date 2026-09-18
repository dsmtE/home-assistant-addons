# Minecraft Server (Java Edition)

Run a Minecraft Java Edition server with support for mods and plugins directly on Home Assistant OS.

## Features

✅ Multiple server types (Vanilla, Paper, Forge, Fabric, etc.)
✅ Easy configuration through Home Assistant UI

## Quick Start

1. **Accept the EULA**: Set `eula: true` in configuration
2. Configure memory settings based on your hardware
3. Start the add-on
4. Connect to `<your-ha-ip>:25565`

## Configuration Options

### Essential Settings

- **`eula`** (required): Must be `true` to accept Minecraft EULA
- **`server_type`**: VANILLA, PAPER, SPIGOT, PURPUR, FORGE, FABRIC, QUILT, MODRINTH
- **`minecraft_version`**: LATEST or specific version (e.g., 1.20.4)
- **`memory_min`** / **`memory_max`**: Memory allocation in MB
- **`prefer_ipv6`**: Bind the server **dual-stack** (IPv6 + IPv4). Enable this to host externally over IPv6 (e.g. behind CGNAT). LAN IPv4 keeps working.

### Server Properties

- **`server_name`**: MOTD displayed in server list
- **`gamemode`**: survival, creative, adventure, spectator
- **`difficulty`**: peaceful, easy, normal, hard
- **`max_players`**: Maximum concurrent players
- **`pvp`**: Enable/disable player vs player combat
- **`view_distance`**: Render distance in chunks (lower = better performance)

### World Settings

- **`level_name`**: World folder name
- **`level_seed`**: World generation seed (optional)
- **`level_type`**: default, flat, amplified, etc.

### Modrinth Modpacks

To use a Modrinth modpack:

```yaml
server_type: MODRINTH
modrinth_modpack: "modpack-slug-or-id"
modrinth_version_type: release  # optional: release, beta, alpha
modrinth_loader: fabric  # optional: forge, fabric, quilt
```

Find modpack slugs on [Modrinth](https://modrinth.com/modpacks).

### Advanced Options

- **`online_mode`**: Verify player accounts with Mojang (set `false` for offline mode)
- **`whitelist`**: Comma-separated Minecraft usernames allowed to join (empty = no whitelist). Highly recommended when the server is exposed to the internet.
- **`enable_command_block`**: Allow command blocks
- **`jvm_opts`**: Additional JVM arguments

## Network Configuration

Default port: **25565/TCP**

The add-on runs on the **host network**, so it answers directly on your Home Assistant host's IP address(es) — no port mapping or in-LAN forwarding is needed.

- **Local**: Connect to `<ha-ip>:25565`
- **External (IPv6, recommended)**: Enable `prefer_ipv6`, make sure your host has a global IPv6 address, and create an **inbound rule for TCP 25565** on your router/box (with IPv6 there is no NAT/port forwarding, but the box firewall still blocks inbound by default). Players connect with a hostname or `[IPv6-address]:25565`. IPv4 on your LAN keeps working (dual-stack).
- **External (IPv4)**: Requires a public IPv4 + classic port forwarding — not possible behind CGNAT. Use a VPN (ZeroTier/Tailscale) or an IPv6 tunnel for friends without native IPv6.

**TIP — data folder visibility:** Two folders keep the server files out of the container's hidden Docker layers (same layout as [hamc-server-java](https://github.com/williamcorsel/hassio-addons/tree/main/hamc-server-java)):
- `/data` ← mapped from the Home Assistant `addons_config` share → browseable world/mods/config at `addons_config/minecraft-server-java/`
- `/hassio_data` ← mapped from the `data` share → additional data volume

## Data Location

The server operates on `/data`, backed by two shares so nothing is hidden inside the container:

| Container path | Backed by | Browseable at |
|---|---|---|
| `/data` | `addon_config` share | `addons_config/minecraft-server-java/` (File editor / Samba) |
| `/hassio_data` | data share | `data/` (File editor / Samba) |

Contents of `/data` (`addons_config/minecraft-server-java/`):

```
world/              # World save
mods/               # Mod files (Forge/Fabric)
config/             # Mod configurations
server.properties   # Server settings
ops.json            # Operators
whitelist.json      # Whitelist
logs/               # Server logs
```

## Troubleshooting

**Server won't start:**
- Verify `eula: true` is set
- Check available system RAM
- Review addon logs for errors

**Can't connect:**
- Confirm server shows "Done" in logs
- Check firewall/port forwarding
- Verify IP address and port

## Links

- [Minecraft EULA](https://aka.ms/MinecraftEULA)
- [itzg Documentation](https://docker-minecraft-server.readthedocs.io/)
- [Modrinth Modpacks](https://modrinth.com/modpacks)
