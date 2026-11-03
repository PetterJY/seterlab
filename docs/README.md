# Seterlab

Configuration for Seterlab, my home server: a Lenovo V50s SFF running Ubuntu Server 26.04 LTS. Services run as rootless Podman containers, defined as Quadlet units in `quadlet/` and started by systemd.

## Architecture

```text
Device on the tailnet
  │  <service>.seterlab.no → the server's Tailscale address
  ▼
Tailscale → ufw → Nginx container (TLS for *.seterlab.no)
  │  proxy_pass http://<container>:<port>
  ├── <stack> network ── <service>  <service>
  └── <stack> network ── <service>
```

Every service runs as a rootless Podman container, with one `.container` file per service and one `.network` file per stack. The apps' own data, like databases and settings, lives on the server and isn't in this repo.

The server has two disks: an NVMe for the system and app data, and an HDD for bulk data like media.

## Networking

Nginx also runs in a container and is open for HTTP and HTTPS on ports 80 and 443, with HTTP only redirecting to HTTPS. Its config is in `nginx/conf.d/`, with one server block per subdomain.

A Let's Encrypt wildcard certificate covers every subdomain. TLS ends at Nginx, and the traffic from Nginx to the services is mostly plain HTTP, which never leaves the host. Each stack has its own Podman network, and Nginx joins all of them to reach each service by container name.

## Tailscale

The server is only meant to be reached over Tailscale. Public DNS points `*.seterlab.no` at the server's Tailscale address, which doesn't route outside the tailnet.

The access policy, by role:

| Who | Can reach |
| --- | --- |
| Every member | Their own devices, and the server on ports 80 and 443 |
| Admins | SSH to the server, and the server as an exit node |

Grants target `tag:server` rather than an IP address.
