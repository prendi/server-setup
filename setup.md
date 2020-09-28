# Tools

1. Install tools:

```bash
sudo apt-get install nfs-common
```

2. Mount the folders:

```bash
sudo mount -t nfs 10.9.0.33:Downloads /mnt/Downloads
```

3. Edit /etc/fstab

```bash
# <file system>     <dir>       <type>   <options>   <dump>     <pass>
10.9.0.33:Downloads /mnt/Downloads  nfs      defaults    0       0
```

Docker compose as a systemd unit
================================

Create file `/etc/systemd/system/docker-compose@.service`

```ini
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always

WorkingDirectory=/etc/docker/compose/%i

# Remove old containers, images and volumes
ExecStartPre=/usr/local/bin/docker-compose down -v
ExecStartPre=/usr/local/bin/docker-compose rm -fv
ExecStartPre=-/bin/bash -c 'docker volume ls -qf "name=%i_" | xargs docker volume rm'
ExecStartPre=-/bin/bash -c 'docker network ls -qf "name=%i_" | xargs docker network rm'
ExecStartPre=-/bin/bash -c 'docker ps -aqf "name=%i_*" | xargs docker rm'

# Compose up
ExecStart=/usr/bin/docker-compose up

# Compose down, remove containers and volumes
ExecStop=/usr/bin/docker-compose down -v

[Install]
WantedBy=multi-user.target
```

Place your `docker-compose.yml` into `/etc/docker/compose/myservice` and call

```
systemctl start docker-compose@myservice
```


Docker cleanup timer with system
================================

Create `/etc/systemd/system/docker-cleanup.timer` with this content:

```ini
[Unit]
Description=Docker cleanup timer

[Timer]
OnUnitInactiveSec=12h

[Install]
WantedBy=timers.target
```

And service file `/etc/systemd/system/docker-cleanup.service`:

```ini
[Unit]
Description=Docker cleanup
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/tmp
User=root
Group=root
ExecStart=/usr/bin/docker system prune -f

[Install]
WantedBy=multi-user.target
```

run `systemctl enable docker-cleanup.timer` for enabling the timer
