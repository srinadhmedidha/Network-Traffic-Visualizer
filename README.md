# Network Traffic Visualizer & Real-Time Threat Detector

A desktop application that captures live network traffic, visualizes protocol activity in real time, and flags suspicious behavior such as port scans, ARP spoofing, and malicious payload patterns. Built with Python, Scapy, Tkinter, and Matplotlib.

> **Administrator / root privileges are required.** Packet sniffing (TCP, UDP, ICMP, ARP) only works with elevated permissions. See [Installation & Running](#installation--running).

---

## Table of Contents

- [Features](#features)
- [Threat Detection Capabilities](#threat-detection-capabilities)
- [Screenshots](#screenshots)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation & Running](#installation--running)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Known Issues](#known-issues)
- [Limitations and Future Improvements](#limitations-and-future-improvements)
- [Disclaimer](#disclaimer)

## Features

- **Live packet capture**: sniffs packets in real time using Scapy and shows a scrolling feed with timestamp, source, destination, and protocol summary.
- **Live protocol distribution graph**: a Matplotlib chart (dark/neon theme) showing cumulative TCP, UDP, and ICMP packet counts over time.
- **Real-time threat alerts**: every packet is passed through a rule-based detection engine, and alerts appear instantly in a dedicated panel.
- **Capture controls**: Start, Stop, and Pause the capture at any time.
- **CSV export**: save captured packets (timestamp, source IP, destination IP, protocol, length, summary, alert) for later analysis.
- **Login and dashboard**: simple login screen and home dashboard for navigation.
- **IPv4 and IPv6 traffic display**, including ICMPv6 and ARP packets.

## Threat Detection Capabilities

Implemented in `detector.py`:

| Detection | Description |
|---|---|
| TCP SYN scan | Many SYN packets to many distinct ports from one source within a time window |
| UDP scan | High rate of UDP packets to diverse ports from one source |
| Suspicious TCP flags | FIN, NULL, and XMAS scan patterns |
| ICMP flood | Excessive ICMP packets from a single source |
| ARP spoofing | An IP address suddenly claimed by a different MAC address |
| Payload signatures | Basic directory traversal and SQL injection patterns in HTTP-style traffic |
| Suspicious user agents | Tools like `sqlmap`, `masscan`, and Nmap Scripting Engine in HTTP traffic |
| Known bad IPs | Traffic to or from IPs on a configurable blocklist |

All thresholds, time windows, signatures, and the IP blocklist are configurable in the `CONFIG` dictionary at the top of `detector.py`.

## Screenshots

### Login Page
![Login page](screenshots/login.png)

### Dashboard
![Dashboard](screenshots/dashboard.png)

### Live Traffic Analysis
Live packet feed, threat alerts panel, capture controls, and the live protocol distribution graph.

![Live output](screenshots/live-output.png)

### Exported CSV Output
![Exported CSV](screenshots/exported-output.png)

## Project Structure

```
.
├── main.py          # Entry point; launches the application
├── visualizer.py    # GUI (login, dashboard, live traffic page), packet capture, graphing, CSV export
├── detector.py      # Rule-based threat detection engine
├── README.md
└── screenshots/     # Images used in this README
```

## Requirements

- Python 3.8 or newer (with Tkinter)
- [Scapy](https://scapy.net/)
- [Matplotlib](https://matplotlib.org/)
- **Windows only:** [Npcap](https://npcap.com/) (the packet capture driver)

Install the Python libraries:

```bash
pip install scapy matplotlib
```

## Installation & Running

First, clone the repository:

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### Windows

1. Install [Python](https://www.python.org/downloads/) and tick **Add python.exe to PATH** in the installer.
2. Install [Npcap](https://npcap.com/) and tick **Install Npcap in WinPcap API-compatible Mode**. Restart your terminal afterwards.
3. Open PowerShell **as Administrator** (right-click it in the Start menu, then **Run as administrator**).
4. Go to the project folder, install the libraries, and run:

   ```powershell
   cd <path-to-project-folder>
   py -m pip install scapy matplotlib
   py main.py
   ```

   If `py` is not recognized, use `python` instead.

### macOS

1. Install Python from [python.org](https://www.python.org/downloads/macos/). Its installer includes a modern Tk, which Apple's built-in Python lacks.
2. In the project folder, create an environment and install the libraries:

   ```bash
   python3 -m venv ~/venvs/nettraffic
   source ~/venvs/nettraffic/bin/activate
   python3 -m pip install scapy matplotlib
   ```

3. Run with `sudo`, using the environment's Python:

   ```bash
   sudo ~/venvs/nettraffic/bin/python main.py
   ```

   Keep the virtual environment on your internal disk (not on an external drive) to avoid the issue described in [Troubleshooting](#troubleshooting).

### Linux

```bash
sudo apt install python3-tk        # if Tkinter is missing
python3 -m pip install scapy matplotlib
sudo python3 main.py
```

### Login

Use the demo credentials:

- **Username:** `admin`
- **Password:** `password`

> The login uses hardcoded demo credentials to show the UI flow. It is not a real authentication system.

## Usage

1. Log in and click **Start Live Traffic Analysis**.
2. Click **Start** to begin capturing.

| Button | Action |
|---|---|
| **Start** | Begin capturing packets |
| **Stop** | Stop capturing |
| **Pause** | Pause/resume updates without ending the session |
| **Export CSV** | Save captured packets and any alerts to a CSV file |
| **< Back to Home** | Return to the dashboard (stops capture automatically) |

Up to 20,000 packets are kept in memory for export, and the graph shows a rolling window of the most recent data points.

## How It Works

1. `main.py` creates the Tkinter window and launches the `ApplicationController` from `visualizer.py`.
2. When capture starts, Scapy's `sniff()` runs in a background thread so the GUI stays responsive.
3. Each packet is counted by protocol, displayed in the live feed, and passed to `analyze_packet()` in `detector.py`.
4. Any alerts returned are displayed in the threat alerts panel.
5. Once per second, the graph updates with cumulative per-protocol counts.

## Troubleshooting

| Problem | Fix |
|---|---|
| `python: command not found` (macOS) | Use `python3` instead of `python`. |
| `Python was not found; run without arguments to install from the Microsoft Store` (Windows) | Use `py` instead of `python`, or reinstall Python with **Add python.exe to PATH** ticked. |
| `No libpcap provider is available` (Windows) | Install [Npcap](https://npcap.com/) with **WinPcap API-compatible Mode** enabled, then restart the terminal. |
| No packets appear / sniffing fails | Run the terminal as Administrator (Windows) or use `sudo` (macOS/Linux). |
| `ModuleNotFoundError: No module named 'matplotlib'` (or `scapy`) | Install the libraries with the same Python you run the app with: `python -m pip install scapy matplotlib`. |
| `running scripts is disabled on this system` when activating a virtual environment (PowerShell) | Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, or skip activation and call the environment's `python.exe` directly. |
| `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb0` when the project or virtual environment is on an external drive (macOS) | macOS creates hidden `._*` files on exFAT/NTFS drives, and Python tries to read them. Create the virtual environment on your internal disk (for example `~/venvs`). |
| Blank or crashing window on macOS | Apple's built-in Python ships an old Tk. Install Python from python.org and recreate the virtual environment. |

## Known Issues

- Under some traffic patterns (for example, a UDP packet from a source to a new destination port) the detector can raise a `KeyError`, which makes Scapy print `Socket ... failed with ... It was closed.` and stop capturing. This comes from replacing a `defaultdict` with a plain `dict` while pruning old timestamps in `detect_udp_scans` and `detect_tcp_scans`.
- Exporting to CSV re-runs the detector on stored packets, so alerts in the export may differ from the live alerts.

## Limitations and Future Improvements

- Detection is rule/threshold based and may produce false positives or miss sophisticated attacks.
- The detector analyzes IPv4 traffic only; IPv6 packets are shown but not checked.
- TCP connect scan detection is not yet implemented.
- ARP scan detection and automatic state cleanup are marked as TODOs.
- The Settings page is a placeholder.
- Possible improvements: network interface selection, alert cooldowns, a config file for detection rules, PCAP import/export, alert logging to file, proper authentication, and more protocol coverage.

## Disclaimer

This tool is intended for **educational purposes and for monitoring networks you own or have explicit permission to analyze**. Capturing traffic on networks without authorization may be illegal. The author is not responsible for any misuse.

## Author

Srinadh Medidha [GitHub profile](https://github.com/srinadhmedidha)