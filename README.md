# fumitm (MITM Certificate Fixer Upper)

Script to automatically verify and fix TLS certificate trust issues caused by zero-trust MITM proxy solutions (Cloudflare WARP, NetSkope, etc.) that perform TLS inspection.

## Supported Providers

| Provider | Status | Platform |
|----------|--------|----------|
| Cloudflare WARP | ✅ Supported | macOS, Linux, Windows |
| NetSkope | 🔜 Planned | - |

## Usage

### Linux/macOS

```bash
# Download the script
curl -LsSf https://raw.githubusercontent.com/aberoham/fumitm/main/fumitm.py -o fumitm.py
chmod +x ./fumitm.py

# Check status (no changes made)
./fumitm.py

# Apply fixes
./fumitm.py --fix

# Run with detailed debug output (useful for troubleshooting)
./fumitm.py --debug
```

### Windows

```powershell
# Download the Windows-specific script
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/aberoham/fumitm/main/fumitm_windows.py" -OutFile "fumitm_windows.py"

# Check status (no changes made)
python fumitm_windows.py

# Apply fixes to all supported tools
python fumitm_windows.py --fix

# Fix only specific tools (can specify multiple)
python fumitm_windows.py --fix --tools node --tools python
python fumitm_windows.py --fix --tools node-npm,gcloud

# List all available tools and their tags
python fumitm_windows.py --list-tools

# Run with detailed debug/verbose output (useful for troubleshooting)
python fumitm_windows.py --debug
python fumitm_windows.py --verbose

# Show help and all available commands
python fumitm_windows.py --help
```

#### Windows Command Line Options

- `-h, --help` - Show help message and exit
- `--fix` - Actually make changes (default is status check only)
- `--tools, --tool TOOLS` - Specific tools to check/fix (can be specified multiple times)
  - Examples: `--tools node --tools python` or `--tools node,gcloud`
- `--list-tools` - List all available tools and their tags
- `--debug, --verbose` - Show detailed debug information

## FU MITM Rationale

When your organization runs a zero-trust MITM proxy (like Cloudflare WARP Gateway or NetSkope) with TLS inspection enabled, the proxy intercepts and records virtually all HTTPS traffic for policy enforcement and security auditing. These tools achieve this introspection by presenting their own root certificate to your TLS clients -- essentially performing a sanctioned man-in-the-middle (MITM) attack on your TLS (aka SSL) connections.

Typically, macOS and Windows themselves will automatically trust these certificates through system keychains. Most third-party development tools completely ignore these system certificates. Each tool maintains its own certificate bundle or looks for specific environment variables. This fragmentation creates endless annoying "certificate verify failed" errors across your toolchain whenever TLS inspection is turned on.

One particularly annoying detail is that simply pointing tools to your organization's MITM certificate by itself rarely works. You often need to append the custom CA to an existing bundle of public CAs, which quickly becomes a brittle process that needs repeating for each tool.

FU MITM!

## Don't Disable Your Proxy

Whilst the quick temporary workaround might be to toggle your MITM proxy OFF, this is incredibly distressing to any nearby Information Security professionals who will one day need to forensically examine dodgy dependencies or MCPs that have slipped onto your laptop.

The act of toggling your proxy off also seriously hints that you have no clue what you're doing, as understanding TLS certificate-based trust is a critical concept underpinning modern vibe'n.

## Requirements

### General
- Python 3 (macOS, Linux, Windows)
- Your MITM proxy must be installed and connected

### Cloudflare WARP (current provider)
- Cloudflare WARP client must be installed and connected
- `warp-cli` command must be available (macOS/Linux) or `warp-cli.exe` (Windows)
- Administrator privileges may be required for some fixes on Windows

## Contribute

Something amiss or not quite right? Please post the full output of a run to an issue or simply submit a PR

## List of supported fixes

### Linux/macOS
- **Node.js/npm**: configures `NODE_EXTRA_CA_CERTS` for Node.js and the cafile setting for npm
- **Python**: sets the `REQUESTS_CA_BUNDLE`, `SSL_CERT_FILE`, and `CURL_CA_BUNDLE` environment variables
- **gcloud**: configures the `core/custom_ca_certs_file` for the Google Cloud `gcloud` CLI
- **Git**: configures Git to use the custom certificate bundle via `http.sslCAInfo`
- **curl**: configures `CURL_CA_BUNDLE` environment variable for curl
- **Java/JVM**: adds the Cloudflare certificate to any found Java keystore (cacerts)
- **jenv**: adds the Cloudflare certificate to all jenv-managed Java installations
- **DBeaver**: targets the bundled JRE and adds the certificate to its keystore
- **wget**: configures the `ca_certificate` in the `.wgetrc` file
- **Podman**: installs certificate in `~/.docker/certs.d/` (persistent) and Podman VM's trust store (if running)
- **Rancher Desktop**: installs certificate in `~/.docker/certs.d/` (persistent) and Rancher VM's trust store (if running)
- **Colima**: installs certificate in `~/.docker/certs.d/` (persistent, applied on start) and Colima VM's trust store (if running)
- **Android Emulator**: helps install certificate on running Android emulators
- **Gradle**: sets `systemProp` entries in `gradle.properties` (respecting `GRADLE_USER_HOME`) for the WARP certificate.
 
### Windows
- **Node.js/npm**: configures `NODE_EXTRA_CA_CERTS` for Node.js and the cafile setting for npm
- **Python**: sets the `REQUESTS_CA_BUNDLE`, `SSL_CERT_FILE`, and `CURL_CA_BUNDLE` environment variables
- **Google Cloud SDK (gcloud)**: configures the `core/custom_ca_certs_file` for the Google Cloud `gcloud` CLI
- **Java/JVM**: adds the Cloudflare certificate to any found Java keystore (cacerts)
- **wget**: configures the `ca_certificate` in the `.wgetrc` file
- **Podman**: installs certificate in Podman container runtime
- **Rancher Desktop**: installs certificate in Rancher Desktop Kubernetes environment
- **Git**: configures Git to use the custom certificate bundle via `http.sslCAInfo`
- **Windows Certificate Store**: installs the certificate in the Windows system certificate store

#### Windows-Specific Notes

The Windows version (`fumitm_windows.py`) includes Windows-specific functionality:

- Uses Windows Registry to locate certificates and configuration
- Handles Windows paths and file permissions
- Works with Windows-specific certificate stores
- Supports PowerShell environment variable management

### VS Code Devcontainers / WSL

Fumitm should auto-detect VS Code devcontainers and WSL environments where `warp-cli` is only available on the underlying host. Within these environments, fumitm will guide the user where to obtain their Cloudflare cert and will skip slow verification tests.

Fumitm should auto-detect WSL environments where `warp-cli` is only available on the underlying Windows host. Within WSL, fumitm will guide the user where to obtain their Cloudflare cert and will skip slow verification tests.

## Installation Alternative

You can also run the script directly from the repository:

### Linux/macOS
```bash
# Clone the repository
git clone https://github.com/aberoham/fumitm.git
cd fumitm

# Run the script
./fumitm.py --fix
```

### Windows
```powershell
# Clone the repository
git clone https://github.com/aberoham/fumitm.git
cd fumitm

# Run the Windows-specific script
python fumitm_windows.py --fix
```

## Troubleshooting

If you encounter issues:

1. Ensure WARP is connected: `warp-cli status`
2. Run with debug output: `./fumitm.py --debug` (Linux/macOS) or `python fumitm_windows.py --debug` (Windows)
3. Check that Python 3 is properly installed and in your PATH
4. Verify you have appropriate permissions for the tools you're trying to fix

