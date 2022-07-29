# Headscale-UI
A web frontend for the [headscale](https://github.com/juanfont/headscale) Tailscale-compatible coordination server.

![](documentation/assets/headscale-ui-demo.gif)

## Installation
Headscale-UI is currently released as a static site: just take the release and host with your favorite web server. Headscale-UI expects to be served from the `/web` path to avoid overlap with headscale on the same domain. Note that due to CORS (see https://github.com/juanfont/headscale/issues/623), headscale UI *must* be served on the same subdomain, or CORS headers injected via reverse proxy.

### Docker Installation
If you are using docker, you can install `headscale` alongside `headscale-ui`, like so:

```yaml
version: '3.5'
services:
  headscale:
    image: headscale/headscale:latest-alpine
    container_name: headscale
    volumes:
      - ./container-config:/etc/headscale
      - ./container-data/data:/var/lib/headscale
    # ports:
      # - 27896:8080
    command: headscale serve
    restart: unless-stopped
  headscale-ui:
    image: ghcr.io/gurucomputing/headscale-ui:latest
    container_name: headscale-ui
    # ports:
      # - 9443:443
```

Headscale UI serves on port 443 and uses a self signed cert by default.

### Proxy Settings
You will need a reverse proxy to install `headscale-ui` on your domain. Here is an example [Caddy Config](https://caddyserver.com/) to achieve this:
```
https://hs.yourdomain.com.au {
	reverse_proxy /web* https://headscale-ui {
		transport http {
			tls_insecure_skip_verify
		}
	}

	reverse_proxy * http://headscale:8080
}

```

### Cross Domain Installation
If you do not want to configure headscale-ui on the same subdomain as headscale, you must intercept headscale traffic via your reverse proxy to fix CORS (see https://github.com/juanfont/headscale/issues/623). Here is an example fix with Caddy, replacing your headscale UI domain with `hs-ui.yourdomain.com.au`:
```
hs.yourdomain.com.au {
  @hs-options {
    host hs.yourdomain.com.au
    method OPTIONS
  }
  @hs-other {
    host hs.yourdomain.com.au
  }
  handle @hs-options {
    header {
      Access-Control-Allow-Origin https://hs-ui.yourdomain.au
      Access-Control-Allow-Headers *
      Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE"
    }
    respond 204
  }
  handle @hs-other {
    reverse_proxy http://headscale:8080 {
      header_down Access-Control-Allow-Origin https://hs-ui.yourdomain.com.au
      header_down Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE"
      header_down Access-Control-Allow-Headers *
    }
  }
}
```

## Development
see [development](/documentation/Development.md) for details

### Style Guide
see [style](/documentation/Style.md) for details

## Architecture
See [architecture](/documentation/Architecture.md) for details
