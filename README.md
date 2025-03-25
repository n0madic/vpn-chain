# WireGuard VPN Chain

This Ansible playbook automatically sets up a chain of WireGuard VPN servers, creating a multi-hop VPN tunnel for enhanced privacy and security.

## Overview

The WireGuard VPN Chain creates a sequence of VPN servers where your traffic flows through multiple servers before reaching the internet. This approach offers several benefits:

- Distributes trust across multiple servers
- Makes traffic correlation more difficult
- Increases privacy by obscuring the original source

## Architecture

The playbook supports three types of server roles:

1. **Entry Server**: The first server in the chain that clients connect to
2. **Intermediate Server(s)**: Middle servers that relay traffic (optional)
3. **Exit Server**: The final server that connects to the internet

If only one server is specified, it will function as both entry and exit (single server mode).

## Prerequisites

- Ansible 2.9+
- Two or more Debian/Ubuntu servers with public IP addresses
- Root or sudo access on all servers
- Python 3 installed on all servers

## Installation

1. Clone this repository:
   ```
   git clone https://github.com/n0madic/vpn-chain.git
   cd vpn-chain
   ```

2. Edit your inventory file to include your servers:
   ```
   # inventory.ini
   [vpn_servers]
   server1.example.com
   server2.example.com
   server3.example.com
   ```

3. Run the playbook:
   ```
   ansible-playbook -i inventory.ini wireguard_chain.yml
   ```

## Configuration Options

Edit the variables at the top of the `wireguard_chain.yml` playbook to customize your setup:

| Variable | Description | Default |
|----------|-------------|---------|
| `wg_interface` | WireGuard interface name | `wg0` |
| `subnet_mask` | Subnet mask for the VPN network | `8` |
| `main_subnet` | Main subnet for VPN traffic | `10.0.0.0/8` |
| `server_subnet_prefix` | Prefix for server IPs | `10.100.0` |
| `client_subnet` | Subnet for client IPs | `10.200.0.0/24` |
| `client_ip` | Client IP address | `10.200.0.100/32` |
| `routing_table` | Routing table number | `200` |
| `client_config_file` | Path to generated client config | `./vpn-chain.conf` |
| `entry_server` | Hostname of the preferred entry server | `""` (empty) |
| `exit_server` | Hostname of the preferred exit server | `""` (empty) |

## How It Works

1. The playbook orders the servers to create the chain:
   - If specified, uses the preferred entry and/or exit servers
   - Randomly orders the remaining servers for the middle of the chain
2. Each server generates WireGuard keys and configurations
3. The servers are configured to route traffic to the next server in the chain
4. A client configuration file is generated for connecting to the entry server

## Client Connection

After running the playbook, a client configuration file (`vpn-chain.conf`) will be generated in your current directory. You can use this with any WireGuard client:

- **Linux**: Copy to `/etc/wireguard/` and run `wg-quick up vpn-chain`
- **Windows/macOS**: Import the configuration into the WireGuard client application
- **iOS/Android**: Use the QR code feature in the mobile WireGuard app

## Security Considerations

- Each server in the chain only knows its immediate neighbors
- Traffic is encrypted at each hop
- The exit server doesn't know the original client
- Consider using servers in different jurisdictions for maximum privacy

## Troubleshooting

### Connection Issues
- Check if WireGuard service is running on all servers: `systemctl status wg-quick@wg0`
- Verify firewall rules allow WireGuard ports: `ufw status`

### Routing Problems
- Check routing tables: `ip route show table 200`
- Verify IP forwarding is enabled: `sysctl net.ipv4.ip_forward`

### Performance Issues
- Long chains may increase latency
- Consider using servers that are geographically close to each other

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
