# Pkl Podman

Pkl templates for configuring [Podman](https://podman.io/) containers using [systemd quadlet files](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html).

## Overview

This project provides Pkl templates to generate Podman quadlet files, which are systemd unit files that describe how to run containers. Quadlets provide a declarative way to manage containers with systemd, offering better integration with the system's service manager.

## Usage

Create a container definition file (e.g., `nginx.pkl`):

```pkl
amends "package://pkl.declix.org/pkl-podman@1.0.0/Container.pkl"

unit = new {
    description = "Nginx Web Server Container"
    after = "network-online.target"
}

container = new {
    image = "docker.io/library/nginx:latest"
    containerName = "nginx"
    
    publishPort {
        "80:80"
        "443:443"
    }
    
    volume {
        "/etc/nginx/conf.d:/etc/nginx/conf.d:ro"
        "/var/www/html:/usr/share/nginx/html:ro"
    }
    
    pull = "missing"
}

service = new {
    restart = "always"
    restartSec = 5
}

install = new {
    wantedBy = "multi-user.target"
}
```

Generate the quadlet file:

```bash
pkl eval nginx.pkl > ~/.config/containers/systemd/nginx.container
systemctl --user daemon-reload
systemctl --user start nginx.service
```

For system-wide containers:

```bash
pkl eval nginx.pkl > /etc/containers/systemd/nginx.container
systemctl daemon-reload
systemctl start nginx.service
```

## Features

### Container Configuration

- **Image management**: Specify container images with pull policies
- **Networking**: Port publishing, custom networks, DNS configuration
- **Storage**: Volume mounts, tmpfs, secrets
- **Environment**: Environment variables and files
- **Health checks**: Built-in health check support
- **Resource limits**: CPU and memory constraints
- **Security**: Capabilities, read-only rootfs, user namespaces

### Systemd Integration

- **Service management**: Restart policies, dependencies, timeouts
- **Resource control**: CPU and memory limits via systemd
- **Logging**: Integration with journald
- **Dependencies**: Proper ordering with other systemd units

## Volumes

Define Podman volumes using volume quadlet files:

```pkl
amends "package://pkl.declix.org/pkl-podman@1.0.0/Volume.pkl"

unit = new {
    description = "PostgreSQL Data Volume"
    after = "local-fs.target"
}

volume = new {
    volumeName = "postgres-data"
    driver = "local"
    
    label {
        ["app"] = "postgres"
        ["purpose"] = "database-storage"
    }
}

install = new {
    wantedBy = "multi-user.target"
}
```

Generate and use the volume:

```bash
pkl eval postgres-volume.pkl > ~/.config/containers/systemd/postgres-data.volume
systemctl --user daemon-reload
systemctl --user start postgres-data.service
```

## Examples

See the `examples/` directory for configurations:

**Container quadlets:**
- `nginx.pkl` - Web server with health checks
- `postgres.pkl` - Database with persistent storage
- `redis.pkl` - Cache with security hardening

**Volume quadlets:**
- `postgres-volume.pkl` - Database storage volume
- `shared-volume.pkl` - Shared application volume with tmpfs

## Quadlet File Locations

Podman looks for quadlet files in these locations:

**User containers:**
- `$HOME/.config/containers/systemd/`
- `/etc/containers/systemd/users/$(UID)/`

**System containers:**
- `/etc/containers/systemd/`
- `/usr/share/containers/systemd/`

## Dependencies

This project depends on:
- [pkl-systemd](https://github.com/declix/pkl-systemd) for base systemd unit definitions

## Quadlet Types

This package supports two types of Podman quadlets:

### Container Quadlets (.container)

Generated quadlet files can contain multiple sections:

- `[Unit]` - Standard systemd unit metadata
- `[Container]` - Podman-specific container configuration
- `[Service]` - Optional systemd service configuration
- `[Install]` - Systemd installation directives

The `[Container]` section is translated by Podman into appropriate `podman run` arguments when the service starts.

### Volume Quadlets (.volume)

Volume quadlet files define persistent storage:

- `[Unit]` - Standard systemd unit metadata
- `[Volume]` - Podman volume configuration
- `[Install]` - Systemd installation directives

Volume services ensure the volume exists before containers that depend on it start.

## Testing

```bash
pkl test
```

## License

MIT