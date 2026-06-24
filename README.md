# Caddy + Tailscale HTTPS Proxy

Docker Compose setup for running private web services behind Caddy with HTTPS on a Tailscale `.ts.net` name.

This is meant for a small self-hosted machine where services should be reachable over your tailnet without public DNS, public port forwarding, or manual certificate handling.

## What It Runs

- `tailscale`: joins the host to your tailnet and exposes `tailscaled.sock`.
- `caddy`: reverse proxy with automatic HTTPS for the Tailscale domain.
- `web1`, `web2`, `web3`: sample backend services using Apache `httpd`.
- `proxy-network`: private Docker network used by Caddy to reach the backend services.

## Request Flow

```text
Client on tailnet
  -> https://your-machine.your-tailnet.ts.net/web1
  -> Caddy
  -> web1:80 on proxy-network
```

See [architecture.md](./architecture.md) for the Mermaid diagram.

## Prerequisites

- Docker and Docker Compose
- Tailscale account
- Tailscale auth key from the admin console

Use an ephemeral auth key if the machine is disposable. Use a reusable key if you want the container to rejoin without manual work after rebuilds.

## Setup

1. Clone the repository.

   ```bash
   git clone <repository_url>
   cd https-caddy-tailscale
   ```

2. Create `.env` in the project root.

   ```bash
   cp .env.example .env
   ```

   Then set `TS_AUTHKEY` in `.env`.

3. Update the Tailscale domain in [caddy/Caddyfile](./caddy/Caddyfile).

   ```caddy
   your-machine-name.your-tailnet.ts.net {
     import network_paths
   }
   ```

   You can find the full machine name in the Tailscale admin console under **Machines**.

4. Update or remove the LAN-only HTTP block if it does not match your network.

   ```caddy
   http://192.168.0.31 {
     import network_paths
   }
   ```

5. Start the stack.

   ```bash
   docker-compose up -d
   ```

6. Test from a device connected to the same tailnet.

   ```text
   https://your-machine-name.your-tailnet.ts.net/web1
   https://your-machine-name.your-tailnet.ts.net/web2
   https://your-machine-name.your-tailnet.ts.net/web3
   ```

## Configuration

### Caddy

[caddy/Caddyfile](./caddy/Caddyfile) defines one reusable route group:

```caddy
(network_paths) {
  handle_path /web1/* {
    reverse_proxy web1:80
  }
}
```

Add more services by adding another `handle_path` block and putting the backend service on `proxy-network`.

Caddy uses `/var/run/tailscale/tailscaled.sock` from the Tailscale container. This lets it request certificates for the `.ts.net` domain.

### Tailscale

The Tailscale container uses:

- `network_mode: host`
- `NET_ADMIN` and `NET_RAW`
- `/dev/net/tun`
- persistent state in `./tailscale/varlib`

The auth key is read from `.env`. Do not commit `.env`.

### Volumes

Runtime state is intentionally ignored by Git:

- `caddy/data`
- `caddy/config`
- `tailscale/tmp`
- `tailscale/varlib`

## Common Tasks

View logs:

```bash
docker-compose logs caddy
docker-compose logs tailscale
```

Restart:

```bash
docker-compose down
docker-compose up -d
```

Reload after Caddyfile changes:

```bash
docker-compose restart caddy
```

## Troubleshooting

### Tailscale does not join

- Check that `TS_AUTHKEY` is set in `.env`.
- Confirm the key is valid and not expired.
- Check logs with `docker-compose logs tailscale`.

### HTTPS does not work

- Confirm the Caddy site address is the exact `.ts.net` name of this machine.
- Confirm Caddy can read `./tailscale/tmp/tailscaled.sock`.
- Check logs with `docker-compose logs caddy`.

### Backend route returns an error

- Confirm the backend service is running.
- Confirm it is attached to `proxy-network`.
- Confirm the service name in `reverse_proxy` matches the Compose service name.

## Security Notes

- Services are exposed through Tailscale, not directly to the public internet.
- Keep backend services off host ports unless required.
- Treat the Tailscale auth key as a secret.
- Review ACLs in Tailscale if this is used by more than one person.
