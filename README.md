# Docker Compose Setup with Tailscale and Caddy for HTTPS

## Introduction
This README provides a detailed guide on setting up a Docker Compose environment that integrates Tailscale and Caddy for secure HTTPS communication. The configuration involves using Caddy as a web server and reverse proxy, and Tailscale for creating a secure, private network.

## Prerequisites
- Docker and Docker Compose installed on your system.
- An active Tailscale account. To get your Tailscale auth key, visit the [Tailscale admin console](https://login.tailscale.com/admin/authkeys).

## Components
- **Caddy**: An efficient web server that automatically handles HTTPS certificates.
- **Tailscale**: A secure VPN service that creates a private network for your devices.
- **Web Servers (web1, web2, web3)**: Sample HTTP servers for demonstration purposes.

## Configuration

### Docker Compose File (`docker-compose.yml`)
This file orchestrates the deployment of the Caddy server, Tailscale service, and three web servers. Key configurations include:

1. **Caddy Service**: Configures the Caddy server, specifying the necessary ports (80, 443) and linking to the Caddyfile and other directories for configuration and data storage.

2. **Tailscale Service**: Sets up Tailscale within the Docker environment, enabling secure communication between containers and external Tailscale nodes.

3. **Web Services**: Deploys three instances of the HTTP server, demonstrating the load-balancing and reverse proxy capabilities of Caddy.

### Caddyfile Configuration (`Caddyfile`)
The Caddyfile is the central configuration file for the Caddy web server. It defines how requests are routed and handled. Key features in the Caddyfile include:
- Automatic HTTPS: Caddy will automatically obtain and renew SSL certificates for secure communication.
- Reverse Proxy Settings: Configuration directives for routing requests to appropriate backend services.

### Tailscale Auth Key (`.env`)
The Tailscale auth key is stored in a `.env` file to keep it secure and separate from the main `docker-compose.yml` file. This file is used to set environment variables for the Tailscale service in the Docker Compose configuration. `env_file: .env` is used in the `docker-compose.yml` file to load the Tailscale auth key from the `.env` file.

### Volumes Configuration
Proper volume mapping is crucial for enabling Caddy and Tailscale to function correctly within Docker. The configuration ensures that necessary files, like the Caddyfile and Tailscale's socket file, are accessible within containers.

## Setup and Usage

### Step-by-Step Instructions
1. **Clone the Repository**: Start by cloning this repository to your local machine.

2. **Tailscale Authentication**: Ensure that Tailscale is authenticated and running. This may involve logging into your Tailscale account and connecting your device to your Tailscale network.

3. **Launching Services**: Run `docker-compose up` to start the Caddy server, Tailscale service, and web servers. This command pulls the necessary Docker images and creates the containers based on the `docker-compose.yml` configuration.

4. **Accessing Web Services**: Once the services are up and running, you can access the web servers through your Tailscale network securely via HTTPS.

5. **Customization**: Feel free to customize the `Caddyfile` and Docker Compose file according to your specific requirements.

### Tailscale Certificate Management
The command `docker exec tailscaled tailscale cert <domain>.ts.net` is used to generate a certificate for a specific domain within the Tailscale network, allowing Caddy to serve HTTPS content for that domain. Be sure restart `docker-compose` after generating a new certificate.

## Conclusion
This setup demonstrates a scalable and secure way to deploy web services using Docker, Caddy, and Tailscale. It's suitable for personal projects, development environments, homelabs, or small-scale enterprise applications. The automatic handling of HTTPS by Caddy and the secure networking provided by Tailscale make this setup robust for various use cases.

For further reading and community support, you can explore discussions on Tailscale's forum and Caddy's community pages