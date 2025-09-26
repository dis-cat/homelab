# Self Hosted Stack

Various self hosted services built on podman containers and served remotely through tailscale.

## Services


## Dependencies
- Tailscale
- Podman, podman-compose

## Notes
Must install and run tailscale on the host machine outside of podman.
Remember to register pods as systemd services using `podman-compose systemd register`. Then enable the services.
Edit the `podman-compose@.service` file to add the `--force-recreate` option to `podman-compose up --no-start` in order to avoid errors when rebooting.
You must disable dns stub listening in resolv.conf in order to free up port 53 on the host (ubuntu). Yop must also allow unprivileged use of port 53 with `echo "net.ipv4.ip_unprivileged_port_start=53" | sudo tee /etc/sysctl.d/20-dns-privileged-port.conf` or editing the file directly (make sure its editable!). (peep https://github.com/hat3ph/docker-adguard-unbound)
Set dns in resolve.conf to 127.0.0.1 (with fallback dns as 9.9.9.9, etc. (mostly for initial pull of docker images, can maybe be removed after set up... security vs convenience idk)) in order for local dns to work on exit node and the host in general.

Add two dns rewrites to adguard home:
- `*.domain.name` -> `<local server ip>`
- `*.tailscale-sub-domain.domain.name` -> `<tailscale server ip>`

When migrating remember to backup your adguard home configuration in `adguard_data_conf:./AdGuardHome.yaml`.

Syncthing:
- required GUI settings: connections->global discovery
- connect a device:
  - copy id from server ui
  - add device on client, paste server id
  - add remote device->sharing->addresses: tcp://<server-ts-name>.<ts-fun-name>.ts.net:22000
  - confirm on server
