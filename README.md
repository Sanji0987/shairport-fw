# shairport-fw (firewall wrapper)

A setup script I made for myself to run [Shairport Sync](https://github.com/mikebrady/shairport-sync) (AirPlay audio receiver) on Arch/CachyOS with PipeWire, without leaving firewall ports open 24/7 or passwords in plaintext at rest.

## What it does

- Installs and configures shairport-sync with PipeWire output and soxr interpolation
- Stores the AirPlay password in a root-only file (`/etc/shairport-sync-password`, mode 600)
- Password is injected into the config only while the service is running, wiped on stop
- Firewall rules (ufw) open TCP 5000 and UDP 6001-6010 to **LAN only** on start, close on stop
- The firewall helper script is root-owned at `/usr/local/bin` to prevent privilege escalation
- Single `pkexec` password prompt per start/stop

## Requirements

- Arch-based system (uses pacman)
- PipeWire running
- An AirPlay device (iPhone, iPad, Mac) on the same network

## Install

```
git clone <this repo>
cd shairport-fw
chmod +x setup-shairport
./setup-shairport
```

It will ask for a device name and password, then set up everything.

## Usage

```
shareport start   # opens ports, injects password, starts shairport-sync
shareport stop    # stops shairport-sync, wipes password, closes ports
```

## What gets created

| File | Owner | Purpose |
|------|-------|---------|
| `/etc/shairport-sync.conf` | root | Shairport Sync config |
| `/etc/shairport-sync-password` | root (600) | AirPlay password, not readable by users |
| `/usr/local/bin/shairport-sync-fw` | root (755) | Firewall + password inject/wipe helper |
| `~/.local/bin/shareport` | user | Start/stop wrapper |
| `~/.config/systemd/user/shairport-sync.service` | user | Systemd user service |

## Security

- Ports are only open while the service is running
- Only private LAN subnets can connect (192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12)
- Password is not in the config file at rest
- The root helper uses absolute paths to prevent PATH hijacking
- Input is sanitized before being written to config files
