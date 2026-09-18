## 1.0.4

- Added `prefer_ipv6` option (dual-stack IPv6/IPv4 binding) for hosting over IPv6 (e.g. behind CGNAT)
- Added `whitelist` option for public servers
- Pinned base image to `itzg/minecraft-server:2026.9.0-java25` for reproducible builds
- Removed redundant port mapping (ignored under `host_network`)
- Added `memory_min <= memory_max` validation, cleaned EULA logs

## 1.0.3

- Map `/data` to `/addons_config` folder for easier configuration management of the server files

## 1.0.2

- Added `host_network: true` to config for proper network connectivity
- Only install needed additional packages (xz-utils)

## 1.0.1

- Added Modrinth modpack support
- Simplified documentation
- Cleaned up AppArmor profile

## 1.0.0

- Initial release
- Support for multiple server types (Vanilla, Paper, Forge, Fabric, etc.)
- Configurable memory allocation
- Based on itzg/docker-minecraft-server
