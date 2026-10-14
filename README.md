# imgshare

Turn files on a remote server - images, HTML pages, whole directories, logs,
videos - into links you can open in your local browser.

```
$ ssh user@server
$ imgshare 1.png
Expires at 11:18
http://203.0.113.10:8000/bYq_WhCNwCKd9P8C/1.png
```

Ctrl+click the link (terminals with OSC 8 hyperlink support) or paste it into
your browser.

- Single Python 3 script, standard library only - nothing else to install
- Images, HTML, PDF, audio and video open in the browser; text files (logs,
  source code, Markdown, CSV, ...) are shown as plain text; anything else downloads
- Share a directory to serve an HTML page together with its CSS/JS/images
- Video and audio support seeking (HTTP Range)
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
imgshare 1.png [report.html ...]   share files and print links (expire after 1 hour by default)
imgshare ./site/                   share a directory (opens its index.html, or lists it)
imgshare -t 10m 1.png              set the lifetime (s/m/h/d)
imgshare -x ./site/                disable the HTML sandbox (see below)
imgshare -l                        list active shares
imgshare -s                        stop the server and remove all shares
```

Sharing several paths at once also prints an `All:` link to an overview page
(images are shown as thumbnails).

A single self-contained `.html` file can be shared directly. If the page loads
other files (`style.css`, `img/...`, `app.js`), share its directory instead so
those files are reachable too.

## Configuration

| Variable        | Default                     | Description                        |
|-----------------|-----------------------------|------------------------------------|
| `IMGSHARE_HOST` | address you SSHed to        | host name / public IP used in links |
| `IMGSHARE_PORT` | `8000`                      | listen port                        |
| `IMGSHARE_BIND` | `0.0.0.0`                   | listen address                     |

State (share records, PID file, server log) lives in
`${XDG_CACHE_HOME:-~/.cache}/imgshare`.

## Security notes

- Traffic is plain HTTP: the link and the content are visible to anyone on the
  network path. Don't use it for sensitive files.
- Anyone who has the link can open it until it expires. Use `-t` for short
  lifetimes and `imgshare -s` to revoke everything immediately.
- Inside shared directories, dotfiles and dot-directories (`.git`, `.env`, ...)
  are never served, and symlinks pointing outside the directory are refused.
- HTML and SVG files are served with a `Content-Security-Policy: sandbox`
  header, so each page runs in its own unique origin. Scripts, module scripts
  and `fetch()` of the page's own assets still work, but `localStorage`,
  cookies and similar storage do not. Use `-x` for pages that need them, and
  only with HTML you trust.
- For private viewing, an SSH tunnel avoids exposing anything publicly:
  `ssh -L 8000:localhost:8000 user@server` with `IMGSHARE_BIND=127.0.0.1`
  and `IMGSHARE_HOST=localhost`.
