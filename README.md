# Self Hosted Stack

Various self hosted services built on podman containers and served remotely through tailscale.

## Services


## Dependencies
- tailscale
- podman-compose
- ufw
- avahi-daemon

## Installation
### Server Setup
- INITIAL:
	- (intall ubuntu, enable ssh, update, upgrade)
	- (copy ssh id from local machine) 
		- shh-copy-id user@server
  - sudo apt install podman-compose ufw avahi-daemon
	- sudo snap install tailscale
	- (quality of life improvements)
		- sudo snap install --classic helix
		- sudo apt install fish bat net-tools
		- chsh -s $(which fish)
		- alias --save bat batcat
		- (edit ~/.config/fish/config.fish)
			- set -gx EDITOR 'hx'
			- set -gx VISUAL 'hx'
	- sudo tailscale login
	- (reserve ip on local network)
  
- SSH:
	- (edit /etc/ssh/sshd_config)
		- #Port 22 -> Port <new ssh port>
		- #PasswordAuthentication yes -> PasswordAuthentication no
	- systemctl daemon-reload
	- systemctl restart ssh.socket
	- sudo reboot

- UFW:
	- sudo ufw enable
	- sudo ufw allow from <ipv4 subnet> to any port <new ssh port>
	- sudo ufw allow from <ipv6 subnet> to any port <new ssh port>
	- sudo ufw allow in on tailscale0 to any port <new ssh port>
	- sudo ufw defualt allow outgoing
	- sudo ufw default deny incoming

- GIT:
	- ssh-keygen -t ed25519 -C "<email>"
	- ssh-agent -s
	- ssh-add ~/.ssh/id_ed25519
	- cat ~/.ssh/id_ed25519.pub
	- (copy ssh key into github)
	- git clone <github repository shh address>
	- git config --global user.email "<email>"
	- git config --global user.name "<name>"
	
- PODMAN-COMPOSE:
	- sudo podman-compose systemd -a create-unit
	- sudo loginctl enable-linger <user>

### Services
- DNS: (start in homelab/dns)
	- (edit /etc/systemd/resolvd.conf)
		- #DNSStubListener=yes -> DNSStubListener=no
	- sudo systemctl restart systemd-resolved
	- echo "net.ipv4.ip_unprivileged_port_start=53" | sudo tee /etc/sysctl.d/20-dns-privileged-port.conf
	- (setup .env)
	- (uncomment admin portal port map in compose.yml)
	- podman-compose systemd -a register
	- systemctl --user enable --now podman-compose@dns
	- (temporarily open ports for webui before reverse proxy is setup -- add tailscale or ipv6 instead if necessary)
		- sudo ufw allow from <ipv4 subnet> to any port 3333
		- sudo ufw allow from <ipv4 subnet> to any port 8080
	- (connect to admin portal and finish intallation)
	- (connect to dashboard and setup)
	- upstream: 
	```
	tls://dns.quad9.net
	https://dns.quad9.net/dns-query
	quic://dns.surfsharkdns.com
	tls://dns.surfsharkdns.com
	tls://dns10.quad9.net
	https://dns10.quad9.net/dns-query
	https://dns.nextdns.io
	tls://dns.nextdns.io
	https://anycast.dns.nextdns.io
	tls://anycast.dns.nextdns.io
	https://dns.adguard-dns.com/dns-query
	tls://dns.adguard-dns.com
	quic://dns.adguard-dns.com
	https://dns.mullvad.net/dns-query
	tls://dns.mullvad.net	
	 ```
	- fallback:
	```
	tls://ada.openbld.net
	https://doh.dns.sb/dns-query
	tls://dot.sb
	tls://getdnsapi.net
	tls://unicast.censurfridns.dk
	tls://dns.cmrg.net
	https://wikimedia-dns.org/dns-query
	tls://wikimedia-dns.org
	```
	- bootstrap:
	```
	2620:fe::fe
	2620:fe::9
	9.9.9.9
	149.112.112.112
	2620:fe::10
	9.9.9.10
	94.140.14.14
	94.140.15.15
	2a10:50c0::ad1:ff
	2a10:50c0::ad2:ff
	185.71.138.138
	2001:67c:930::1
	```
	- dns rewrite:
		- `*.lan-prefix.host.tld` -> <hostname>.local
		- `*.host.tld` -> <server-ts-name>.<ts-fun-name>.ts.net
	- (edit /etc/systemd/resolvd.conf)
		- #DNS= -> DNS=127.0.0.1
		- #FallbackDNS= -> FallbackDNS=9.9.9.9
		- #DNSSEC=no -> DNSSEC=yes
		- #DNSOverTLS=no -> DNSOverTLS=yes
	- sudo systemctl restart systemd-resolved
	- (re-comment admin portal port map in compose.yml)
	- systemctl --user restart podman-compose@dns
	- (remove ufw rule for admin portal and dashboard)
	- sudo ufw allow from <ipv4 subnet> to any port 53
	- sudo ufw allow from <ipv6 subnet> to any port 53
	- sudo ufw allow in on tailscale0 to any port 53
	- (add local and tailnet ip to tailscale dns)
	- (had to manually set dns for lan on mac... idk why)

- REVERSE PROXY: (start in homelab/reverse_proxy)
	- sudo ufw allow from <ipv4 subnet> to any port 80
	- sudo ufw allow from <ipv4 subnet> to any port 443
	- sudo ufw allow from <ipv6 subnet> to any port 80
	- sudo ufw allow from <ipv6 subnet> to any port 443
	- (setup .env)
	- podman-compose systemd -a register
	- systemctl --user enable --now podman-compose@reverse_proxy

- EXIT NODE: (start in homelab/exit_node)
	- (setup .env)
	- podman-compose systemd -a register
	- systemctl --user enable --now podman-compose@exit_node

- FILE SYNC: (start in homelab/file_sync)
	- sudo ufw allow in on <main interface> to any port 22000
	- (setup .env)
	- podman-compose systemd -a register
	- systemctl --user enable --now podman-compose@file_sync
	- (setup web ui)
		- (add username and password)
		- Connections/Global Discovery: ✓
	- (add server device from laptop to share previous folders)
		- Sharing/Addresses: tcp://<server-ts-name>.<ts-fun-name>.ts.net:22000
	- (add other devices and folders)

- HOME: (start in homelab/home)
	- podman-compose systemd -a register
	- systemctl --user enable --now podman-compose@home
