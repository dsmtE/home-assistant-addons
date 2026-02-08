# Minecraft Server (Java Edition)

Run a Minecraft Java Edition server on Home Assistant OS with support for vanilla, modded, and plugin-based gameplay.

## Features

- Multiple server types: Vanilla, Paper, Spigot, Purpur, Forge, Fabric, Quilt, Modrinth Modpacks
- Easy configuration through Home Assistant UI
- Persistent world data in `/addon_config/minecraft-server-java`

## Quick Start

1. Set `eula: true` to accept the [Minecraft EULA](https://aka.ms/MinecraftEULA)
2. Configure memory settings for your hardware
3. Start the add-on
4. Connect to `<your-ha-ip>:25565`

## Basic Configuration

```yaml
eula: true
server_type: VANILLA
minecraft_version: LATEST
memory_max: 4096
```

## Server Types

| Type | Description |
|------|-------------|
| VANILLA | Standard Minecraft server (no mods/plugins) |
| PAPER | High-performance server with plugin support (recommended) |
| FORGE/FABRIC/QUILT | Mod loaders - place mods in `/addon_config/minecraft-server-java/mods/` |
| MODRINTH | Install complete modpacks from Modrinth |

## Memory Recommendations

| RAM | Use Case |
|-----|----------|
| 2GB | Small vanilla servers (3-5 players) |
| 4GB | Medium servers or light modpacks (10-15 players) |
| 6GB+ | Large servers or heavy modpacks (20+ players) |


## Documentation

See the **Documentation** tab for all configuration options.

## Credits

Based on [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)

Inspired by existing Home Assistant addons:
- [williamcorsel/hamc-server-java](https://github.com/williamcorsel/hassio-addons/tree/main/hamc-server-java)
- [cyclemat/minecraft_vanilla_server](https://github.com/cyclemat/Home-Assistant-Gameservers-ADDONS/tree/main/minecraft_vanilla_server)
- [hassio-addons/addon-ubuntu-base](https://github.com/hassio-addons/addon-ubuntu-base) - S6-overlay and Bashio installation patterns
