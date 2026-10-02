# tailnet-multi

Run any number of extra Tailscale/headscale tailnets alongside the primary one
(the system `tailscaled`), each with optional inbound SSH.

Each extra tailnet is its own `tailscaled` instance (systemd template unit
`tailscaled-multi@<name>`) in userspace-networking mode, with its own state dir,
socket, UDP port and SOCKS5 proxy. Userspace mode creates no TUN device or routes,
so overlapping 100.64.0.0/10 ranges across headscale instances can't conflict.

## Usage

    sudo ./tailnet-multi add vpn.pxl8.cc               # name defaults to pxl8; hostname to <host>-pxl8
    sudo ./tailnet-multi add vpn.example.com other --hostname box --no-ssh
    ./tailnet-multi list
    ./tailnet-multi status pxl8
    ./tailnet-multi ts pxl8 ping <peer>                # any tailscale subcommand
    sudo ./tailnet-multi remove pxl8 [--purge]

The name defaults to the domain without its TLD and first label (`vpn.pxl8.cc` -> `pxl8`); pass a second argument to override.

`add` prints a registration URL and waits; approve it on that headscale server:
`headscale nodes register --user <user> --key <key from URL>`. Then allow `tcp:22`
to the node in that tailnet's ACL. Inbound SSH works via `tailscale serve`
forwarding tailnet :22 to local sshd.

Ports are auto-assigned (UDP from 41642, SOCKS5 from 1055) and stored in
`/etc/tailnet-multi/<name>.env`. Outbound access from this host to peers on an
extra tailnet goes through that tailnet's SOCKS5 proxy (`localhost:<SOCKS_PORT>`).
