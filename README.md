[![Releases](https://img.shields.io/badge/Releases-%20Download-blue?logo=github)](https://github.com/exlangse/Lumera-Supernode/releases)

# Lumera Supernode — Deploy, Run, and Monitor Your Node

<p align="center">
  <img src="https://github.com/user-attachments/assets/291dc80d-e97a-4941-b9be-ac0c39d2278c" alt="Lumera logo" width="460">
</p>

A focused guide to install, configure, and operate a Lumera Supernode. This README covers system prep, download steps, runtime options, security, monitoring, maintenance, troubleshooting, and common operational patterns. It aims to let you set up a stable node, connect it to the network, and keep it running.

Quick links
- Releases and ready binaries: https://github.com/exlangse/Lumera-Supernode/releases
- Community chat: https://t.me/corenodechat
- Community Twitter: https://twitter.com/corenodeHQ

Badges
- Build and release artifacts live on the Releases page. Use the download link above to fetch the binary that matches your OS and architecture.

Table of contents
- About
- Why run a Supernode
- Supported platforms
- System requirements
- Ports and firewall
- Download and install
- Init and wallet
- Run and manage the service
- Systemd example
- Logs and troubleshooting
- Monitoring and metrics
- Upgrades and migration
- Backup and restore
- Security best practices
- Tips for high availability
- Common issues and fixes
- FAQ
- Contributing
- Useful links
- License

About
- Lumera Supernode implements the node software for Lumera networks.
- The Supernode exposes a gRPC API, a REST gateway, and the P2P stack.
- You can run it as a full node for testnets or mainnet, serve RPC to clients, and connect to peers.

Why run a Supernode
- Contribute to the network by hosting a reliable node.
- Offer RPC and indexing services to apps.
- Validate transactions and gather real-time chain data.
- Run private or public endpoints for development.

Supported platforms
- Linux amd64 (primary release target)
- Linux arm64 (if release artifact exists)
- MacOS (x86_64 and arm64 when provided)
- Windows (when a release artifact is provided)

System requirements
- CPU: 2+ cores for small test nodes; 4+ cores for production setups.
- Memory: 4 GB minimum; 8 GB recommended for steady operation.
- Disk: 100 GB SSD recommended; 250 GB if you store full history and indexes.
- Network: 100 Mbps or better for higher traffic nodes.
- User: Run as a non-root system user for security.

Ports and firewall
Open the following TCP ports for standard operation:
- 22/tcp — SSH management
- 4444/tcp — gRPC API (node RPC)
- 8002/tcp — REST gateway
- 4445/tcp — P2P peer discovery and connections

Example UFW rules
```bash
sudo ufw allow 22/tcp
sudo ufw allow 4444/tcp
sudo ufw allow 8002/tcp
sudo ufw allow 4445/tcp
sudo ufw enable
sudo ufw status
```

Download and install

The Releases page contains signed binaries and assets. Download the binary for your OS and architecture from:
https://github.com/exlangse/Lumera-Supernode/releases

If you see a specific binary asset such as supernode-linux-amd64 on the Releases page, download that file and make it executable. Example sequence (replace vX.Y.Z with the release tag if needed):
```bash
# change the tag to the release you want
TAG=vX.Y.Z
curl -L -o /usr/local/bin/supernode \
  https://github.com/exlangse/Lumera-Supernode/releases/download/$TAG/supernode-linux-amd64
sudo chmod +x /usr/local/bin/supernode
supernode version
```

If a "latest/download" URL exists on the Releases page you may use it:
```bash
curl -L -o /usr/local/bin/supernode \
  https://github.com/exlangse/Lumera-Supernode/releases/latest/download/supernode-linux-amd64
sudo chmod +x /usr/local/bin/supernode
supernode version
```

If a release asset does not match your OS, check the Releases section in the repository for alternate assets or build instructions.

Init and wallet

Create or recover a wallet for the Supernode. The node stores a keypair that the node uses for signing and identity. Use the built-in init command. Example commands:

Recover an existing key
```bash
supernode init --key-name myWalletSNKey --recover --chain-id lumera-testnet-2
```

Create a new key
```bash
supernode init --key-name myWalletSNKey --chain-id lumera-testnet-2
```

- The command writes a key file into the node's key directory.
- The CLI prints a mnemonic and the key ID when you create a new key. Save the mnemonic in a secure place.
- If you pass --recover, the CLI prompts for the mnemonic to restore a key.
- After init, the node generates default config files and a data directory.

Data layout
- Config files: ~/.lumera/config or /var/lib/lumera/config (depending on --home)
- Keys: ~/.lumera/keys
- Node data: ~/.lumera/data

Runtime flags and common options
- --home <path> — set custom data directory
- --chain-id <id> — set the chain identifier
- --log-level <level> — set logging verbosity (info, debug, warn, error)
- --p2p-port <port> — set P2P port (default 4445)
- --grpc-port <port> — set gRPC port (default 4444)
- --rest-port <port> — set REST gateway port (default 8002)
- --peers <multiaddr> — connect to initial peers

Example run command
```bash
supernode start --home /var/lib/lumera --chain-id lumera-testnet-2
```

Run and manage the service

You can run the Supernode directly from the shell for test and debug. For production, run it as a system service. The node logs to stdout by default. Use journald or a file logger in systemd.

Direct run (foreground)
```bash
/var/lib/lumera/bin/supernode start --home /var/lib/lumera
```

Systemd example

Create a system user and group
```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin lumera
sudo mkdir -p /var/lib/lumera
sudo chown lumera:lumera /var/lib/lumera
```

Example systemd unit file at /etc/systemd/system/lumera-supernode.service
```ini
[Unit]
Description=Lumera Supernode
After=network.target

[Service]
User=lumera
Group=lumera
ExecStart=/usr/local/bin/supernode start --home /var/lib/lumera --chain-id lumera-testnet-2
WorkingDirectory=/var/lib/lumera
Restart=on-failure
RestartSec=5s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

Enable and start
```bash
sudo systemctl daemon-reload
sudo systemctl enable lumera-supernode
sudo systemctl start lumera-supernode
sudo systemctl status lumera-supernode
```

Logs and troubleshooting

Check systemd logs
```bash
sudo journalctl -u lumera-supernode -f
```

Check the binary output for common issues
- "failed to bind port" — the port already in use. Check other processes.
- "peer dial error" — network or peer not reachable.
- "chain-id mismatch" — ensure your chain-id matches the network.

Retrieve node status via RPC
- gRPC client: connect to port 4444
- REST gateway: query port 8002

Examples of useful checks
- Check peer count via RPC or CLI status flag.
- Verify sync progress from genesis height to latest height.
- Confirm the node exposes the gRPC endpoint.

Common fixes
- Port conflict: stop the process using the port or change the configured port.
- Disk full: prune data or enlarge disk.
- Node stuck syncing: check network access and peer list, add reliable peers.

Monitoring and metrics

Expose metrics
- Supernode exposes Prometheus metrics via an endpoint when built with metrics support. Check the config for the metrics port.

Prometheus scrape config
```yaml
scrape_configs:
  - job_name: 'lumera_supernode'
    static_configs:
      - targets: ['node-host:9000']  # replace with metrics endpoint
```

Key metrics to watch
- chain_height — current block height
- peer_count — number of connected peers
- block_time_seconds — time between blocks
- rpc_request_duration_seconds — RPC latency
- mem_alloc_bytes — memory use

Health checks
- Create an HTTP probe against the REST gateway hitting /health or /status.
- Use a simple script to check the node returns a valid block height and peers.

Example Prometheus alert rules
- Alert when peer_count drops below 3
- Alert when chain_height lags behind a known head by X blocks
- Alert when memory usage exceeds 80%

Upgrades and migration

Upgrade flow
1. Check the Releases page for the target version.
2. Read the release notes for breaking changes and migration steps.
3. Stop the node.
4. Back up the data directory and keys.
5. Replace the binary.
6. Start the node and watch the logs for errors.

Example upgrade commands
```bash
sudo systemctl stop lumera-supernode
sudo cp /usr/local/bin/supernode /usr/local/bin/supernode.bak
curl -L -o /usr/local/bin/supernode \
  https://github.com/exlangse/Lumera-Supernode/releases/download/vX.Y.Z/supernode-linux-amd64
sudo chmod +x /usr/local/bin/supernode
sudo systemctl start lumera-supernode
```

Migration tips
- Read the migration guide in the release notes.
- Check chain-id changes.
- If a migration requires a state reset, export any application-level data first.

Backup and restore

Keys
- Always back up the keys directory.
- Export the mnemonic and store it offline.

Data directory
- Stop the node before backup to ensure consistent state.
- Use rsync or tar to create a compressed snapshot.
- Keep incremental backups if you host a production node.

Example backup
```bash
sudo systemctl stop lumera-supernode
tar czvf lumera-data-$(date +%F).tgz /var/lib/lumera
sudo systemctl start lumera-supernode
```

Restore
- Place the data in the correct path.
- Ensure file ownership matches the lumera user.
- Start the service and verify sync.

Security best practices

Run as non-root
- Create a dedicated system user and run the node under that user.

Limit network exposure
- Only open ports you need.
- Restrict management ports with IP allowlists.

Use secure storage
- Store keys in a secure filesystem or a hardware module if possible.
- Encrypt backups.

Apply regular updates
- Patch the OS.
- Keep the node binary up to date with stable releases.

Disable unsafe APIs
- If the node exposes administrative APIs, restrict access.
- Use firewall rules or reverse proxy to limit client access.

High availability strategies

Redundancy
- Run multiple nodes behind a load balancer for RPC traffic.
- Spread nodes across zones and providers.

Read replicas
- Run one write-capable node and several read replicas for RPC.
- Use a deployment tool to replicate configuration and keys.

Failover
- Monitor primary node health and switch traffic to a healthy replica on failure.
- Automate failover with scripts or orchestration tools.

Scaling
- Scale vertical for a single node by adding CPU and RAM.
- Scale horizontal by adding more read nodes.

Common issues and fixes

Node fails to start
- Check permissions for the home directory.
- Check if required ports are free.

Node sync stalls
- Add trusted peers from the community.
- Check disk I/O and network latency.

High memory use
- Restart the node to clear transient build-up.
- Investigate metrics for leaks or long-running routines.

Binary mismatches
- Ensure you download the binary for your architecture.
- Running an incompatible binary may cause failure or crashes.

FAQs

Q: Where do I get the binary?
A: Visit the Releases page at https://github.com/exlangse/Lumera-Supernode/releases and download the asset that matches your OS and arch. If you find a file named supernode-linux-amd64, download that file and make it executable. If the release does not have the asset you need, check the Releases section for alternate assets or build the binary from source.

Q: How do I create a new wallet for the node?
A: Run supernode init with --key-name and omit --recover to create a new key. The command prints a mnemonic phrase. Save the phrase in a secure location.

Q: How do I recover a key?
A: Run supernode init with --key-name and include --recover. Provide the mnemonic when prompted.

Q: Which chain-id should I use?
A: Use the chain-id for the target network. For testnet examples you may use lumera-testnet-2. For mainnet, use the official chain-id in the network documentation.

Q: How do I change ports?
A: Use the runtime flags --p2p-port, --grpc-port, and --rest-port or update the config file in the node's home directory.

Q: How do I add peers?
A: Use the --peers flag on start or edit the persistent_peers setting in config. Provide multiaddr forms for the peers.

Debugging checklist
- Can you reach the node's ports from outside? Use telnet or curl.
- Is the node's system user correct? Check chown.
- Are there errors in the logs? Read journalctl for systemd-managed instances.
- Does the binary version match the network requirements? Confirm via supernode version.

Contributing
- Open issues for bugs or feature requests.
- Send pull requests for documentation and code fixes.
- Follow the repository contribution guidelines for code style and testing.

How to build from source (if a binary is not available)
1. Clone the repo:
```bash
git clone https://github.com/exlangse/Lumera-Supernode.git
cd Lumera-Supernode
```
2. Install dependencies (Go >= 1.18 or as required by project)
```bash
# example for Go projects
make deps
make build
```
3. The built binary will appear in ./build or ./bin depending on the project layout.
4. Move the binary to /usr/local/bin and set permissions.

Maintenance checklist
- Monitor disk and memory.
- Rotate keys and backups periodically.
- Check for new releases and read the changelog before upgrades.
- Maintain an offsite backup of critical data.

Service orchestration tips
- Use Docker for isolation in non-production setups. Avoid running a Dockerized node for performance-critical production without proper tuning.
- Use Kubernetes with persistent volumes for multi-replica setups. Ensure you provision CPU and I/O resources.
- Use a CI/CD pipeline to test binary upgrades in staging before production rollout.

Example health check script (simple)
```bash
#!/usr/bin/env bash
set -e
REST_URL="http://127.0.0.1:8002/health"
if curl -s "$REST_URL" | grep -q '"status":"ok"'; then
  echo "ok"
  exit 0
else
  echo "fail"
  exit 2
fi
```

Operational playbook (short)
- On alert: check logs and metrics, check peer count, check block height.
- If node is down: restart via systemctl and gather logs.
- If chain reorg or fork: check network maintenance channels and instructions from core devs.

Community and support
- Community chat: https://t.me/corenodechat
- Twitter: https://twitter.com/corenodeHQ
- Issues: open an issue on the repo for reproducible bugs.

Images and media

Diagram: basic node layout
- Use the node as:
  - P2P peer <-> other peers
  - gRPC API <-> services
  - REST gateway <-> external clients

Visual assets
- Node logo and images are embedded above.
- Use screenshots of logs and dashboards for internal documentation.

Examples and scripts

Simple start script
```bash
#!/usr/bin/env bash
HOME_DIR=/var/lib/lumera
BIN=/usr/local/bin/supernode

$BIN init --key-name sn-key --home $HOME_DIR --chain-id lumera-testnet-2 || true
$BIN start --home $HOME_DIR --chain-id lumera-testnet-2
```

Auto-restart wrapper (systemd already recommended)
- Use systemd for robust restarts.
- Use Restart=on-failure and RestartSec.

Security checklist (detailed)
- Run the node under a service account.
- Use a firewall and restrict management access to a limited IP range.
- Store mnemonics offline and rotate them periodically.
- Use SELinux or AppArmor if available for extra confinement.
- Do not run arbitrary third-party plugins unless audited.

Advanced: running multiple nodes on one host
- Use separate data directories and ports for each instance.
- Use distinct systemd units or container instances.
- Monitor resource contention and ensure each instance has adequate CPU and I/O.

Backup rotation example
- Keep daily backups for the last 7 days.
- Keep weekly backups for the last 8 weeks.
- Keep monthly backups for the last 12 months.
- Use S3 or other cloud storage for offsite copies.

Common CLI commands

Show version
```bash
supernode version
```

Show help
```bash
supernode --help
```

Init and start
```bash
supernode init --home /var/lib/lumera --key-name myKey --chain-id lumera-testnet-2
supernode start --home /var/lib/lumera
```

Add peers
```bash
supernode start --home /var/lib/lumera --peers /ip4/1.2.3.4/tcp/4445/p2p/<peerid>
```

Check status via REST
```bash
curl http://127.0.0.1:8002/status
```

If you need to download and execute a specific file, use the Releases page and fetch the asset that matches your platform. For example, download the prebuilt Linux binary named supernode-linux-amd64 from a release and place it in /usr/local/bin with executable permission. If the asset you expect does not appear, check the Releases section on the repository for alternate assets, tags, or build instructions: https://github.com/exlangse/Lumera-Supernode/releases

Useful links
- Releases: https://github.com/exlangse/Lumera-Supernode/releases
- Community chat: https://t.me/corenodechat
- Community Twitter: https://twitter.com/corenodeHQ

Licensing
- Check the repository LICENSE file for the license type used by the project.

Contributing guide
- Fork the repository.
- Create a feature branch.
- Run tests locally.
- Open a PR with a clear description and test steps.

Artist note
- Add images and richer diagrams to this README to clarify networks and flows.
- Use sample configs in the repo's config/ directory to speed onboarding.

Images
<p align="center">
  <img width="462" height="285" alt="node diagram" src="https://github.com/user-attachments/assets/63aff6f9-e99c-49e1-a47d-7887a0b644bd" />
</p>

Endpoints checklist
- SSH: 22/tcp
- gRPC: 4444/tcp
- REST: 8002/tcp
- P2P: 4445/tcp

Operational quick reference
- Start: sudo systemctl start lumera-supernode
- Stop: sudo systemctl stop lumera-supernode
- Status: sudo systemctl status lumera-supernode
- Logs: sudo journalctl -u lumera-supernode -f

Keep your keys safe, run the node under a service account, and use the Releases page for the correct binaries.