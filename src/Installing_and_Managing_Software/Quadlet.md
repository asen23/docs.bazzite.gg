---
authors:
  - "@asen23"

preview:
  alpha: 150
  image: "../img/podman.png"
description: "Podman Icon"
---

## What is Quadlet?

![podman|385x358, 50%](../img/podman.png)

Quadlet are a features of [podman](https://podman.io/) that enable user to run container as [systemd](https://systemd.io/) units. It works by using a declarative syntax like [docker compose](https://docs.docker.com/compose/) but integrates to systemd and use podman as a backend.

### Quick Example:

Create a file called `~/.config/containers/systemd/nginx.container` with content below.
```
[Container]
ContainerName=nginx
Image=docker.io/nginxinc/nginx-unprivileged
PublishPort=8080:8080
```

Save it and run the code below.

```sh
systemctl --user daemon-reload
systemctl --user start nginx
xdg-open localhost:8080
```

![nginx welcome page|1296x400](../img/nginx_welcome.png)

## Use Cases

Quadlet can be used for application packaged as container such as server application. You can find a lot of example of containerized application from [Linux Server](https://docs.linuxserver.io/images/).

## Managing Quadlet

Quadlet can be managed like any other systemd service using below command.

Checking quadlet status
```sh
systemctl --user status nginx
```

Stopping quadlet
```sh
systemctl --user stop nginx
```

You can see more command in [man systemctl](https://man.archlinux.org/man/systemctl.1) or [tldr systemctl](https://tldr.inbrowser.app/pages/linux/systemctl).

### Quadlet file location

You can put your quadlet in these location sorted by priority.
- `$XDG_RUNTIME_DIR/containers/systemd/` - Usually used for temporary quadlet
- `~/.config/containers/systemd/` - Recommended location
- `/etc/containers/systemd/users/$(UID)`
- `/etc/containers/systemd/users/`

### Running Quadlet on Startup

You may want to run your quadlet automatically on startup, but if you tried to enable the unit you will find the command errored out. Thats because the quadlet haven't defined when to autostart. To fix it you can add these lines to your quadlet file. Most of the time `default.target` is what you want but if you need other target you can read on systemd docs.
```
[Install]
WantedBy=default.target
```

For example:
```
[Container]
ContainerName=nginx
Image=docker.io/nginxinc/nginx-unprivileged
PublishPort=8080:8080

[Install]
WantedBy=default.target
```

After that you can run `systemctl --user enable --now nginx`

!!! note

    You can skip the `--now` part if you already started it before.

### Converting Docker Compose to Quadlet Unit

You will find that most of containerized app in the web are built using docker compose. Even the Linux Server that is linked above have all container documented using compose file. So you will need to convert it first before running it as quadlet, fortunately you can use [podlet](https://github.com/containers/podlet) to help converting it.

### Running Rootful Container as Quadlet

While ideally you would run all container using rootless podman, sadly not all container will work with it. If you noticed in the beginning, this guide used nginx-unprivileged rather than the normal nginx, this because it need root to function. To use rootful podman, you will need to use different quadlet path and run using root systemctl (without `--user`).

Rootful Quadlet Path
- `/run/containers/systemd/` - Temporary quadlet
- `/etc/containers/systemd/` - Recommended location
- `/usr/share/containers/systemd/` - Image defined

## Example

### Minecraft Server

Documentation: https://docker-minecraft-server.readthedocs.io/en/latest
Quadlet File:
```
[Container]
Environment=EULA=TRUE
Image=docker.io/itzg/minecraft-server
PublishPort=25565:25565
Volume=/path/to/data:/data:z
```
!!! note

    Use absolute path for volume, e.g `/home/username/minecraft/data`.

### Plex Server

Documentation: https://github.com/plexinc/pms-docker
Quadlet File:
```
[Container]
ContainerName=plex
Environment=TZ=Your/TimeZone
Image=docker.io/plexinc/pms-docker
Network=host
Volume=/path/to/config:/config:z
Volume=/path/to/transcode:/transcode:z
Volume=/path/to/media:/data:z
```
!!! note

    You can find list of timezone [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
!!! note

    Use absolute path for volume, e.g `/home/username/plex/config`.
!!! note

    You can mount multiple volume for your media, e.g `Volume=/path/to/media:/tv:z` and `Volume=/path/to/another/media:/movie:z`. Consult the documentation for more info.

## Project Website

https://podman.io/

### Useful Links
- https://docs.podman.io/en/stable/markdown/podman-systemd.unit.5.html
- https://www.redhat.com/en/blog/quadlet-podman

<hr>

[**<-- Back to Installing and Managing Software on Bazzite**](./index.md)