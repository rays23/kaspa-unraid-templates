# Kaspa Node + Monitor — Unraid Community Templates

Unraid Docker Templates for running a **Kaspa Full-Node** (rusty-kaspad) with the **KaspaNodeMonitor** web dashboard.

<img src="icons/kaspa-node-icon.png" alt="Kaspa" width="64" height="64" />

## What's Included

| Template | Image | Purpose |
|---|---|---|
| **Kaspa-Node** | `kaspanet/rusty-kaspad:latest` | Rust-based Kaspa Full-Node with UTXO index |
| **Kaspa-Monitor-Server** | `rays23/kaspa-monitor-server:latest` | gRPC-to-WebSocket bridge for monitoring data |
| **Kaspa-Monitor-Client** | `rays23/kaspa-monitor-client:latest` | Nginx-based real-time web dashboard |

## Prerequisites

- Unraid 6.12+ (tested on 7.2.3)
- Community Applications plugin installed
- ~20-50 GB free disk space for blockchain data

## Installation

### Step 1: Install Kaspa-Node

1. Search for **Kaspa-Node** in Community Applications
2. Click **Install** — all defaults are preconfigured
3. The node will start syncing the blockchain (Initial Block Download). This takes several hours.
4. Wait until IBD is complete before proceeding (check Docker logs for sync progress)

> **Important:** The template already includes `--rpclisten=0.0.0.0:16110` which is required for the monitor to connect via gRPC. Do not remove this parameter.

### Step 2: Install Kaspa-Monitor-Server

1. Search for **Kaspa-Monitor-Server** in Community Applications
2. Click **Install** — default `NODE_URL` is `127.0.0.1:16110` (correct for host networking)
3. The server will connect to the Kaspa-Node via gRPC and expose a WebSocket API on port **8124**

### Step 3: Install Kaspa-Monitor-Client

1. Search for **Kaspa-Monitor-Client** in Community Applications
2. Click **Install**
3. **After first start**, edit the config file at `/mnt/user/appdata/kaspa-monitor/client/config.json`:

```json
{
  "welcomeText": "Kaspa Node Monitor",
  "wsApiURL": "ws://YOUR_UNRAID_IP:8124/ws",
  "theme": "mainDark"
}
```

Replace `YOUR_UNRAID_IP` with your Unraid server's IP address (e.g., `192.168.178.39`).

4. Restart the Kaspa-Monitor-Client container
5. Open the dashboard at `http://YOUR_UNRAID_IP:2989/`

## Ports

| Port | Protocol | Purpose |
|---|---|---|
| 16110 | TCP | gRPC API (Node ↔ Monitor communication) |
| 16111 | TCP | P2P Network (Node ↔ Kaspa Network) |
| 17110 | TCP | wRPC Borsh (WASM clients) |
| 18110 | TCP | wRPC JSON (browser-based monitoring) |
| 8124 | TCP | WebSocket API (Monitor Server ↔ Client) |
| 2989 | TCP | Web Dashboard (Monitor Client) |

## Network Architecture

```
┌─────────────────┐     gRPC      ┌──────────────────────┐    WebSocket    ┌──────────────────────┐
│   Kaspa-Node    │──────────────▶│  Kaspa-Monitor-Server │◀──────────────▶│  Kaspa-Monitor-Client │
│  (Bridge)       │  Port 16110   │  (Host Network)       │   Port 8124    │  (Bridge)             │
│  rusty-kaspad   │               │  Node.js Backend      │                │  Nginx + React UI     │
└─────────────────┘               └──────────────────────┘                └──────────────────────┘
     Ports:                                                                     Port:
     16110 (gRPC)                                                               2989 (Dashboard)
     16111 (P2P)
     17110 (wRPC Borsh)
     18110 (wRPC JSON)
```

## Configuration

### Kaspa-Node Parameters

The node starts with these default arguments (configurable via `PostArgs`):

| Parameter | Default | Description |
|---|---|---|
| `--utxoindex` | enabled | Required for wallet/monitoring functionality |
| `--disable-upnp` | enabled | Recommended for server environments |
| `--maxinpeers` | 64 | Maximum incoming peer connections |
| `--outpeers` | 32 | Outgoing peer connections |
| `--rpclisten` | 0.0.0.0:16110 | **Critical:** Binds gRPC to all interfaces |

### Monitor-Server Environment Variables

| Variable | Default | Description |
|---|---|---|
| `NODE_URL` | 127.0.0.1:16110 | gRPC endpoint of the Kaspa-Node |
| `PORT` | 8124 | WebSocket server port |
| `HOST` | 0.0.0.0 | Bind address |
| `SERVE_FRONTEND` | false | Use separate client container |
| `ALLOW_SERVER_INFORMATION` | true | Show server info in dashboard |
| `LOCATION` | Unraid | Location label in dashboard |
| `LOG_LEVEL` | info | Logging verbosity |

### Monitor-Client Configuration (config.json)

| Key | Default | Description |
|---|---|---|
| `welcomeText` | Kaspa Node Monitor | Dashboard title |
| `wsApiURL` | ws://UNRAID_IP:8124/ws | WebSocket URL — **must be configured** |
| `theme` | mainDark | UI theme (mainDark, mainLight) |

## Troubleshooting

### Dashboard shows no data

1. **Check if Kaspa-Node IBD is complete:** The node needs to finish syncing before data is available. Check Docker logs for the Kaspa-Node container.
2. **Verify gRPC connectivity:** The `--rpclisten=0.0.0.0:16110` parameter must be in the node's PostArgs. Without it, gRPC only binds to 127.0.0.1 inside the container.
3. **Check config.json:** Ensure `wsApiURL` points to your Unraid server's actual IP, not `localhost`.
4. **Restart order:** If you restart containers, maintain the order: Node → Server → Client.

### gRPC Error 14 UNAVAILABLE

This error means the Monitor-Server cannot reach the Node's gRPC port. Common causes:
- Missing `--rpclisten=0.0.0.0:16110` in Kaspa-Node PostArgs
- Kaspa-Node not fully started yet
- Port 16110 not properly mapped in Docker

### Port conflicts

If ports 8124 or 2989 are already in use, change them in the respective template configuration.

## Storage Requirements

- **Blockchain data:** ~20-50 GB (grows over time)
- **Monitor data:** Minimal (<10 MB)

## Credits

- **Kaspa Node:** [kaspanet/rusty-kaspa](https://github.com/kaspanet/rusty-kaspa) — Rust implementation of the Kaspa protocol
- **KaspaNodeMonitor:** [imalfect/KaspaNodeMonitor](https://github.com/imalfect/KaspaNodeMonitor) — Node monitoring dashboard
- **Templates by:** rays23

## License

MIT License — see [LICENSE](LICENSE) for details.