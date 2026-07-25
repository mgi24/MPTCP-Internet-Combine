# MPTCP Internet Combine
Combine 2 (or more) internet links into a single faster connection using MPTCP.

Tools used: sing-box, socat, mptcpize.

**WARNING:** experimental scripts — review before running on production systems.

If you see "line 4: $'\r': command not found", convert line endings:

```bash
sudo sed -i 's/\r$//' "namefile".sh
```

**Requirements**
- Linux VPS or host with a recent kernel (5.14+ recommended)

**Quick start (Both server and client)**
1. Clone the repo:

```bash
git clone https://github.com/mgi24/MPTCP-Internet-Combine.git
cd MPTCP-Internet-Combine
```

**SERVER**
1. Edit the variables at the top of [serverinstall.sh](serverinstall.sh) (`PUBLIC_IP`, `interface`, `Socat_Port`, `socat_internal_port`) then run:

```bash
sudo bash serverinstall.sh
sudo bash serverup.sh
```

**CLIENT**
1. Edit the variables at the top of [serverapply/clientinstall.sh](serverapply/clientinstall.sh) (`VPS_IP`, `IP2`, `interface1`, `interface2`, `socat_internal_port`) then run:

```bash
sudo bash clientinstall.sh
sudo bash clientup.sh
```

**LAN Creator (optional)**
You can create a local DHCP-enabled LAN that is ready to route traffic via the MPTCP tunnel using `lan_creator.sh`:

- File: [serverapply/lan_creator.sh](serverapply/lan_creator.sh)
- Edit the variables at the top of the script:
  - `interface` — the local interface to host the LAN (e.g. `eth2`)
  - `ip` — gateway IP to assign to that interface (e.g. `10.0.99.1`)

Run:

```bash
sudo bash serverapply/lan_creator.sh
```

**Web Config Panel (Client)**

A browser-based configuration panel is available on the client machine. It runs on **port 80** and is managed by systemd (`python.mptcp.service`).

- View MPTCP status, interfaces, traffic stats, and active connections
- Configure sing-box Shadowsocks settings, output interfaces, and iptables rules
- Start/stop services (sing-box, socat, keepalive, python panel)

Access via browser: `http://<client_ip>/`

The panel is auto-started on boot via systemd.

**Systemd Services (Client)**

| Service | Unit | Description |
|---|---|---|
| MPTCP daemon | `mptcp.service` | Multipath TCP kernel daemon |
| Web config panel | `python.mptcp.service` | Python web UI (port 80) |
| sing-box | `sing-box.service` | Shadowsocks proxy |
| socat bridge | `socat-bridge.service` | Port forwarding to socat |
| TCP keepalive | `tcp-keepalive.service` | Keepalive tuning |

All client services are enabled on boot by default after running `clientinstall.sh`.

**Start and Stop**

```bash
sudo bash serverup.sh           # start server services
sudo bash serverdown.sh         # stop server services
sudo bash clientup.sh           # start client services
sudo bash clientdown.sh         # stop client services
sudo bash flushing.sh           # cleanup (server & client)
sudo bash speedlimiter.sh start # apply tc rate limits
sudo bash speedlimiter.sh stop  # remove tc rate limits
```

**Important variables**
- `PUBLIC_IP` — public IP address of the VPS (mandatory, set in [serverinstall.sh](serverinstall.sh))
- `interface` — external network interface name (e.g. `eth0`) on the server (mandatory)
- `Socat_Port` — public port for the mptcpized socat listener (default `8888`)
- `socat_internal_port` — local port for the `sing-box` inbound listener (server default `8080`, client uses `8081`)

**Security and final notes**
- Review firewall rules and NAT: MASQUERADE will change outbound source addresses.
- `rp_filter` is disabled in installers to allow asymmetric routing for MPTCP; assess your threat model before changing this.
- Scripts are experimental: audit and test in a safe environment first.

