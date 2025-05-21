# Securely Self-Host Your Services with Caddy and Tailscale via Docker Compose

Tired of complex setups for exposing local services to the internet securely? This project provides a straightforward Docker Compose configuration to run web services behind Caddy, your automatic HTTPS reverse proxy, with secure remote access powered by Tailscale.

No more manual port forwarding or wrestling with SSL certificates!

## Why This Setup?

-   **Automatic HTTPS:** Caddy handles SSL certificate acquisition and renewal seamlessly, leveraging Tailscale for `.ts.net` domain certificates.
-   **Secure Remote Access:** Tailscale creates a private, encrypted network (a "tailnet") for your devices, making your services accessible from anywhere without exposing them directly to the public internet.
-   **Simplified Networking:** Docker Compose orchestrates the services, and Caddy acts as a reverse proxy, directing traffic to your backend applications.
-   **Easy to Deploy:** Get up and running with a few simple commands.
-   **Demonstrates Best Practices:** Uses `.env` for secrets, persistent volumes for data, and a clear, modular Caddy configuration.

## Architecture Diagram

This diagram illustrates how requests flow through the system:

```mermaid
graph TD
    User[External User] -->|HTTPS via Tailscale VPN| CaddyService[Caddy Container Port 443]

    subgraph Docker Host
        CaddyService -->|Reverse Proxy via proxy-network| Web1[web1:80 on proxy-network]
        CaddyService -->|Reverse Proxy via proxy-network| Web2[web2:80 on proxy-network]
        CaddyService -->|Reverse Proxy via proxy-network| Web3[web3:80 on proxy-network]

        CaddyService <-->|Uses tailscaled.sock| TailscaleService[Tailscale Container]
        TailscaleService -->|Connects to| TailscaleVPN[Tailscale VPN]

        subgraph "proxy-network (Docker Network)"
            Web1
            Web2
            Web3
        end
    end

    User -->|Potentially HTTP local access| CaddyServicePort80[Caddy Container Port 80]
    CaddyServicePort80 -->|Forwards to HTTPS or serves local IP| CaddyService
```

## Prerequisites

Before you begin, ensure you have the following:

1.  **Docker and Docker Compose:** Installed on your system. [Install Docker](https://docs.docker.com/engine/install/), [Install Docker Compose](https://docs.docker.com/compose/install/).
2.  **Tailscale Account:** A free Tailscale account. [Sign up here](https://tailscale.com/start).
3.  **Tailscale Auth Key:** An auth key from your Tailscale account. Ephemeral keys are recommended for services. Generate one in the [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys) under "Auth keys".

## Project Structure

-   `docker-compose.yaml`: Defines and configures all the services (Caddy, Tailscale, example web apps).
-   `caddy/`: Directory containing Caddy configurations.
    -   `Caddyfile`: The main configuration file for Caddy. **You'll need to edit this!**
-   `.env` (you will create this): Stores your Tailscale authentication key.
-   `tailscale/`: Directories used by Tailscale for persistent state and its socket file.

## Getting Started

1.  **Clone the Repository:**
    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Create and Configure `.env` File:**
    Create a file named `.env` in the root of the project directory with your Tailscale auth key:
    ```ini
    # .env
    TS_AUTHKEY=tskey-auth-your-very-long-auth-key-goes-here
    # Optional: Define your Tailscale hostname if you want to use it in Caddyfile (though Caddy can often get it automatically)
    # TS_HOSTNAME=your-machine-name
    ```
    Replace `tskey-auth-your-very-long-auth-key-goes-here` with your actual Tailscale auth key.

3.  **Configure `Caddyfile`:**
    Open `caddy/Caddyfile` and change the placeholder domain to your actual Tailscale machine name.
    You can find your machine's Tailscale name in the [Tailscale Admin Console](https://login.tailscale.com/admin/machines). It will look something like `your-machine-name.your-tailnet-name.ts.net`.

    Change this line:
    ```caddy
    docker-desktop.ainu-herring.ts.net { # <-- CHANGE THIS
      import network_paths
    }
    ```
    to (for example):
    ```caddy
    your-cool-server.your-tailnet.ts.net { # <-- Use your actual Tailscale FQDN
      import network_paths
    }
    ```
    The `http://192.168.0.31` entry is for local LAN access over HTTP. You can keep it, modify it, or remove it as needed.

4.  **Start the Services:**
    Launch everything using Docker Compose:
    ```bash
    docker-compose up -d
    ```
    The `-d` flag runs the containers in detached mode.

5.  **Verify:**
    -   Check that your machine has joined your Tailscale network in the [Tailscale Admin Console](https://login.tailscale.com/admin/machines).
    -   Access one of your web services via its Tailscale HTTPS URL, e.g., `https://your-machine-name.your-tailnet-name.ts.net/web1`. You should see the default Apache page from the `web1` service.

## Configuration Deep Dive

-   **`docker-compose.yaml`:**
    -   `caddy` service: Runs the Caddy web server. It mounts the `Caddyfile`, persistent storage volumes (`./caddy/data`, `./caddy/config`), and crucially, the `tailscaled.sock` from the `tailscale` container. This socket allows Caddy to directly integrate with Tailscale for certificate management.
    -   `tailscale` service: Runs the Tailscale daemon. It uses `network_mode: host` and `cap_add: [NET_ADMIN, NET_RAW]` to manage network interfaces. Its state is persisted in `./tailscale/varlib`. The `.env` file provides the `TS_AUTHKEY`.
    -   `web1`, `web2`, `web3`: Simple Apache `httpd` services used as examples. They are on the `proxy-network` and are not directly exposed.

-   **`caddy/Caddyfile`:**
    -   `(network_paths)`: A reusable snippet that defines how paths like `/web1/*` are proxied to the backend services.
    -   `your-machine-name.your-tailnet.ts.net { ... }`: This block defines a site. Caddy will automatically obtain an HTTPS certificate for this Tailscale domain.
    -   `http://192.168.0.31 { ... }`: An example of serving a site over plain HTTP for a local IP.

## Troubleshooting & Tips

-   **View Logs:** To see the logs from Caddy or Tailscale:
    ```bash
    docker-compose logs caddy
    docker-compose logs tailscale
    ```
-   **Tailscale Auth Issues:**
    -   Ensure your `TS_AUTHKEY` in `.env` is correct and not expired.
    -   Check `docker-compose logs tailscale` for authentication errors. You might see messages about "login interactive" if the key is invalid or missing.
-   **Caddy Certificate Issues:**
    -   Ensure Caddy can communicate with Tailscale. The volume mount for `tailscaled.sock` in `docker-compose.yaml` is essential.
    -   Check `docker-compose logs caddy`. Caddy is usually very informative about its certificate process.
    -   Make sure the domain name in your `Caddyfile` is a valid Tailscale name for the machine running these services.
-   **Restarting:**
    ```bash
    docker-compose down
    docker-compose up -d
    ```

## Best Practices Incorporated

This setup demonstrates several best practices:
-   **Infrastructure as Code:** `docker-compose.yaml` defines your service stack.
-   **Secrets Management:** Using `.env` for the Tailscale auth key.
-   **Persistent Storage:** Volumes are used for Caddy data/config and Tailscale state.
-   **Automatic HTTPS:** Caddy's core feature, enhanced by Tailscale integration.
-   **Network Segmentation:** The `proxy-network` isolates backend services.
-   **Clear Configuration:** Caddyfile snippets promote readability and maintainability.

Happy self-hosting!
```
