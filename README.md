# Self Hosted Stack

Various self hosted services built on podman containers and served remotely through tailscale.

## Services


## Dependencies
- Tailscale
- Podman, podman-compose
- deno (only needed for generating obsidian sync setup uri on the server.)

## Notes
Must install and run tailscale on the host machine outside of podman.
Remember to register pods as systemd services using `podman-compose systemd register`. Then enable the services.
Edit the `podman-compose@.service` file to add the `--force-recreate` option to `podman-compose up --no-start` in order to avoid errors when rebooting.
You must disable dns stub listening in resolv.conf in order to free up port 53 on the host (ubuntu).
Set dns in resolve.conf to 127.0.0.1 (with fallback dns as 9.9.9.9, etc.) in order for local dns to work on exit node and the host in general.

Add two dns rewrites to adguard home:
- `*.domain.name` -> `<local server ip>`
- `*.tailscale.sub.domain` -> `<tailscale server ip>`

When migrating remember to backup your adguard home configuration in `adguard_data_conf:./AdGuardHome.yaml`.

Setting up obsidian-couchdb: (assuming in `./obsidian`)
- add execution permission to setup script: `chmod +x couchdb_init.sh`
- run `couchdb_init.sh`
- run `deno run generate_setup_uri.ts`

This repository includes snippets of documentation and scripts from other authors. This is because the purpose of this repository is to allow me to bring this homelab stack up onto a new server with minimal effort and reliance on other online resources. Also, I have trust issues. I have included credit and licenses where this is the case.
