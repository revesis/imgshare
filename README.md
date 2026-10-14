# imgshare

Turn an image on a remote server into a link you can open in your local browser.

```
$ ssh user@server
$ imgshare 1.png
Expires at 11:18
http://203.0.113.10:8000/bYq_WhCNwCKd9P8C/1.png
```

Ctrl+click the link (terminals with OSC 8 hyperlink support) or paste it into
your browser.

- Single Python 3 script, standard library only - nothing else to install
- Each share gets a random token; only shared files are reachable
- Links expire (default 1 hour)
- The background server starts on demand and exits when no shares remain
- Links keep working after you log out, until they expire

## Requirements

- Python 3.7+ on the server
- A public IP on the server, with the port (default `8000`) open in the
  firewall / cloud security group

## Install

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/revesis/imgshare/main/imgshare -o ~/.local/bin/imgshare
chmod +x ~/.local/bin/imgshare
```

Set the public address used in links (recommended, required on most cloud
servers where the interface only has a private IP):

```sh
echo 'export IMGSHARE_HOST=<your-public-ip-or-domain>' >> ~/.bashrc
```

Open the port, e.g.:

```sh
sudo ufw allow 8000/tcp
```

## Usage

```
imgshare 1.png [2.jpg ...]   share images and print links (expire after 1 hour by default)
imgshare -t 10m 1.png        set the lifetime (s/m/h/d)
imgshare -l                  list active shares
imgshare -s                  stop the server and remove all shares
```

Sharing several files at once also prints an `All:` link to a page showing
all of them.

## Configuration

| Variable        | Default                     | Description                        |
|-----------------|-----------------------------|------------------------------------|
| `IMGSHARE_HOST` | address you SSHed to        | host name / public IP used in links |
| `IMGSHARE_PORT` | `8000`                      | listen port                        |
| `IMGSHARE_BIND` | `0.0.0.0`                   | listen address                     |

State (share records, PID file, server log) lives in
`${XDG_CACHE_HOME:-~/.cache}/imgshare`.

## Security notes

- Traffic is plain HTTP: the link and the image are visible to anyone on the
  network path. Don't use it for sensitive images.
- Anyone who has the link can open it until it expires. Use `-t` for short
  lifetimes and `imgshare -s` to revoke everything immediately.
- For private viewing, an SSH tunnel avoids exposing anything publicly:
  `ssh -L 8000:localhost:8000 user@server` with `IMGSHARE_BIND=127.0.0.1`
  and `IMGSHARE_HOST=localhost`.
