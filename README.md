# GNS3 Network Simulator MCP Server v2.0

[![MCP Protocol](https://img.shields.io/badge/MCP-Protocol-blue.svg)](https://modelcontextprotocol.io/)
[![FastMCP](https://img.shields.io/badge/FastMCP-2.12.0-green.svg)](https://github.com/anselmholden/fastmcp)
[![Python](https://img.shields.io/badge/Python-3.10--3.13-yellow.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-red.svg)](LICENSE)
[![GNS3](https://img.shields.io/badge/GNS3-Compatible-orange.svg)](https://gns3.com/)

An MCP (Model Context Protocol) server that exposes 40+ tools for AI-driven
GNS3 network simulation: topology creation, device configuration, and
simulation management through natural language.

---

## What's New in v2.0

- **40+ tools** (up from 12): complete GNS3 API coverage
- **Modular architecture**: clean separation of concerns
- **Configuration templates**: 15+ pre-built network configs (OSPF, BGP, VLANs, NAT, etc.)
- **Advanced console control**: enhanced Telnet with auto-detection and config management
- **Bulk operations**: configure multiple devices simultaneously
- **Topology validation**: automated network health checks
- **Snapshot management**: version control for projects
- **Drawing tools**: annotate topologies with text and shapes

---

## Feature Matrix

| Category | Tools | Capabilities | Use Cases |
|----------|-------|--------------|-----------|
| Server & Compute | 2 tools | Server info, compute management | Infrastructure monitoring |
| Project Management | 8 tools | Full lifecycle management | Organization, backups |
| Node Management | 13 tools | Complete device control | Device deployment, management |
| Link Management | 3 tools | Connection management | Topology building |
| Configuration | 3 tools | Console access, templates | Device setup, automation |
| Templates | 2 tools | Template & appliance management | Rapid deployment |
| Snapshots | 4 tools | Version control | Backup, restore |
| Packet Capture | 2 tools | Traffic analysis | Troubleshooting, monitoring |
| Drawing | 2 tools | Topology annotation | Documentation |
| Advanced | 3 tools | Bulk ops, validation | Enterprise operations |

---

## Installation & Setup

### Prerequisites

- **GNS3 Server** running on `http://localhost:3080` (default)
- **Python 3.10 – 3.13** installed (`python --version`)
  - Tested and working on **Python 3.13**.
  - **Python 3.14 is not supported** — it currently fails during dependency
    installation. Use 3.13 (or any 3.10–3.13) until this is resolved.
- An MCP-compatible client — **[Claude Code CLI](https://docs.claude.com/en/docs/claude-code)** (recommended) or Gemini CLI

---

### Managing Multiple Python Versions

If your default `python` is a version this project doesn't support (for example
**3.14**), you don't need to uninstall it. You can keep several versions side by
side and pick the right one (3.13) only when creating the virtual environment.
Once the `.venv` is created with the correct interpreter, every later command
(`pip`, `python -m gns3_mcp.server`, Claude Code) automatically uses it.

#### Windows — the `py` launcher

Windows ships a launcher called `py` that knows about every installed version.

```powershell
# List the versions you have installed (the * marks the default)
py --list
#  -V:3.14 *        Python 3.14 (64-bit)
#  -V:3.13          Python 3.13 (64-bit)
#  -V:3.11          Python 3.11 (64-bit)

# Run a specific version without changing the default
py -3.13 --version          # -> Python 3.13.x

# Create the project's venv with 3.13 (this is the key step)
py -3.13 -m venv .venv
```

**Installing another version:** download it from [python.org/downloads](https://www.python.org/downloads/)
(or `winget install Python.Python.3.13`). The `py` launcher registers it
automatically — no need to touch the `PATH`. To avoid the 3.14 problem entirely
you can also just install 3.13 and leave 3.14 alone.

#### Linux / macOS

Installed versions are usually exposed as version-suffixed binaries:

```bash
# Check what's available
python3.13 --version
which python3.13

# Create the venv with a specific version
python3.13 -m venv .venv
```

If `python3.13` isn't installed:

- **Ubuntu/Debian:** `sudo apt install python3.13 python3.13-venv`
  (older releases may need the deadsnakes PPA)
- **macOS (Homebrew):** `brew install python@3.13`

For juggling many versions cleanly, **[pyenv](https://github.com/pyenv/pyenv)**
is the standard tool:

```bash
pyenv install 3.13            # download & build 3.13
pyenv local 3.13             # pin 3.13 for this folder (writes .python-version)
python -m venv .venv         # now 'python' is 3.13 here
```

#### Confirm the venv is using the right version

After activating the `.venv`, `python` always points at the version you created
it with — verify before installing:

```bash
python --version             # should report 3.13.x, not 3.14.x
```

If it reports the wrong version, delete the `.venv` folder and recreate it with
the explicit interpreter (`py -3.13 -m venv .venv` / `python3.13 -m venv .venv`).

---

### Install with Claude Code CLI (Recommended)

This is the setup used to develop the project. Three steps:

#### 1. Clone and set up the Python environment

```bash
# Clone the repository (use your own fork if you have one)
git clone https://github.com/wael-rd/gns3-mcp-server.git
cd gns3-mcp-server

# Make sure you are on Python 3.10-3.13 (3.14 will fail):
python --version

# Create an isolated virtual environment and install the package.
# If your default python is 3.14, create the venv with 3.13 explicitly
# (see "Managing Multiple Python Versions" above):
python -m venv .venv          # if 'python' is already 3.10-3.13
#   py -3.13 -m venv .venv     # Windows, when default is 3.14
#   python3.13 -m venv .venv   # Linux / macOS

# Activate it
#   Windows (PowerShell):
.venv\Scripts\Activate.ps1
#   Linux / macOS:
source .venv/bin/activate

# Install the server and its dependencies into the venv
pip install -e .
```

> The MCP server runs as the module `gns3_mcp.server`. Installing with
> `pip install -e .` registers the package so it can be launched with
> `python -m gns3_mcp.server`.

#### 2. Register the server with Claude Code

Point Claude at the Python interpreter **inside the `.venv`** so it always uses
the right dependencies. Use the absolute path to your clone:

```bash
# Windows (PowerShell) - literal path to .venv\Scripts\python.exe
claude mcp add gns3 -- "C:\path\to\gns3-mcp-server\.venv\Scripts\python.exe" -m gns3_mcp.server

# Linux / macOS
claude mcp add gns3 -- "/path/to/gns3-mcp-server/.venv/bin/python" -m gns3_mcp.server
```

Optional - pass GNS3 connection settings as environment variables (only needed
if your GNS3 server isn't on `http://localhost:3080` or requires authentication):

```bash
claude mcp add gns3 \
  -e GNS3_SERVER_URL=http://192.168.1.100:3080 \
  -e GNS3_USERNAME=admin \
  -e GNS3_PASSWORD=secret \
  -- "C:\path\to\gns3-mcp-server\.venv\Scripts\python.exe" -m gns3_mcp.server
```

Scope flags (where the server is registered):

- `--scope user` — available in all your projects (how this repo is set up)
- `--scope project` — shared with your team via a committed `.mcp.json`
- `--scope local` *(default)* — only the current directory

#### 3. Verify the connection

```bash
claude mcp list
```

You should see:

```
gns3: C:\...\gns3-mcp-server\.venv\Scripts\python.exe -m gns3_mcp.server - Connected
```

Now just talk to Claude:

```
> List all my GNS3 projects
> Create a project called "Lab1" with two routers connected together
```

To remove or reconfigure later: `claude mcp remove gns3`, then add it again.

---

### Install with Claude Desktop (Alternative)

Claude Desktop is configured through a JSON file instead of a command. First
complete step 1 above (clone the repo and create the `.venv` with `pip install -e .`).

#### 1. Open the config file

In Claude Desktop go to **Settings → Developer → Edit Config**, or open the file
directly:

- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Linux:** `~/.config/Claude/claude_desktop_config.json`

#### 2. Add the server under `mcpServers`

```json
{
  "mcpServers": {
    "gns3": {
      "command": "C:\\Users\\yuri\\mcp-servers\\gns3-mcp-server\\.venv\\Scripts\\python.exe",
      "args": [
        "-m",
        "gns3_mcp.server"
      ],
      "env": {
        "GNS3_SERVER_URL": "http://localhost:3080"
      }
    }
  }
}
```

**What to change:**

- **`command`** — replace with the absolute path to **your** clone's venv
  interpreter. Keep the doubled backslashes (`\\`) on Windows, since they are
  required inside JSON. On Linux/macOS use the forward-slash path instead, e.g.
  `"/home/you/gns3-mcp-server/.venv/bin/python"`.
- **`GNS3_SERVER_URL`** — point it at your GNS3 server. Leave
  `http://localhost:3080` if it runs locally with the default port.
- **Authentication (optional)** — if your GNS3 server requires it, add the
  credentials inside `env`:
  ```json
  "env": {
    "GNS3_SERVER_URL": "http://localhost:3080",
    "GNS3_USERNAME": "admin",
    "GNS3_PASSWORD": "secret"
  }
  ```

> If the file already has other servers under `mcpServers`, just add the `"gns3"`
> block alongside them (mind the commas between entries). If the file is empty,
> paste the whole snippet above.

#### 3. Restart Claude Desktop

Fully quit and reopen the app. The GNS3 tools appear under the tools icon once
the server connects.

---

### Install with Gemini CLI (Alternative)

```bash
# 1. Clone and install
git clone https://github.com/wael-rd/gns3-mcp-server.git
cd gns3-mcp-server
python -m pip install -e .

# 2. Add to Gemini CLI (uses the bundled launcher scripts)
gemini mcp add gns3 "path/to/gns3-mcp-server/run.bat"  # Windows
gemini mcp add gns3 "path/to/gns3-mcp-server/run.sh"   # Linux/Mac

# 3. Test the connection
gemini "List all GNS3 projects"
```

> The `run.bat` / `run.sh` launchers auto-create the `.venv` and install
> dependencies on first run.

---

## Available MCP Tools (40+)

### Quick Reference

See [docs/TOOL_REFERENCE.md](docs/TOOL_REFERENCE.md) for complete documentation
of all 42 tools.

### Key Tool Categories

#### Server & Compute Management (2 tools)
- `gns3_get_server_info` - Get GNS3 server version and information
- `gns3_list_computes` - List all available compute servers

#### Project Management (8 tools)
- `gns3_list_projects` - List all projects with status
- `gns3_create_project` - Create new projects
- `gns3_get_project` - Get project details
- `gns3_update_project` - Update project settings
- `gns3_open_project` - Open existing project
- `gns3_close_project` - Close project (stops nodes)
- `gns3_delete_project` - Permanently delete project
- `gns3_duplicate_project` - Copy project with new name

#### Node Management (13 tools)
- `gns3_list_nodes` - List all devices in project
- `gns3_add_node` - Add device from template
- `gns3_get_node` - Get device details
- `gns3_update_node` - Update device settings
- `gns3_delete_node` - Remove device
- `gns3_start_node` / `gns3_stop_node` - Control device state
- `gns3_suspend_node` / `gns3_reload_node` - Advanced control
- `gns3_duplicate_node` - Clone device
- `gns3_start_all_nodes` / `gns3_stop_all_nodes` - Bulk operations

#### Link Management (3 tools)
- `gns3_list_links` - List all connections
- `gns3_add_link` - Connect two devices
- `gns3_delete_link` - Remove connection

#### Topology Tools (1 tool)
- `gns3_get_topology` - Complete network overview

#### Console & Configuration (3 tools)
- `gns3_send_console_commands` - Send CLI commands to devices
- `gns3_get_node_config` - Get device configuration
- `gns3_apply_config_template` - Apply pre-built configurations

#### Template & Appliance (2 tools)
- `gns3_list_templates` - List available device templates
- `gns3_list_appliances` - List available appliances

#### Snapshot Management (4 tools)
- `gns3_list_snapshots` - List project snapshots
- `gns3_create_snapshot` - Create backup
- `gns3_restore_snapshot` - Restore from backup
- `gns3_delete_snapshot` - Delete snapshot

#### Packet Capture (2 tools)
- `gns3_start_capture` - Start packet capture on link
- `gns3_stop_capture` - Stop packet capture

#### Drawing & Annotation (2 tools)
- `gns3_add_text_annotation` - Add text labels
- `gns3_add_shape` - Add shapes (rectangle, ellipse)

#### Advanced Tools (3 tools)
- `gns3_get_idle_pc_values` - Optimize Dynamips routers
- `gns3_bulk_configure_nodes` - Configure multiple devices at once
- `gns3_validate_topology` - Check for common issues

---

## Usage Examples

### Example 1: Complete Network Setup

```
User: "Create a new project called 'Enterprise_WAN' with 3 sites connected via routers"

The assistant creates the project, lists available templates, adds 3 routers
(HQ, Branch1, Branch2), switches, and PCs, connects them with proper WAN and
LAN links, configures IP addressing and OSPF routing, and starts all devices.

User: "Configure OSPF on all routers with area 0"

The assistant applies the OSPF template to each router and saves the
configurations.
```

### Example 2: VLAN Configuration on a Layer 3 Switch

```
User: "Set up VLANs 10, 20, 30 on my switch for Sales, Engineering, and Management"

The assistant creates the VLANs with names, configures the trunk port to the
router, sets access ports for each department, and configures inter-VLAN routing.

User: "Show me the running configuration"

The assistant uses gns3_get_node_config to retrieve and display the config.
```

### Example 3: Network Troubleshooting

```
User: "My network between HQ and Branch1 has connectivity issues"

The assistant runs gns3_validate_topology, checks node status, starts a packet
capture on the WAN link, sends diagnostic commands to the routers, and analyzes
the routing tables.

The assistant: "Issue found: Interface GigabitEthernet0/1 on HQ router is
administratively down."

User: "Fix it please"

The assistant sends 'no shutdown' and verifies connectivity with ping tests.
```

### Example 4: Bulk Configuration Deployment

```
User: "Configure all routers with SSH access, username 'admin', password 'cisco123'"

The assistant uses gns3_list_nodes to find all routers, then
gns3_bulk_configure_nodes with the SSH template to configure the domain, crypto
keys, and VTY lines on each, returning a per-device status.
```

---

## Configuration Templates Library

The server includes 15+ pre-built, tested configuration templates.

### Routing Protocols
- **OSPF**: single/multi-area, router-id, network statements
- **EIGRP**: AS configuration, auto-summary control
- **BGP**: eBGP/iBGP, neighbor configuration, route reflectors
- **Static Routes**: standard and default routes

### Switching
- **VLANs**: creation and naming
- **Trunk Ports**: 802.1Q encapsulation, allowed VLANs
- **Access Ports**: VLAN assignment, PortFast, BPDU Guard

### Services
- **DHCP**: pool configuration, DNS, excluded addresses
- **NAT/PAT**: overload configuration, ACLs
- **SSH**: secure access with crypto keys

### Security
- **Standard ACLs**: simple permit/deny rules
- **Extended ACLs**: protocol, port-based filtering
- **Basic Hardening**: service disabling, password encryption

### Management
- **Basic Router Setup**: hostname, domain, console settings
- **Interface Configuration**: IP addressing, descriptions
- **Logging**: syslog configuration
- **SNMP**: community strings and access control
- **NTP**: time synchronization
- **Banners**: MOTD and login messages

### Quality of Service
- **QoS Marking**: DSCP marking, class maps, policy maps

**Usage Example:**

```python
# Apply OSPF routing
gns3_apply_config_template(
    node_id="router-id",
    template_name="ospf",
    template_params={
        "process_id": 1,
        "router_id": "1.1.1.1",
        "networks": [
            {"network": "192.168.1.0", "wildcard": "0.0.0.255", "area": 0},
            {"network": "10.0.0.0", "wildcard": "0.0.0.3", "area": 0}
        ]
    },
    save_config=True
)
```

---

## Advanced Configuration

### Environment Variables

```bash
# Set custom GNS3 server
export GNS3_SERVER_URL="http://192.168.1.100:3080"

# Configure authentication
export GNS3_USERNAME="admin"
export GNS3_PASSWORD="secure_password"

# SSL/TLS settings
export GNS3_VERIFY_SSL="false"
```

---

## Troubleshooting

### Connection failed
- Check the GNS3 server is running.
- Verify the server URL (`GNS3_SERVER_URL`).
- Check firewall settings.

### Device template not found
- Verify device templates are installed in GNS3 (import them via the GNS3 GUI).

### Authentication failed
- Check `GNS3_USERNAME` / `GNS3_PASSWORD`.

### Dependency installation fails
- Confirm you are on **Python 3.10–3.13**. **Python 3.14 is not supported** and
  fails during install — recreate the venv with a 3.13 interpreter:
  ```bash
  py -3.13 -m venv .venv        # Windows (with the Python launcher)
  python3.13 -m venv .venv      # Linux / macOS
  ```

### Claude Code: server shows "Failed to connect"
```bash
# 1. Confirm the venv python path is correct and absolute:
claude mcp get gns3

# 2. Make sure the package is installed INTO that venv:
.venv\Scripts\python.exe -m gns3_mcp.server   # should start without ModuleNotFoundError
#   If it errors, re-run:  .venv\Scripts\python.exe -m pip install -e .

# 3. Re-add the server if the path was wrong:
claude mcp remove gns3
claude mcp add gns3 -- "C:\path\to\gns3-mcp-server\.venv\Scripts\python.exe" -m gns3_mcp.server
```

### Debug logging
```bash
export GNS3_MCP_DEBUG=1
```

---

## Technical Architecture

### System Components

```
+---------------+      +-------------------+      +---------------+
|   AI Client   |<---->|  GNS3 MCP Server  |<---->|  GNS3 Server  |
|               |      |                   |      |               |
| - AI Interface|      | - 40+ MCP Tools   |      | - REST API    |
| - Tool        |      | - Async Client    |      | - WebSocket   |
|   Discovery   |      | - Error Handling  |      | - Real-time   |
| - JSON-RPC    |      |                   |      |               |
+---------------+      +-------------------+      +---------------+
```

### Protocol Flow

1. **Tool Discovery**: the MCP client discovers all available tools.
2. **Request Processing**: a user request is mapped to a specific MCP tool.
3. **API Translation**: the MCP tool converts the request to a GNS3 REST API call.
4. **Response Processing**: the GNS3 response is transformed into a readable result.
5. **Real-time Updates**: WebSocket connections provide live status updates.

---

## Supported Platforms

- Windows 10/11 (x64)
- macOS 10.15+ (Intel / Apple Silicon)
- Ubuntu 18.04+ (x64 / ARM64)
- CentOS 7/8 (x64)

---

## Contributing

Contributions are welcome. Please see the [Contributing Guide](CONTRIBUTING.md)
for details.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file
for details.

---

## Acknowledgments

- **GNS3 Team** - for the network simulation platform
- **FastMCP** - for the MCP framework
- **Model Context Protocol** - for the open standard enabling AI tool use
