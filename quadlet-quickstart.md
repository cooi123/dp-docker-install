# DataPower Container with Quadlet Quick Start Guide

Quadlet allows you to run Podman containers as systemd services, providing automatic startup, restart policies, and better integration with system management.

> **Note:** This guide assumes you've already completed the prerequisites from the [main quickstart guide](quickstart.md), including Podman installation, registry login, and creating persistent volumes.

## Prerequisites

Verify Quadlet is available:
```bash
podman --version
/usr/libexec/podman/quadlet
```

## Step 1: Configure the Container File

1. **Copy the sample container file** to the appropriate Quadlet directory:

   **For system-wide service** (requires root):
   ```bash
   sudo cp idg.container.sample /etc/containers/systemd/idg.container
   ```

   **For user service** (no root required):
   ```bash
   mkdir -p ~/.config/containers/systemd
   cp idg.container.sample ~/.config/containers/systemd/idg.container
   ```

2. **Edit the container file** to update volume paths if needed:
   ```bash
   # For system service
   sudo nano /etc/containers/systemd/idg.container
   
   # For user service
   nano ~/.config/containers/systemd/idg.container
   ```

   Update the `Volume=` lines to point to your actual directory paths. The `%d` variable will be replaced with the directory containing the `.container` file.

3. **Test the container file configuration** (recommended before deploying):
   ```bash
   # For system service
   /usr/libexec/podman/quadlet --dryrun /etc/containers/systemd/idg.container
   
   # For user service
   /usr/libexec/podman/quadlet --dryrun ~/.config/containers/systemd/idg.container
   ```

   This will validate your configuration and show you the generated systemd unit file without actually creating it.

## Step 2: Reload systemd and Start Service

**For system service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now idg.service
```

**For user service:**
```bash
systemctl --user daemon-reload
systemctl --user enable --now idg.service
```

The service will now start automatically on boot and restart if the container stops.

## Managing the Service

**Common commands:**

```bash
# System service (add sudo)
systemctl start idg.service
systemctl stop idg.service
systemctl restart idg.service
systemctl status idg.service
systemctl enable idg.service
systemctl disable idg.service

# User service (add --user)
systemctl --user start idg.service
systemctl --user stop idg.service
systemctl --user restart idg.service
systemctl --user status idg.service
systemctl --user enable idg.service
systemctl --user disable idg.service
```

## Viewing Logs

**Important:** `journalctl` shows systemd service logs (service start/stop events, errors), but **not** the container's application output.

**View container application logs:**
```bash
podman logs idg
podman logs -f idg  # Follow in real-time
```

**View systemd service logs:**
```bash
journalctl -u idg.service  # System service
journalctl --user -u idg.service  # User service
```

**Optional:** To send container logs to journalctl, add `LogDriver=journald` to the `[Container]` section and restart the service.

## Updating the Container

After editing the container file or pulling a new image:

```bash
# System service
sudo systemctl daemon-reload
sudo systemctl restart idg.service

# User service
systemctl --user daemon-reload
systemctl --user restart idg.service
```

## Links

- [Podman Quadlet Documentation](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
