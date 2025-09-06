# Self Hosted Stack

Various self hosted services built on podman containers and served remotely through tailscale.

## Services


## Notes
Must install and run tailscale on the host machine outside of podman.
Remember to register pods as systemd services using `podman-compose systemd register`. Then enable the services.
Edit the `podman-compose@.service` file to add the `--force-recreate` option to `podman-compose up --no-start` in order to avoid errors when rebooting.
You must disable dns stub listening in resolv.conf in order to free up port 53 on the host (ubuntu).
Set dns in resolve.conf to 127.0.0.1 (with fallback dns as 9.9.9.9, etc.) in order for local dns to work on exit node and the host in general.

Add two dns rewrites to adguard home:
- `*.domain.name` -> `<local server ip>`
- `*.tailscale.sub.domain` -> `<tailscale server ip>`
