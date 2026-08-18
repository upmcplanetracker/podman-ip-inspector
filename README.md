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

Features
--------

*   List all networks – shows network names, subnets, and attached containers.
*   IP extraction – shows both IPv4 and IPv6 addresses for every container.
*   Tree view (`-m`) – hierarchical view of networks with containers indented.
*   Usage summary (`-u`) – shows how many IPv4 addresses are used vs. available per network.
*   Color output – network names (yellow), container names (green), IPv4 (magenta), IPv6 (cyan).
*   Duplicate subnet detection – warns if the same subnet is used on multiple networks.
*   Filter by network (`-n`) or container (`-c`) – show only what you need.
*   Works with rootless & rootful – run normally for rootless, or with `sudo` to inspect rootful containers.
*   No extra dependencies – requires only `podman`, `jq`, and optionally `perl` (for colours).

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
          --color             Force color output (auto-detected by default)
          --no-color          Disable colour output
    
    If both -n and -c are given, show only that container on that specific network.
    If no options are given, show a detailed table.

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

Color Coding
-------------

| Element          | Colour |
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

* * *

Generated for GitHub – use this HTML if you prefer a web page instead of markdown.
