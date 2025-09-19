# Bitcoin Core v30 Node Ban Script

A plug-and-play script to automatically ban/disconnect Bitcoin Core v30 (Satoshi:30) nodes from any Bitcoin node. Features easy one-command cron installation for continuous protection.

## Purpose

This script is not primarily about banning peers — it is a mirror held up to the
practice of filtering nodes by user agent. If banning is considered fair play,
it can be applied in both directions.

> “He who fights with monsters should look to it that he himself does not become a monster.”  
> — Friedrich Nietzsche

## Features

- **One-command cron installation** - Set it and forget it!
- Works with any Bitcoin Core/Knots node with RPC enabled
- Automatic detection of Core v30 nodes via subversion string (Satoshi:30)
- Option to ban or just disconnect nodes
- Configurable ban duration
- Dry-run mode for testing
- Support for both command-line arguments and config files
- Safe handling of different address formats (IPv4, IPv6, Tor)
- Built-in logging to track banning activity

## Requirements

- Bitcoin Core with RPC enabled
- `jq` (JSON processor)
- `bash`

## Prerequisites

## Accessing Your Node
You'll need SSH access to your Bitcoin node. If you're running:
- **Bitcoin Core**: `ssh [username]@[IP-ADDRESS]`
- **Umbrel**: `ssh umbrel@umbrel.local` (password: same as web interface)
- **Start9**: SSH must be enabled first in System → SSH → Add New Key, then `ssh start9@[NODE-NAME].local`
- **RaspiBlitz**: `ssh admin@[IP-ADDRESS]`
- **MyNode**: `ssh mynode@[IP-ADDRESS]`
- **RaspiBolt/Custom**: `ssh [username]@[IP-ADDRESS]`

Not sure how to SSH?
- **Mac/Linux**: Open Terminal and use `ssh username@node-ip`
- **Windows**: Use [PuTTY](https://putty.org/) or Windows Terminal

## Quick Start (Plug & Play)

**Option 1: Automatic detection (Simplest)**
```bash
# Automatically detects .cookie or bitcoin.conf file
wget https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh && \
chmod +x standalone-ban-core-v30.sh && \
./standalone-ban-core-v30.sh --install-cron
```

**Option 2: Manual credentials**
```bash
wget https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh && \
chmod +x standalone-ban-core-v30.sh && \
./standalone-ban-core-v30.sh -u yourrpcuser -P yourrpcpass --install-cron
```
*Start9 users: This is your option. Get credentials from Services → Bitcoin Core → Config → RPC Settings*

**Option 3: Custom config file (Usually not needed)**
```bash
# Only needed if auto-detection fails or you need custom settings
# The script already reads your bitcoin.conf automatically
cat > ~/.bitcoin/ban-core.conf << EOF
rpc_user=yourrpcuser
rpc_password=yourrpcpass
rpc_host=127.0.0.1
rpc_port=8332
EOF

# Install with cron
./standalone-ban-core-v30.sh -c ~/.bitcoin/ban-core.conf --install-cron
```

**Option 4: One-line install on Umbrel**
```bash
# Use manual credentials with --umbrel flag
wget https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh && \
chmod +x standalone-ban-core-v30.sh && \
./standalone-ban-core-v30.sh -u yourrpcuser -P yourrpcpass --umbrel --install-cron
```


That's it! The script will now run every 10 minutes automatically.

## Verification

Always verify the download before running it.

**1) SHA-256**
```bash
shasum -a 256 standalone-ban-core-v30.sh
```
Compare the output with the checksum shown on the v1.0 release page.

**2) GPG signature**
```bash
# Option A: fetch the public key from the release assets
curl -LO https://github.com/cmyk/Ban-Core/releases/download/v1.0/ban-core-gpg-public.asc
gpg --import ban-core-gpg-public.asc

# Option B: fetch from a keyserver
gpg --keyserver keyserver.ubuntu.com --recv-keys 6E3806FC

# Verify the signature
curl -LO https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh.asc
gpg --verify standalone-ban-core-v30.sh.asc
```
If it says `Good signature`, you’re good.


## Full Installation Steps

1. Download the script:
```bash
wget https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh
# or
curl -LO https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh
```

2. Make it executable:
```bash
chmod +x standalone-ban-core-v30.sh
```

3. Ensure you have `jq` installed:
```bash
# Ubuntu/Debian
sudo apt-get install jq

# macOS
brew install jq

# RHEL/CentOS/Fedora
sudo yum install jq
```

## Authentication Methods

The script supports multiple authentication methods, tried in this order:

1. **Cookie Authentication (Default)** - Automatically finds and uses Bitcoin Core's `.cookie` file
2. **Manual Credentials** - Specify with `-u` and `-P` flags
3. **Bitcoin Config** - Reads from `bitcoin.conf` if no cookie found
4. **Custom Config** - Use `-c` to specify a ban-core.conf file

For most users, the script works without any authentication parameters!

## Usage

### Basic Usage

```bash
# Ban all Core v30 nodes (uses cookie auth automatically)
./standalone-ban-core-v30.sh

# With manual credentials
./standalone-ban-core-v30.sh -u yourrpcuser -P yourrpcpassword

# Disconnect only (no ban)
./standalone-ban-core-v30.sh --disconnect-only

# Dry run (see what would happen without doing it)
./standalone-ban-core-v30.sh --dry-run

# Specify custom cookie path
./standalone-ban-core-v30.sh --cookie-path /custom/path/.cookie
```

### Cron Installation (Automatic)

```bash
# Ensure you have crontab installed (if not already)
sudo apt-get update && sudo apt-get install cron
sudo systemctl enable cron && sudo systemctl start cron

# Install with default 10-minute interval (auto-detects auth)
./standalone-ban-core-v30.sh --install-cron

# Or with manual credentials
./standalone-ban-core-v30.sh -u user -P pass --install-cron

# Install with custom interval (e.g., every 5 minutes)
./standalone-ban-core-v30.sh -u user -P pass --install-cron --cron-interval 5

# Install using config file
./standalone-ban-core-v30.sh -c ~/.bitcoin/ban-core.conf --install-cron

# View installed cron job
crontab -l

# Check logs
tail -f /tmp/ban-core.log

# Remove cron job
./standalone-ban-core-v30.sh --uninstall-cron
```

### Advanced Usage

```bash
# Custom RPC host and port
./standalone-ban-core-v30.sh -h 192.168.1.100 -p 8332 -u user -P pass

# Custom ban duration (30 days in seconds)
./standalone-ban-core-v30.sh -u user -P pass -d 2592000

# Using a config file
./standalone-ban-core-v30.sh -c ~/.bitcoin/ban-core.conf
```

### Command Line Options

- `-h, --host HOST` - RPC host (default: 127.0.0.1)
- `-p, --port PORT` - RPC port (default: 8332)
- `-u, --user USER` - RPC username (overrides auto-detection)
- `-P, --password PASS` - RPC password (overrides auto-detection)
- `--cookie-path PATH` - Path to .cookie file (default: auto-detect)
- `-d, --duration SECONDS` - Ban duration in seconds (default: 5 years)
- `-D, --disconnect-only` - Only disconnect, don't ban
- `-n, --dry-run` - Show what would be done without doing it
- `-c, --config FILE` - Read settings from config file
- `--install-cron` - Install as cron job (runs every 10 minutes)
- `--uninstall-cron` - Remove cron job
- `--cron-interval MIN` - Set cron interval in minutes (default: 10)
- `--umbrel` - Enable Umbrel compatibility
- `-H, --help` - Show help message

## Configuration File Format

Create a config file with the following format:

```ini
rpc_host=127.0.0.1
rpc_port=8332
rpc_user=yourusername
rpc_password=yourpassword
ban_duration=157680000
disconnect_only=false
```

## Automation

The script includes built-in cron support for easy automation. Simply use `--install-cron`!

### systemd Timer

1. Create service file `/etc/systemd/system/ban-core.service`:
```ini
[Unit]
Description=Ban Bitcoin Core v30 nodes
After=bitcoind.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/standalone-ban-core-v30.sh -c /etc/bitcoin/ban-core.conf
User=bitcoin
StandardOutput=journal
StandardError=journal
```

2. Create timer file `/etc/systemd/system/ban-core.timer`:
```ini
[Unit]
Description=Run ban-core every hour
Requires=ban-core.service

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

3. Enable and start:
```bash
sudo systemctl enable ban-core.timer
sudo systemctl start ban-core.timer
```

## Security Considerations

- Store RPC credentials securely
- Use config files with restricted permissions (600)
- Consider running with limited user privileges
- Review ban list periodically

## How It Works

1. Connects to Bitcoin Core via RPC
2. Retrieves all peer information
3. Filters peers with "Satoshi:30" in their subversion string
4. For each Core v30 peer:
   - Extracts the base IP address
   - Disconnects the node
   - Optionally bans the IP address

## Platform-Specific Guides

### Start9 Detailed Setup

Start9 runs Bitcoin Core in a podman container. The script automatically detects this and handles it for you.

**Warning**: Start9 has a read-only filesystem. The script and cron job will be removed after system updates or restarts. You'll need to reinstall after each Start9 update. For a permanent solution, consider running the script from an external system that connects to your Start9 node.

1. **Enable SSH Access**:
   - Go to System → SSH → Add New Key
   - Add your SSH public key
   - SSH into your node: `ssh start9@[NODE-NAME].local`

2. **Get RPC Credentials**:
   - Open Start9 web interface
   - Navigate to: Services → Bitcoin Core → Config → RPC Settings
   - Copy your RPC username and password

3. **Test Connection** (optional):
   ```bash
   # Check if you can see your peers
   sudo podman exec -it bitcoind.embassy bitcoin-cli getpeerinfo
   ```

4. **Download and Run the Script**:
   ```bash
   # Download the script
   wget https://github.com/cmyk/Ban-Core/releases/download/v1.0/standalone-ban-core-v30.sh
   chmod +x standalone-ban-core-v30.sh
   
   # Run with your credentials
   ./standalone-ban-core-v30.sh -u <your-rpc-username> -P <your-rpc-password>
   ```
   The script will automatically detect Start9 and use podman to execute commands.

5. **Set Up Automation**:
   ```bash
   # Create config file with your credentials
   mkdir -p ~/.bitcoin
   echo "rpc_user=<your-username>" > ~/.bitcoin/ban-core.conf
   echo "rpc_password=<your-password>" >> ~/.bitcoin/ban-core.conf
   
   # Install cron job
   ./standalone-ban-core-v30.sh -c ~/.bitcoin/ban-core.conf --install-cron
   ```

### Umbrel Notes

Umbrel users should use the `--umbrel` flag to ensure proper Docker container execution. The script will automatically handle the container communication.

## Troubleshooting

### Cannot connect to Bitcoin node
- Verify RPC is enabled in bitcoin.conf
- Check RPC credentials
- Ensure firewall allows connection
- Verify Bitcoin Core is running

### Start9-specific issues
- **Script not detecting Start9**: Ensure you're running the script directly on Start9, not from a remote machine
- **Permission denied**: Use `sudo` when running the script if needed
- **Container not found**: Verify Bitcoin Core is running: `sudo podman ps | grep bitcoin`
- **Test manually**: `sudo podman exec -it bitcoind.embassy bitcoin-cli getpeerinfo | grep -i "Satoshi:30"`
- **Script disappears after restart/update**: Start9 has a read-only filesystem. Need to reinstall after each update/restart or run from an external system:
   ```bash
   # From external machine with Tor installed
   ./standalone-ban-core-v30.sh \
      -h YOUR_START9_TOR_ADDRESS.onion \
      -p 8332 \
      -u YOUR_RPC_USERNAME \
      -P YOUR_RPC_PASSWORD \
      --install-cron
   ```
   Get your Tor address from: Services -> Bitcoin Core -> Interfaces -> Tor

### jq: command not found
Install jq using your package manager (see Installation section)

### Ban fails
- Node may already be banned
- Check Bitcoin Core logs
- Verify you have sufficient RPC permissions

## Credits

This project is based on the excellent work in [Ban-Knots](https://github.com/noosphere888/Ban-Knots) by @noosphere888.  
Thanks to the original author for providing the foundation.

## License

This script is released into the public domain. Use at your own risk.
