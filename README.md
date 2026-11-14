# Movies Server

A simple HTTP server for watching movies stored on your Mac in Kodi over your home network.
Runs with Docker Compose on OrbStack using the official nginx image.
Provides a browsable file index and HTTP Range support for seeking. Movies stay on your Mac;
the container mounts only the selected folder, with read-only access.

## Setup

Start OrbStack with Docker Compose available, then run:

```sh
cp .env.example .env
```

Set `BIND_IP` in `.env` to your Mac's LAN IPv4 address. You can find it in
System Settings → Wi-Fi → Details → TCP/IP. By default, the server is accessible
only from the Mac itself.

`MOVIES_DIR` defaults to `$HOME/Movies`. The folder must already exist.
Set `MOVIES_UID` and `MOVIES_GID` to the folder owner's user and group IDs
(run `id -u` and `id -g` for your current user). The defaults are `501:20`.
Local settings in `.env` are excluded from Git.

## Start and stop

Run these commands from the project directory:

```sh
# Create and start the container, or apply configuration changes.
docker --context orbstack compose up -d

# Stop the server.
docker --context orbstack compose stop

# Start an existing container.
docker --context orbstack compose start

# Check its status.
docker --context orbstack compose ps
```

After the first launch, OrbStack shows the **movies-server** project with a
**movies** container. You can start and stop it using the OrbStack app.
Automatic restarts are disabled (`restart: "no"`), so start the server manually
after restarting Docker or OrbStack.

After editing `.env`, run `docker --context orbstack compose up -d` to apply the changes.
To remove the container, run `docker --context orbstack compose down`.
Your movie files remain untouched.

## Kodi

Add `http://<BIND_IP>:8765/` as an HTTP source under
Videos → Files → Add videos → Browse → Add network location
(protocol: Web server directory / HTTP). If you changed `PORT`, use that port instead.
No username or password is required. An existing source using the same URL will keep working.

Add movies to the shared folder and refresh the directory in Kodi.
The server does not transcode video; format support depends on your TV and Kodi.

## Network access and Mac sleep

This server is intended for a trusted home network. Anyone who can reach the
configured address and port can read files in the shared folder. Do not expose
this port to the internet. Hidden paths and access through symbolic links are
blocked. The movie folder is mounted read-only to prevent writes from the container.

Your Mac and OrbStack must stay running during playback. The container does not
invoke macOS `caffeinate`. To prevent idle sleep while watching, run
`caffeinate -i` in a separate Terminal window and press Ctrl+C when finished.
Keep your laptop lid open.

Reserve your Mac's LAN address on your router to keep the Kodi source URL stable.
If the address changes, update both `.env` and the Kodi source, then run
`docker --context orbstack compose up -d`.

## Updating nginx

The nginx image is pinned by digest for reproducibility. To update it, replace
the image digest in `compose.yaml` with the current digest of the official
`nginx:stable-alpine` image, then run `docker --context orbstack compose up -d`.

## References

- [OrbStack and Docker Compose](https://docs.orbstack.dev/docker/)
- [Official nginx image](https://hub.docker.com/_/nginx)
