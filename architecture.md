# Architecture

```mermaid
graph TD
    User[Client on tailnet] -->|HTTPS| Caddy[Caddy :443]

    subgraph Host[Docker host]
        Caddy -->|/web1| Web1[web1:80]
        Caddy -->|/web2| Web2[web2:80]
        Caddy -->|/web3| Web3[web3:80]

        Caddy <-->|tailscaled.sock| Tailscale[tailscale]
        Tailscale --> Tailnet[Tailscale tailnet]

        subgraph ProxyNet[proxy-network]
            Web1
            Web2
            Web3
        end
    end

    Lan[LAN client] -->|optional HTTP| Caddy80[Caddy :80]
    Caddy80 --> Caddy
```

## Notes

- Caddy terminates HTTPS for the Tailscale `.ts.net` name.
- Caddy reaches backend services over `proxy-network`.
- Tailscale state is persisted under `./tailscale/varlib`.
- Caddy data and config are persisted under `./caddy`.
- The LAN HTTP block is optional and should match your local network.
