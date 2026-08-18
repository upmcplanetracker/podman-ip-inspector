podman-ip-inspector
===================

An interactive CLI tool to display Podman networks, subnets, containers, and their IP addresses - with color, tree view, usage stats, and duplicate subnet warnings.

Overview
--------

`podman-ip-inspector` is a Bash script that queries your Podman environment and presents a clear overview of all user-defined networks, their subnets, attached containers, and the IP addresses assigned to each container (both IPv4 and IPv6). It works with **rootless** Podman (default) and **rootful** Podman (if you run with `sudo`).

The output can be shown as:

*   a **detailed table** (default),
*   an **ASCII tree** (with `-m`), and
*   an **IPv4 usage summary** (with `-u`).

All views are **color‑coded** for quick visual scanning, and a **duplicate subnet warning** is displayed automatically to help you avoid routing conflicts.

You can also opt in to extra columns/details per network or container - **driver** (e.g. `bridge` vs `pasta`), **gateway**, **DNS-enabled**, **internal-only**, and container **MAC address** - without cluttering the default view.

Features
--------

*   List all networks - shows network names, subnets, and attached containers.
*   IP extraction - shows both IPv4 and IPv6 addresses for every container.
*   Tree view (`-m`) - hierarchical view of networks with containers indented.
*   Usage summary (`-u`) - shows how many IPv4 addresses are used vs. available per network.
*   Extra columns (opt-in) - network driver, gateway, DNS-enabled, internal-only, and container MAC address.
*   Color output - network names (yellow), container names (green), IPv4 (magenta), IPv6 (cyan).
*   Duplicate subnet detection - warns if the same subnet is used on multiple networks.
*   Filter by network (`-n`) or container (`-c`) - show only what you need.
*   Works with rootless & rootful - run normally for rootless, or with `sudo` to inspect rootful containers.
*   No extra dependencies - requires only `podman`, `jq`, and optionally `perl` (for colors).

Installation
------------

1.  Save the script (e.g., `podman-ip-inspector`) in your `$PATH` (e.g., `~/.local/bin/`).
2.  Make it executable:
    
        chmod +x podman-ip-inspector
    
3.  Ensure dependencies are installed:
    
        sudo apt install jq perl     # Debian/Ubuntu
        sudo dnf install jq perl     # Fedora/RHEL
    
4.  Run it:
    
        ./podman-ip-inspector
    

Usage
-----

    podman-ip-inspector [OPTIONS]
    
    Options:
      -h, --help              Show this help message
      -n, --network NAME      Show only the specified network
      -c, --container NAME    Show only the specified container
      -m, --map               Show an ASCII tree of networks and containers
      -u, --usage             Show IPv4 usage summary (used/total) per network
      -d, --driver            Show network driver column (e.g. bridge, pasta)
      -g, --gateway           Show gateway column
          --dns               Show DNS-enabled flag column
          --internal          Show internal-only flag column
          --mac               Show container MAC address alongside its IP
      -a, --all-extra         Enable driver, gateway, dns, internal, and mac columns
          --color             Force color output (auto-detected by default)
          --no-color          Disable color output
    
    If both -n and -c are given, show only that container on that specific network.
    If no options are given, show a detailed table with the core columns only.
    Extra columns (driver/gateway/dns/internal/mac) are off by default; opt in with
    their flags individually, or use -a/--all-extra for all of them at once.

Examples
--------

### 1\. Full table view (default)

    podman-ip-inspector

Sample output:

    NETWORK NAME        SUBNET(S)                           CONTAINERS & IPs
    ------------        --------                            ----------------
    adsb.network        10.89.0.0/24, fd52:317c:ee6:631::/64  airspy (10.89.0.2, fd52:317c:ee6:631::2)
    adsb.network        10.89.0.0/24, fd52:317c:ee6:631::/64  dump978 (10.89.0.8, fd52:317c:ee6:631::8)
    ...

### 2\. Tree view (`-m`)

    podman-ip-inspector -m

Sample output:

    Network: adsb.network (10.89.0.0/24, fd52:317c:ee6:631::/64)
       ├── airspy         10.89.0.2, fd52:317c:ee6:631::2
       ├── dump978        10.89.0.8, fd52:317c:ee6:631::8
       └── ...
    Network: plex.network (10.89.1.0/24, fd52:317c:ee6:632::/64)
       └── plex           10.89.1.7, fd52:317c:ee6:632::7

### 3\. Usage summary (`-u`)

    podman-ip-inspector -u

Outputs the table plus:

    IPv4 Usage Summary (used / total available):
    ---------------------------------------------
    adsb.network: 10.89.0.0/24   Used: 11/254   Free: 243
    plex.network: 10.89.1.0/24   Used: 1/254    Free: 253
    ...

### 4\. Filter by network (`-n`)

    podman-ip-inspector -n plex.network -m -u

### 5\. Filter by container (`-c`)

    podman-ip-inspector -c plex -m

### 6\. Rootful Podman

    sudo podman-ip-inspector

### 7\. Driver and gateway columns (`-d -g`)

    podman-ip-inspector -d -g

Sample output:

    NETWORK NAME  DRIVER  GATEWAY    SUBNET(S)     CONTAINERS & IPs
    ------------  ------  -------    --------      ----------------
    adsb.network  bridge  10.89.0.1  10.89.0.0/24  airspy (10.89.0.2)
    pasta.network pasta                            plex (10.89.1.7)

Useful for spotting at a glance which networks use `bridge` vs `pasta` (or `macvlan`, etc.), since driver choice affects how containers get routed and whether they get real LAN IPs.

### 8\. All extra columns (`-a`)

    podman-ip-inspector -a

Adds driver, gateway, DNS-enabled, internal-only, and MAC address all at once - table columns become `DRIVER | GATEWAY | DNS | INTERNAL`, and container MAC addresses are appended in brackets next to their IPs.

### 9\. Extra info in tree view

    podman-ip-inspector -m -a

Sample output:

    📡  adsb.network (10.89.0.0/24) [bridge] gw:10.89.0.1 dns:yes
       └── airspy 10.89.0.2 [aa:bb:cc:dd:ee:01]
    📡  pasta.network () [pasta] gw: dns:no internal
       └── plex 10.89.1.7 [aa:bb:cc:dd:ee:02]

Color Coding
-------------

| Element          | Color |
|------------------|--------|
| Network name     | Yellow |
| Container name   | Green  |
| IPv4 address     | Magenta|
| IPv6 address     | Cyan   |

Colors are **auto‑detected** if your terminal supports them. You can force `--color` or disable with `--no-color`.

Notes
-----

*   **Rootless vs. Rootful**: The script uses the `podman` command from the user's environment. When run without `sudo`, it shows **rootless** containers. When run with `sudo`, it shows **rootful** containers. All features work identically in both modes.
*   **Dependencies**: `jq` is required for parsing JSON output. `perl` is optional but recommended for coloring; if absent, colors are automatically disabled.
*   **Duplicate subnet warning**: If two networks share the same subnet (e.g., both use `10.89.0.0/24`), a warning is printed. This is a common misconfiguration that can cause routing issues especially upon startup due to race conditions.
*   **IPv6 support**: Both IPv4 and IPv6 addresses are shown. The usage summary only counts IPv4 addresses for simplicity.
*   **Pasta networks and subnets**: `pasta`-driven networks often don't report a shared `subnets`/gateway the same way `bridge` networks do, since pasta does per-container NAT rather than routing through a shared bridge subnet. It's normal to see blank subnet/gateway columns for a `pasta` network even though the driver column correctly reports `pasta`.
