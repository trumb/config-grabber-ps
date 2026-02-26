# config-grabber-ps

A **PowerShell** utility that SSH's into network switches, runs a list of commands, and saves the output to timestamped text files — one file per device.

Uses the **[Posh-SSH](https://github.com/darkoperator/Posh-SSH)** module for SSH connectivity, supporting both password and SSH key authentication.

> Looking for the Python version? → [config-grabber-public](https://github.com/trumb/config-grabber-public)

## Features

- Connect to **one or many switches** via IP address or a text file list
- **Per-device port support** using `IP:PORT` notation for PAT/NAT environments
- **Password authentication** — prompted securely via `Read-Host -AsSecureString`
- **SSH key authentication** via `-KeyFile`
- **Enable / privileged EXEC mode** support (Cisco-style) via `-Enable`
- Output saved as **`<IP>_<YYYY-MM-DDTHHMMSS>.txt`** per device
- Graceful error handling — a failed device is logged and skipped
- Full `-Verbose` support for debug-level output

## Requirements

- **Windows PowerShell 5.1** or **PowerShell 7+**
- **[Posh-SSH](https://www.powershellgallery.com/packages/Posh-SSH)** module

### Install Posh-SSH

```powershell
Install-Module -Name Posh-SSH -Scope CurrentUser
```

## Usage

```powershell
.\config-grabber.ps1 [-IP <string> | -DeviceFile <path>]
                     -CommandFile <path>
                     -Username <string>
                     [-Password <SecureString>]
                     [-KeyFile <path>]
                     [-Enable]
                     [-Port <int>]
                     [-Timeout <int>]
                     [-OutputDir <path>]
                     [-Verbose]
```

### Parameters

| Parameter | Description |
|---|---|
| `-IP "IP[,IP,...]"` | Single IP or comma-separated list. Supports `IP:PORT` notation. |
| `-DeviceFile <path>` | Text file with one IP per line. Supports `IP:PORT` per line. |
| `-CommandFile <path>` | **Required.** File with commands to run (one per line). |
| `-Username <string>` | **Required.** SSH login username. |
| `-Password <SecureString>` | SSH password. Prompted securely if omitted. |
| `-KeyFile <path>` | SSH private key file for key-based auth. |
| `-Enable` | Enter Cisco privileged EXEC mode after login. |
| `-Port <int>` | Default SSH port (default: `22`). Per-device ports override this. |
| `-Timeout <int>` | Connection timeout in seconds (default: `30`). |
| `-OutputDir <path>` | Output directory (default: current directory). |
| `-Verbose` | Show debug-level logging. |

## Examples

### Single switch — password prompted

```powershell
.\config-grabber.ps1 -IP 192.168.1.1 -CommandFile examples\commands.txt -Username admin
```

### Multiple switches from a file

```powershell
.\config-grabber.ps1 -DeviceFile examples\devices.txt `
    -CommandFile examples\commands.txt `
    -Username admin -OutputDir .\output
```

### SSH key authentication

```powershell
.\config-grabber.ps1 -DeviceFile examples\devices.txt `
    -CommandFile examples\commands.txt `
    -Username admin -KeyFile C:\Users\me\.ssh\id_rsa -OutputDir .\output
```

### Enable mode (Cisco IOS)

```powershell
.\config-grabber.ps1 -DeviceFile examples\devices.txt `
    -CommandFile examples\commands.txt `
    -Username admin -Enable -OutputDir .\output
```

### PAT/NAT — per-device ports

```powershell
# Each device on a different external port
.\config-grabber.ps1 -IP "203.0.113.1:2221,203.0.113.1:2222" `
    -CommandFile examples\commands.txt -Username admin -OutputDir .\output
```

### PAT/NAT — mixed ports in a file

`examples\devices.txt`:
```
# Standard port 22
192.168.1.1

# PAT'd - custom port per device
10.0.0.1:2221
10.0.0.2:2222
```

```powershell
.\config-grabber.ps1 -DeviceFile examples\devices.txt `
    -CommandFile examples\commands.txt -Username admin
```

## Input File Formats

### devices.txt

```
# Lines starting with '#' are comments
# Blank lines are ignored

# Standard SSH port (uses -Port default)
192.168.1.1
192.168.1.2

# PAT'd devices
10.0.0.1:2221
10.0.0.2:2222
```

### commands.txt

```
# Cisco IOS example — one command per line
terminal length 0
show version
show ip interface brief
show running-config
```

## Output

Each device produces one `.txt` file:

```
output\
├── 192.168.1.1_2026-02-26T101930.txt
├── 10.0.0.1_2026-02-26T101936.txt
```

Each file starts with:

```
# Config Grabber Output (PowerShell)
# Device  : 192.168.1.1
# Captured: 2026-02-26T10:19:30.0000000-08:00
# ============================================================

Switch>terminal length 0
Switch>show version
...
```

## Exit Codes

| Code | Meaning |
|---|---|
| `0` | All devices processed successfully |
| `1` | One or more devices failed |

## Security Notes

- Passwords entered via `Read-Host -AsSecureString` are **never stored in shell history**
- Enable passwords are held as `SecureString` in memory; the plain-text form exists for only milliseconds before being zeroed
- `AcceptKey = $true` auto-accepts SSH host keys (equivalent to `AutoAddPolicy`). For high-security environments, disable this and manage a known-hosts file manually

## Project Structure

```
config-grabber-ps\
├── config-grabber.ps1          # Main script
├── README.md                   # This file
├── .gitignore
├── config-grabber-ps.code-workspace
└── examples\
    ├── devices.txt             # Example IP list
    └── commands.txt            # Example Cisco IOS commands
```
