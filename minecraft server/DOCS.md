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
- **`enable_command_block`**: Allow command blocks
- **`jvm_opts`**: Additional JVM arguments

## Network Configuration

Default port: **25565/TCP**

- **Local**: Connect to `<ha-ip>:25565`
- **External**: Forward port 25565 to your Home Assistant server

## Data Location

All server data persists at `/addon_config/minecraft-server-java/`:

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
