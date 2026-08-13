# WiTechTools

> **PowerShell 7 helpdesk and security toolkit for WiTech technicians.**
> Version 4.4.0 · 68 commands · 60 shortcuts

Every command prints a colour-coded table in your terminal, so you can read the answer at a glance instead of squinting at raw output. Most commands are read-only and safe to run on a user's machine while they watch.

**New here?** Read [Getting Started](#getting-started), then [Common Scenarios](#common-scenarios). You do not need to memorise 68 commands — you need to know which four to reach for when someone says "the internet is slow".

**Been here a while?** Jump to the [Quick Reference](#quick-reference) or the [Command Reference](#command-reference).

---

## Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [How to Read This Guide](#how-to-read-this-guide)
- [Quick Reference](#quick-reference)
- [Common Scenarios](#common-scenarios)
- [Command Reference](#command-reference)
  - [Network & Connectivity](#network--connectivity)
  - [Network Diagnostics](#network-diagnostics)
  - [System Health & Performance](#system-health--performance)
  - [Security & Threat Hunting](#security--threat-hunting)
  - [Reporting & Documentation](#reporting--documentation)
  - [Active Directory](#active-directory)
  - [Files, Media & Module Admin](#files-media--module-admin)
- [Logging](#logging)
- [Troubleshooting](#troubleshooting)
- [Known Issues](#known-issues)
- [Running the Tests](#running-the-tests)
- [Project Layout](#project-layout)

---

## Requirements

| Requirement | Detail |
|---|---|
| **PowerShell** | **7.0 or newer.** This is not the blue "Windows PowerShell" 5.1 that ships with Windows — it is a separate install. Run `pwsh` to launch it. The module will refuse to load on 5.1. |
| **Operating system** | Windows 10 or Windows 11 |
| **Permissions** | Most commands run as a standard user. Some need an elevated terminal — see [How to Read This Guide](#how-to-read-this-guide). |

### Optional extras

These are only needed for specific commands. Everything else works without them.

| Extra | Needed by | Notes |
|---|---|---|
| **FFmpeg** on `PATH` | `Set-ImageToPng`, `Set-ImageToJpg` | Image and video conversion |
| **`VIRUSTOTAL_API_KEY`** environment variable | `Get-MalwareInfo` | Or pass `-ApiKey` directly |
| **RSAT Active Directory tools** | `Get-ADUserInfo`, `Unlock-ADUser` | Only on domain-joined machines |
| **Internet access** | `Get-IPInfo`, `Invoke-SpeedTest`, `Invoke-FallbackSpeedTest`, `Get-Download`, `Get-MalwareInfo`, `Get-Network` (vendor lookup only) | |
| **WinRM enabled on the target** | Any command used with `-ComputerName` pointing at another machine | |

---

## Installation

1. **Copy the module folder** to your PowerShell 7 modules directory:

   ```
   C:\Users\<YourName>\Documents\PowerShell\Modules\WiTechTools\
   ```

   > If your Documents folder is redirected to OneDrive, use the OneDrive path — that is the real Documents folder. To find it, run:
   > ```powershell
   > [Environment]::GetFolderPath('MyDocuments')
   > ```

2. **Check the folder contains** these items:

   ```
   WiTechTools.psd1
   WiTechTools.psm1
   PSScriptAnalyzerSettings.psd1
   Public\
   ```

3. **Import it:**

   ```powershell
   Import-Module WiTechTools
   ```

4. **Make it load automatically** in every new terminal by adding that line to your profile:

   ```powershell
   Add-Content $PROFILE 'Import-Module WiTechTools'
   ```

On first import, the module registers a **WiTech** source in the Windows Application event log. If you are not running as administrator the first time, that registration is skipped with a warning — the module still works, it just cannot write to the event log until it is registered once from an elevated terminal.

---

## Getting Started

Verify the install:

```powershell
Get-Module WiTechTools          # should report version 4.4.0
cmds                            # lists every command and its shortcut
```

Try three safe, read-only commands:

```powershell
gip          # your public IP, ISP and location
gds          # disk space on every drive
gnet         # everything alive on the local network
```

Every command has built-in help:

```powershell
Get-Help Get-Network -Full        # everything
Get-Help Get-Network -Examples    # just the examples
```

### If a command is not found

The module is not loaded in that terminal. Run `Import-Module WiTechTools`.

### Reloading after an update

```powershell
Import-Module WiTechTools -Force
```

---

## How to Read This Guide

**Shortcuts (aliases).** Nearly every command has a short form. `Get-Network` and `gnet` are the same command — use whichever you prefer. Eleven commands have no shortcut; those are marked `—`.

**The Admin column** tells you whether you need an elevated terminal (right-click PowerShell 7 → *Run as administrator*):

| Marker | Meaning |
|:---:|---|
| **Yes** | The command **checks** for admin rights. Without them it prints a clear warning and stops — nothing half-finished. |
| **Yes\*** | The command **needs** admin rights but does **not** check. Without them, expect confusing errors or silently incomplete results. This marker is our assessment from reading the code, not something the command enforces. |
| **—** | Runs fine as a standard user. |

**Parameter tables** list every option. Anything marked *required* must be supplied; everything else has a default shown.

**A note on switches.** A parameter marked `switch` is an on/off flag — you write `-PortScan`, not `-PortScan $true`.

**Previewing destructive commands (`-WhatIf` / `-Confirm`).** The six commands that delete files, kill processes, or reset services — `Invoke-DeepDiskCleanup`, `Clear-TempFiles`, `Clear-BrowserCache`, `Invoke-ProcessGuard -AutoKill`, `Reset-NetworkAdapter`, and `Reset-PrintSpooler` — accept two standard safety switches:

- `-WhatIf` lists exactly what the command *would* do and changes nothing. Run it first when you are unsure.
- `-Confirm` prompts you to approve each action before it happens.

By design they do **not** prompt unless you ask, so existing scripts and the TAYi console keep working unchanged.

```powershell
Invoke-DeepDiskCleanup -WhatIf     # show what would be deleted, delete nothing
Clear-TempFiles -Confirm           # ask before clearing each temp folder
```

---

## Quick Reference

Run `cmds` to print this list in your terminal.

### Network & Connectivity

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-Network` | `gnet` | — | Ping-sweep a subnet and identify what each device is |
| `Get-Ports` | `port` | — | Check whether specific TCP ports are open on a host |
| `Get-IPInfo` | `gip` | — | Public IP, ISP, and location |
| `Clear-DNSCache` | `cdns` | **Yes** | Flush DNS, reset the TCP/IP stack, renew DHCP |
| `Invoke-WakeOnLan` | `wol` | — | Wake a sleeping machine by MAC address |
| `Connect-Wifi` | `cwf` | — | Join a Wi-Fi network, creating the profile if needed |
| `Get-WiFiPassword` | `gwf` | — | Show the saved password for a Wi-Fi network |
| `Get-WiFiSurvey` | `wifiscan` | — | Survey nearby Wi-Fi: signal, channel, band, security |
| `Invoke-SpeedTest` | `speed` | — | Internet speed test via Ookla Speedtest CLI |
| `Invoke-FallbackSpeedTest` | `speed2` | — | Download speed test using Cloudflare, no install needed |
| `Watch-LiveTraffic` | `traffic` | — | Live view of network connections by process |
| `Get-Download` | `dl` | — | Download a file with a progress bar |

### Network Diagnostics

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Invoke-NetworkDiag` | `netdiag` | — | Five-step connectivity check with a score out of 5 |
| `Invoke-SubnetScan` | `subscan` | — | Fast ping sweep to find live hosts |
| `Invoke-PortScan` | `pscan` | — | TCP port scan against one host |
| `Get-AdvancedDNS` | `advdns` | — | Full DNS record lookup against a chosen DNS server |
| `Get-ActiveConnections` | `netcon` | — | Current established TCP connections |
| `Get-NetworkInsight` | `netinsight` | — | Adapter details, gateway, DNS, and the ARP table |
| `Test-EndpointReachability` | `pingreach` | — | Ping a host and report average response time |
| `Reset-NetworkAdapter` | `netreset` | **Yes** | Disable and re-enable active network adapters |

### System Health & Performance

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-WiTechSystemInfo` | `gci2` | — | Hardware and OS summary |
| `Get-DiskSpace` | `gds` | — | Free, used, and total space per drive |
| `Get-DiskHealth` | `diskhealth` | Yes\* | S.M.A.R.T. health of physical disks |
| `Get-InstalledSoftware` | `software` | — | Installed programs from the registry, with CSV export |
| `Get-PendingWindowsUpdate` | `winupd` | — | Windows updates waiting to install |
| `Get-ProcessMemoryConsumer` | `memproc` | — | Top memory-hungry processes |
| `Get-ServiceHealth` | `gsvc` | Yes\* | Check critical Windows services, optionally restart them |
| `Get-PrinterStatus` | `printers` | Yes\* | List printers and queue depth, optionally clear stuck jobs |
| `Get-StaleProfiles` | — | — | Find user profiles nobody has touched in N days |
| `Clear-BrowserCache` | `clearcache` | Yes\* | Clear Chrome, Edge, and/or Firefox cache |
| `Clear-TempFiles` | `ctmp` | — | Delete temp files |
| `Invoke-DeepDiskCleanup` | `deepclean` | **Yes** | Aggressive cleanup of system and user caches |
| `Invoke-SystemRepair` | `repair` | **Yes** | Run SFC and DISM to repair Windows |
| `Optimize-CPU` | `ocpu` | **Yes** | Switch to High Performance and disable CPU throttling |
| `Optimize-HDD` | — | **Yes** | Defrag, DISM, SFC, then schedule chkdsk |
| `Reset-PrintSpooler` | `rps` | **Yes** | Stop the spooler, clear the queue, restart it |
| `Invoke-GPUpdate` | `gpo` | Yes\* | Force a Group Policy refresh, locally or remotely |
| `Repair-IdentityTrust` | — | Yes\* | Check and repair the domain trust relationship |

### Security & Threat Hunting

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-MalwareInfo` | `gmi` | — | Check file hashes against VirusTotal |
| `Get-FailedLogins` | `gfl` | **Yes** | Failed logon attempts, grouped by user and source |
| `Get-USBHistory` | `usblog` | **Yes** | Every USB device ever plugged into this machine |
| `Get-LocalAdminAudit` | `ladmin` | — | Who is a local administrator, and who should not be |
| `Invoke-ProcessGuard` | `pwatch` | Yes\* | Watch for CPU/RAM hogs, optionally kill them |
| `Invoke-RansomwareHeuristics` | — | Yes\* | Look for ransomware indicators |
| `Get-LateralMovementHeuristics` | — | **Yes** | Security log patterns suggesting lateral movement |
| `Get-DefenderStatus` | `defender` | — | Microsoft Defender health and recent detections |
| `Get-PersistenceAudit` | `persist` | Yes\* | Autoruns/tasks/services — flags unsigned entries |
| `Get-FirewallAudit` | `fwaudit` | — | Firewall profile state and risky inbound rules |

### Reporting & Documentation

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-TicketContext` | `ticket` | — | One-shot summary for a support call |
| `Get-Events` | `ge` | — | Recent WiTechTools activity from the event log |
| `Get-AssetInventory` | `asset` | — | Full hardware, OS, software and network inventory to CSV |
| `New-IncidentReport` | `incident` | — | Timestamped HTML incident report |
| `Export-SystemSnapshot` | `snap` | — | Save system state as JSON, or compare against an earlier one |
| `Invoke-SystemStateDiff` | — | — | Baseline processes/services/ports and report what changed |
| `Export-EventLogs` | `elogs` | **Yes** | Export Windows event logs to a dated ZIP |
| `Invoke-PatchReport` | `patch` | — | Recent Windows updates and whether patching is overdue |

### Active Directory

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-ADUserInfo` | `aduser` | — | Account status, group membership, password expiry |
| `Unlock-ADUser` | `unlock` | — | Unlock a locked-out account |

### Files, Media & Module Admin

| Command | Shortcut | Admin | What it does |
|---|---|:---:|---|
| `Get-Keys` | `gk` `getkeys` | — | Pull product keys from a CSV key store |
| `Set-Directory` | `sd` `setdir` | — | Create a working folder and move into it |
| `Set-MapDrive` | — | — | Map a network share to a drive letter |
| `Set-ScanDocs` | — | Yes\* | Set up a scanner share with a limited local account |
| `Set-ImageToPng` | `cpng` | — | Convert images and video frames to PNG |
| `Set-ImageToJpg` | `cjpg` | — | Convert images and video frames to JPG |
| `Sync-Module` | `sync` | — | Mirror this module to the team network share |
| `Import-WitechModule` | — | — | Reload the module from disk |
| `Update-WiTechModuleSignature` | — | Yes\* | Code-sign the module (see [Known Issues](#known-issues)) |
| `Get-WiTechCommands` | `cmds` | — | List every command and shortcut |

---

## Common Scenarios

Real tickets, and the commands that answer them fastest. Run them in order — each one narrows the problem down.

### "The internet is slow"

```powershell
speed                  # 1. Is it actually slow? Get a number.
netdiag                # 2. Where does it break? Gateway, DNS, or beyond.
traffic -GroupBy       # 3. Which program is eating the bandwidth?
gnet                   # 4. Is something unexpected on the network?
```

If `netdiag` scores 5/5 but `speed` is poor, the fault is upstream — the ISP or the office link, not the PC.

### "I can't get on the internet at all"

```powershell
netdiag                # Pinpoints the first broken step
gip                    # If this works, you have full internet access
cdns                   # Fixes most DNS and stale-lease problems (needs admin)
netreset               # Last resort: bounce the adapters (needs admin)
```

### "I think this machine has a virus"

```powershell
pwatch                 # Anything eating CPU or RAM abnormally?
gfl                    # Failed logon bursts = someone guessing passwords
usblog -Days 30        # Was something plugged in recently?
Invoke-RansomwareHeuristics   # Encryption indicators
ladmin                 # Did someone add themselves as an admin?
```

> `Invoke-RansomwareHeuristics` reports what it sampled. A clean result means "nothing found in the files I checked" — it is not a guarantee the machine is clean.

### "The printer isn't working"

```powershell
printers               # Is it there, and how deep is the queue?
rps                    # Stop, clear the queue, restart the spooler (needs admin)
printers -ClearStuck   # Remove jobs that refuse to die
```

### "This computer is really slow"

```powershell
memproc                # What is using the RAM?
gds                    # Is the disk nearly full? Under ~10% free causes this.
diskhealth             # Is the drive failing?
gsvc                   # Are critical services stopped?
ctmp                   # Free up space
```

### "I'm locked out" / account problems

```powershell
aduser jdoe            # Locked? Password expired? Which groups?
unlock jdoe            # Unlock and reset the bad-password count
gfl                    # What caused the lockout in the first place?
```

A lockout usually means a stale password saved somewhere — a phone, a mapped drive, or a scheduled task.

### Setting up or handing over a machine

```powershell
asset                  # Record what the machine is, to CSV
snap                   # Save a known-good baseline
gsvc                   # Confirm critical services are healthy
patch                  # Confirm it is up to date
```

Later, if the machine misbehaves, `Invoke-SystemStateDiff -Compare` shows exactly what changed since that baseline.

### Writing up a ticket

```powershell
ticket                 # Summary to paste into the ticket
incident -Title "Machine locks up daily"   # Formal HTML report
elogs -Days 3          # Attach the raw logs (needs admin)
```

---

## Command Reference

### Network & Connectivity

---

#### `Get-Network` · `gnet`

Ping-sweeps a subnet, then works out **what each device actually is** — a printer, a router, a Windows PC, an IP camera. It combines the MAC address vendor, the hostname, the TTL, and (optionally) open ports to make the call.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Target` | string | your local /24 | An IP (`192.168.1.50`), or a range (`10.0.0.0/24`). Omit to scan your own network. |
| `-Timeout` | int | `500` | Milliseconds to wait for each ping reply. Lower is faster but misses slow devices. |
| `-ThrottleLimit` | int | `50` | How many hosts to ping at once. |
| `-PortScan` | switch | off | Also scan ports. Slower, but makes device identification much more accurate. |
| `-Ports` | int[] | 21, 22, 23, 25, 53, 80, 110, 139, 143, 443, 445, 515, 554, 631, 3389, 8080, 8443, 9100 | Which ports to scan when `-PortScan` is used. |

```powershell
gnet                                    # scan your own network
Get-Network -Target "10.0.0.0/24" -Timeout 200
Get-Network -PortScan                   # slower, far better device identification
```

**Reading the Device Type column.** The colour tells you how much to trust it: **green** means proven (it is the gateway, it is this PC, or a port confirmed it), **yellow** means inferred from the hardware vendor, **grey** means a guess from TTL alone. Vendor names are looked up online once, then cached, so the first scan on a new network is slower.

---

#### `Get-Ports` · `port`

Checks whether specific TCP ports are open on a host. Use it to confirm a service is actually listening.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Hostname` | string | `google.com` | Host or IP to test. |
| `-Ports` | int[] | 80, 443, 22, 21, 3389, 8080 | Ports to test. |

```powershell
Get-Ports -Hostname "192.168.1.1"
Get-Ports -Hostname "localhost" -Ports @(80, 443, 1433, 3306)
Get-Ports -Hostname "myserver" -Ports 22
```

---

#### `Get-IPInfo` · `gip`

Shows your **public** IP address, ISP, and rough location, from ipinfo.io. Requires internet access — if it fails, you do not have working internet.

*No parameters.*

```powershell
gip
```

---

#### `Clear-DNSCache` · `cdns` · **Admin required (enforced)**

The "turn networking off and on again" command. Flushes DNS, releases and renews DHCP, resets the TCP/IP and Winsock stacks, and clears the ARP cache. Fixes a large share of "this one site won't load" and "I got a bad IP" problems.

*No parameters.*

```powershell
cdns
```

> Some resets do not take full effect until the machine restarts.

---

#### `Invoke-WakeOnLan` · `wol`

Sends a Magic Packet to wake a sleeping machine. The target must have Wake-on-LAN enabled in its BIOS and network adapter settings, and you must be on the same network segment.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-MacAddress` | string | *required* | Target MAC. Accepts `AA:BB:CC:DD:EE:FF`, `AA-BB-...`, or `AABBCCDDEEFF`. |
| `-BroadcastIP` | string | `255.255.255.255` | Set this to your subnet broadcast (e.g. `192.168.1.255`) if the default does not work. |
| `-Port` | int | `9` | UDP port. Some hardware uses `7`. |
| `-VerifyIP` | string | none | After sending, ping this IP to confirm the machine woke. |
| `-VerifyDelay` | int | `30` | Seconds to wait before verifying. |

```powershell
Invoke-WakeOnLan -MacAddress "AA:BB:CC:DD:EE:FF"
wol "AABBCCDDEEFF" -VerifyIP "192.168.1.50"
wol "AA-BB-CC-DD-EE-FF" -BroadcastIP "192.168.1.255" -Port 7
```

---

#### `Connect-Wifi` · `cwf`

Connects to a Wi-Fi network. If you supply a password it builds a temporary WPA2 profile, connects, then removes the temporary profile file.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SSID` | string | *required* | Network name. |
| `-Password` | SecureString | none | Omit if the machine already has a saved profile. |
| `-InterfaceName` | string | auto-detect | Only needed if the machine has more than one wireless adapter. |

```powershell
Connect-Wifi -SSID "WiTech-Staff" -Password (Read-Host -AsSecureString "Enter Password")
cwf -SSID "WiTech-Guest"
Connect-Wifi -SSID "SecureNet" -Password $mySecurePass -InterfaceName "Wi-Fi 2"
```

> `-Password` is a **SecureString**, so you cannot paste a plain password directly. Use `(Read-Host -AsSecureString)` as shown — it keeps the password off the screen and out of your command history.

---

#### `Get-WiFiPassword` · `gwf`

Shows the saved password for a Wi-Fi network in plain text. Handy when you need to connect a second device and nobody remembers the key.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SSID` | string | current network | Which saved network to reveal. |

```powershell
gwf                                # the network you are on right now
Get-WiFiPassword -SSID "WiTech-Staff"
```

> Only works for profiles saved on this machine, and only shows what the machine already knows.

---

#### `Get-WiFiSurvey` · `wifiscan`

Surveys the Wi-Fi around you: signal strength, channel, band, and security type. Use it to find channel congestion or a weak-signal dead spot.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ShowBSSID` | switch | off | Show each access point's radio MAC — useful for spotting multiple APs on one SSID. |
| `-Live` | switch | off | Keep refreshing. Walk around to find dead spots. Press `Ctrl+C` to stop. |
| `-RefreshSecs` | int | `1` | Refresh interval when `-Live` is used. |

```powershell
wifiscan
wifiscan -Live -RefreshSecs 2
Get-WiFiSurvey -ShowBSSID
```

---

#### `Invoke-SpeedTest` · `speed`

Runs a proper upload and download test using Ookla's Speedtest CLI, downloading the tool automatically on first use. If Ookla is blocked, it falls back to the Cloudflare test.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-OutputFormat` | `Human` `JSON` `CSV` `TSV` | `Human` | `Human` for reading; `JSON` returns an object you can script against. |
| `-ServerID` | string | auto | Pin the test to a specific Speedtest server. |
| `-SaveLog` | switch | off | Append the result to a CSV log. |
| `-LogPath` | string | `%LOCALAPPDATA%\WiTech\SpeedTest_Results.csv` | Where that log goes. |
| `-ShowNetworkInfo` | switch | off | Also print the local network configuration. |
| `-ForceRedownload` | switch | off | Re-download the Speedtest CLI. |

```powershell
speed
Invoke-SpeedTest -ShowNetworkInfo
speed -SaveLog -LogPath "C:\WiTechLogs\speed.csv"
Invoke-SpeedTest -OutputFormat JSON
```

> Logging with `-SaveLog` over several days is the fastest way to prove an intermittent slowdown to an ISP.

---

#### `Invoke-FallbackSpeedTest` · `speed2`

Download-speed test straight against Cloudflare. No external tool to install, nothing to download first — use it when `speed` is blocked or you want a quick number.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SaveLog` | switch | off | Append the result to a CSV log. |
| `-LogPath` | string | `%LOCALAPPDATA%\WiTech\SpeedTest_Results.csv` | Where that log goes. |

```powershell
speed2
speed2 -SaveLog -LogPath "C:\Logs\speed_fallback.csv"
```

> Download only — it does not measure upload.

---

#### `Watch-LiveTraffic` · `traffic`

A live, refreshing table of network connections with the process behind each one. This is how you catch the program quietly saturating the connection.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Top` | int | `28` | Rows to show. |
| `-RefreshSecs` | int | `3` | Seconds between refreshes. |
| `-Filter` | `All` `Established` `Listen` `TimeWait` `CloseWait` | `All` | `Established` = active conversations; `Listen` = services waiting for connections. |
| `-GroupBy` | switch | off | Group by process instead of listing every connection. Best starting view. |

```powershell
traffic -GroupBy
Watch-LiveTraffic -Top 20
traffic -Filter Established -RefreshSecs 1
```

> Runs until you press `Ctrl+C`.

---

#### `Get-Download` · `dl`

Downloads a file with a live progress bar and speed readout.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Url` | string | *required* | HTTP/HTTPS address. |
| `-Destination` | string | `%USERPROFILE%\Downloads` | Folder to save into. |
| `-FileName` | string | from the URL | Save under a different name. |
| `-Force` | switch | off | Overwrite an existing file. |

```powershell
Get-Download -Url "https://example.com/file.zip"
dl -Url "https://example.com/installer.msi" -Destination "C:\Tools" -Force
Get-Download -Url "https://example.com/image.png" -FileName "logo.png"
```

---

### Network Diagnostics

---

#### `Invoke-NetworkDiag` · `netdiag`

The single best first command for any connectivity complaint. Runs five checks in order — adapter, gateway, DNS, internet, custom target — and scores the result out of 5. **The first step that fails is your problem.**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Target` | string | `8.8.8.8` | An extra host to test, e.g. the server the user cannot reach. |
| `-Traceroute` | switch | off | Also trace the route to `-Target`. |

```powershell
netdiag
netdiag -Target "mycorp.com" -Traceroute
Invoke-NetworkDiag -Target "192.168.1.1"
```

**Reading the score:** gateway fails → local cable, Wi-Fi, or switch. DNS fails but gateway is fine → DNS settings, try `cdns`. Internet fails but DNS resolves → upstream or firewall.

---

#### `Invoke-SubnetScan` · `subscan`

A fast, no-frills ping sweep that lists which addresses are alive. Use `gnet` instead if you want to know *what* each device is.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Subnet` | string | `192.168.1.0/24` | Subnet in CIDR form. |
| `-Timeout` | int | `500` | Milliseconds per host. |

```powershell
subscan
Invoke-SubnetScan -Subnet "10.0.0.0/24"
Invoke-SubnetScan -Subnet "192.168.10.0/24" -Timeout 200
```

Returns the list of live IPs, so you can pass it to other commands.

---

#### `Invoke-PortScan` · `pscan`

Scans TCP ports on one host and reports each as open or closed/filtered.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | *required* | Host or IP to scan. |
| `-Ports` | int[] | 21, 22, 23, 53, 80, 443, 445, 3389 | Ports to test. |

```powershell
pscan -ComputerName "localhost"
Invoke-PortScan -ComputerName "192.168.1.50" -Ports @(21, 22, 23, 80)
Invoke-PortScan -ComputerName "my-firewall" -Ports 443
```

> If the hostname cannot be resolved, the command says so once and stops rather than reporting every port as closed.
>
> **Only scan networks you are responsible for.** Port scanning other people's networks may be illegal and will likely trigger their security alerts.

---

#### `Get-AdvancedDNS` · `advdns`

Looks up **all** DNS record types for a domain against a DNS server you choose. Query a public server and your internal one, then compare — differences explain a lot of "it works for me but not for them" problems.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Domain` | string | *required* | Domain to look up. |
| `-Server` | string | `8.8.8.8` | DNS server to ask. `1.1.1.1` is Cloudflare. |

```powershell
Get-AdvancedDNS -Domain "witech.org"
advdns -Domain "google.com" -Server "1.1.1.1"
```

---

#### `Get-ActiveConnections` · `netcon`

Lists currently established TCP connections with the process ID behind each. A quick, static snapshot — use `traffic` for a live view.

*No parameters.* Shows the first 15 connections.

```powershell
netcon
```

---

#### `Get-NetworkInsight` · `netinsight`

A deep look at local networking: each adapter, its IP, gateway, DNS servers, link speed, plus the ARP table of devices recently talked to.

*No parameters.*

```powershell
netinsight
```

Also works on Linux and macOS, where it falls back to `ip`/`ifconfig` and `arp`.

---

#### `Test-EndpointReachability` · `pingreach`

Pings a host a set number of times and reports the average response time. Use it to demonstrate packet loss or high latency.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Endpoint` | string | *required* | Host or IP. |
| `-Count` | int | `4` | Number of pings. |

```powershell
Test-EndpointReachability -Endpoint "google.com"
pingreach -Endpoint "192.168.1.1" -Count 10
```

---

#### `Reset-NetworkAdapter` · `netreset` · **Admin required (enforced)**

Disables and re-enables every active network adapter. A software-level "unplug the cable and plug it back in".

*No parameters.*

```powershell
netreset
```

> **You will lose network connectivity for a few seconds.** Never run this over Remote Desktop or a remote session — you will disconnect yourself and may not get back in.

---

### System Health & Performance

---

#### `Get-WiTechSystemInfo` · `gci2`

Hardware and OS summary: operating system, CPU cores, total memory, and drive space. The first thing to run when you need to know what you are dealing with.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | `localhost` | Query another machine (needs WinRM). |

```powershell
gci2
gci2 -ComputerName "finance-pc"
```

---

#### `Get-DiskSpace` · `gds`

Free, used, and total space for every drive, with a percentage bar. Below roughly 10% free, Windows slows noticeably — check this early on any "slow computer" ticket.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | `$env:COMPUTERNAME` | Query another machine (needs WinRM). |

```powershell
gds
gds -ComputerName "fileserver"
```

---

#### `Get-DiskHealth` · `diskhealth` · *Admin recommended*

Reads the S.M.A.R.T. health status the drive reports about itself. Anything other than *Healthy* means back the data up now.

*No parameters.*

```powershell
diskhealth
```

> Without elevation some drives report incomplete data. A *Healthy* result is not a promise — drives do fail without warning.

---

#### `Get-InstalledSoftware` · `software`

Lists installed programs, read from the registry. Supports filtering and CSV export.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Name` | string | all | Filter by name; wildcards allowed (`*chrome*`). |
| `-Publisher` | string | all | Filter by publisher (`Adobe*`). |
| `-IncludeUpdates` | switch | off | Include Windows updates and hotfix entries. |
| `-ExportPath` | string | none | Write results to a CSV. |

```powershell
software
software -Name "*chrome*"
Get-InstalledSoftware -Publisher "Adobe*" -ExportPath "C:\Reports\adobe.csv"
```

> Reads the registry only. It deliberately avoids the `Win32_Product` method, which is slow and can trigger repair operations on every installed MSI.

---

#### `Get-PendingWindowsUpdate` · `winupd`

Asks the Windows Update service what is waiting to install.

*No parameters.*

```powershell
winupd
```

---

#### `Get-ProcessMemoryConsumer` · `memproc`

Top memory-consuming processes, largest first.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Top` | int | `10` | How many to list. |

```powershell
memproc
memproc -Top 20
```

---

#### `Get-ServiceHealth` · `gsvc` · *Admin needed for `-AutoFix`*

Checks the Windows services that matter — Defender, Firewall, Event Log, DNS Client, Spooler, Workstation, Update, DHCP, Time, Search, Netlogon, BITS — and reports any that are not running.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Services` | string[] | 14 critical services | Check a specific list instead. |
| `-AutoFix` | switch | off | Attempt to start anything stopped. **Needs admin.** |

```powershell
gsvc
gsvc -AutoFix
Get-ServiceHealth -Services @("WinDefend", "MpsSvc")
```

---

#### `Get-PrinterStatus` · `printers` · *Admin needed for `-ClearStuck`*

Lists printers, their status, and how many jobs are queued.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ClearStuck` | switch | off | Delete stuck jobs. **Needs admin.** |

```powershell
printers
printers -ClearStuck
```

> If `-ClearStuck` does not fix it, use `rps` to reset the whole spooler.

---

#### `Get-StaleProfiles`

Finds local user profiles nobody has used recently — useful for reclaiming disk space on shared machines.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-DaysInactive` | int | `30` | Inactivity threshold in days. |

```powershell
Get-StaleProfiles
Get-StaleProfiles -DaysInactive 90
```

> Reports only. It does not delete anything.

---

#### `Clear-BrowserCache` · `clearcache` · *Admin needed for other users*

Clears cache, cookies, and history for Chrome, Edge, and/or Firefox.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Browser` | `All` `Chrome` `Edge` `Firefox` | `All` | Which browser to clear. |
| `-Username` | string | current user | Clear another user's profile. **Needs admin.** |

```powershell
clearcache
clearcache -Browser "Chrome"
clearcache -Browser "Chrome" -Username "Administrator"
```

> **This signs the user out of websites** and clears saved history. Warn them first. Close the browser before running, or locked files will be skipped.

---

#### `Clear-TempFiles` · `ctmp`

Deletes temporary files from the user's temp folders.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-IncludeWindowsTemp` | switch | off | Also clear the Windows system temp folder. |

```powershell
ctmp
ctmp -IncludeWindowsTemp
ctmp -WhatIf                 # preview which folders would be cleared, delete nothing
```

> Files in use are skipped — that is normal and not an error. Supports `-WhatIf`/`-Confirm`.

---

#### `Invoke-DeepDiskCleanup` · `deepclean` · **Admin required (enforced)**

A far more aggressive cleanup than `ctmp`: system temp, every user's temp folder, prefetch, and Windows distribution caches.

*No parameters.*

```powershell
deepclean
deepclean -WhatIf           # preview every path that would be wiped, delete nothing
```

> Clearing prefetch will make the next few application launches slightly slower while Windows rebuilds it. Use `ctmp` first; keep this for when you genuinely need the space. Supports `-WhatIf`/`-Confirm`.

---

#### `Invoke-SystemRepair` · `repair` · **Admin required (enforced)**

Runs the two standard Windows repair tools in the correct order: SFC to check system files, then DISM to repair the component store.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SkipSFC` | switch | off | Skip the SFC scan. |
| `-SkipDISM` | switch | off | Skip the DISM repair. |

```powershell
repair
repair -SkipSFC
Invoke-SystemRepair -SkipDISM
```

> Takes 15–45 minutes. Do not interrupt it. DISM needs internet access to fetch replacement files.

---

#### `Optimize-CPU` · `ocpu` · **Admin required (enforced)**

Switches Windows to the High Performance power plan and disables CPU throttling and core parking.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Action` | `Optimize` `Revert` | `Optimize` | `Revert` restores Balanced and re-enables throttling. |
| `-VerboseOutput` | switch | off | Show detailed progress. |
| `-Force` | switch | off | Proceed even if the elevation check fails. |

```powershell
ocpu
ocpu -Action Revert
```

> **On laptops this significantly reduces battery life and increases heat.** Use `-Action Revert` when finished.

---

#### `Optimize-HDD` · **Admin required (enforced)**

A full maintenance pass: defragment C:, DISM component check, SFC scan, then schedule chkdsk for the next boot.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-VerboseOutput` | switch | off | Show detailed progress. |
| `-NoRestart` | switch | off | Schedule chkdsk but do not reboot. |

```powershell
Optimize-HDD -NoRestart
```

> **Reboots the machine unless you pass `-NoRestart`.** Always use `-NoRestart` on someone else's machine. The scheduled chkdsk runs before Windows loads and can take hours on a large or failing disk.

---

#### `Reset-PrintSpooler` · `rps` · **Admin required (enforced)**

Stops the Print Spooler, deletes everything stuck in the queue, and starts it again. Fixes most "the printer won't print and won't clear" problems.

*No parameters.*

```powershell
rps
```

> **All queued print jobs are lost, for every user on the machine.** Anything half-printed must be sent again.

---

#### `Invoke-GPUpdate` · `gpo` · *Admin needed for computer policy*

Forces a Group Policy refresh and then reports the applied policy summary.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | this machine | Refresh a remote machine (needs WinRM). |
| `-Target` | `Both` `Computer` `User` | `Both` | Which policy scope to refresh. |

```powershell
gpo
gpo -Target Computer
Invoke-GPUpdate -ComputerName "FRONTDESK-02"
```

> Some policies only apply at logon or startup, so a refresh alone may not be enough.

---

#### `Repair-IdentityTrust` · *Admin needed*

Checks the secure channel between the machine and the domain — the trust relationship that, when broken, produces *"The trust relationship between this workstation and the primary domain failed."* Also reports Entra ID join state.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Force` | switch | off | Attempt the repair. Prompts for domain credentials. |

```powershell
Repair-IdentityTrust             # check only, changes nothing
Repair-IdentityTrust -Force      # attempt repair
```

> Without `-Force` it only reports. Repairing requires domain credentials with permission to reset the computer account.

---

### Security & Threat Hunting

---

#### `Get-MalwareInfo` · `gmi`

Takes SHA256 file hashes from a CSV, checks each against VirusTotal, and writes a report of anything flagged malicious.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-TempPath` | string | `%TEMP%` | Where the report is written. |
| `-AlertPath` | string | `C:\witech\AlertDetail.csv` | CSV containing the hashes to check. |
| `-ApiKey` | string | `$env:VIRUSTOTAL_API_KEY` | Your VirusTotal API key. |

```powershell
Get-MalwareInfo -ApiKey "MySecretVTKey"
gmi -AlertPath "C:\WiTech\ThreatAlerts.csv"
```

> Needs internet and a VirusTotal API key. Free keys are rate-limited to roughly 4 lookups per minute, so large lists take a while. Only hashes are sent — never file contents.

---

#### `Get-FailedLogins` · `gfl` · **Admin required (enforced)**

Reads failed logon events (ID 4625) from the Security log, then groups them by targeted username and source. A burst against one account is a password-guessing attempt; a burst across many accounts is a spray attack.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-MaxEvents` | int | `500` | How many events to read back through. |
| `-TopN` | int | `15` | How many targeted accounts to list. |
| `-AlertThreshold` | int | `5` | Failures against one account before it is highlighted. |

```powershell
gfl
gfl -MaxEvents 1000 -AlertThreshold 10
Get-FailedLogins -TopN 5
```

> Most failures are mundane — a stale saved password on a phone or a mapped drive. Look for volume and pattern, not single events.

---

#### `Get-USBHistory` · `usblog` · **Admin required (enforced)**

Lists every USB device ever connected to the machine, from the registry, with last-connected timestamps.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Days` | int | `0` (all) | Only devices seen in the last N days. |
| `-StorageOnly` | switch | off | Only mass-storage devices — the ones that can carry data off-site. |

```powershell
usblog
usblog -Days 14 -StorageOnly
```

---

#### `Get-LocalAdminAudit` · `ladmin`

Lists the local Administrators group and flags entries worth reviewing: enabled local accounts other than the built-in Administrator, domain users granted admin directly rather than through a group, and orphaned SIDs from deleted accounts.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ExpectedMembers` | string[] | none | Your approved list. Anything not on it is flagged. |

```powershell
ladmin
ladmin -ExpectedMembers "Administrator", "CONTOSO\Domain Admins", "CONTOSO\IT-Helpdesk"
ladmin | Where-Object Flag | Export-Csv C:\Reports\admin-flags.csv -NoTypeInformation
```

> Finds the group by its well-known SID, so it works regardless of the system's display language.

---

#### `Invoke-ProcessGuard` · `pwatch` · *Admin needed for `-AutoKill`*

Live watchdog for processes exceeding CPU or memory thresholds.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-CpuThreshold` | int | `80` | CPU percentage that triggers an alert. |
| `-RamThresholdMB` | int | `1500` | Working-set megabytes that trigger an alert. |
| `-Top` | int | `20` | Rows to display. |
| `-SampleSecs` | int | `3` | Seconds between samples. |
| `-AutoKill` | switch | off | **Terminate** offending processes automatically. |

```powershell
pwatch
Invoke-ProcessGuard -CpuThreshold 90 -RamThresholdMB 2000
Invoke-ProcessGuard -SampleSecs 1 -Top 10
```

> **`-AutoKill` terminates processes without asking, and unsaved work is lost.** Watch first without it and confirm the process is genuinely misbehaving. Runs until `Ctrl+C`.

---

#### `Invoke-RansomwareHeuristics` · *Admin needed for shadow-copy check*

Looks for signs of active ransomware: deleted Volume Shadow Copies, known ransomware file extensions and ransom notes, files with abnormally high entropy (a sign of mass encryption), and directories with sudden bursts of modifications.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Path` | string[] | user directories | Specific paths to sweep, e.g. a file share. |
| `-SampleSize` | int | `400` | How many files to sample for entropy. |
| `-RecentMinutes` | int | `15` | Window for detecting modification bursts. |

```powershell
Invoke-RansomwareHeuristics
Invoke-RansomwareHeuristics -Path "D:\Shares\Finance" -SampleSize 1000
Invoke-RansomwareHeuristics -RecentMinutes 60 -Verbose
```

> **This is a heuristic, not a scanner.** The output states its scope: a clean result means "nothing found in the files I sampled", not "this machine is clean". If you suspect an active infection, disconnect the machine from the network first.

---

#### `Get-LateralMovementHeuristics` · *Admin needed*

Scans the Security log for patterns that suggest an attacker moving between machines — unusual logon types, privilege escalation, and remote logon sequences.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-MaxEvents` | int | `1000` | How many Security events to analyse. |

```powershell
Get-LateralMovementHeuristics
Get-LateralMovementHeuristics -MaxEvents 5000
```

> Produces leads for investigation, not verdicts. Administrative work legitimately generates many of the same patterns.

---

#### `Get-DefenderStatus` · `defender`

Microsoft Defender health at a glance: real-time protection, signature age, last scan, tamper protection, and threats detected recently. Says so clearly if Defender is not the active antivirus. Returns a summary object.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ThreatHistoryDays` | int | `7` | How many days back to count recent detections. |

```powershell
defender
Get-DefenderStatus -ThreatHistoryDays 30
Get-DefenderStatus | Select-Object RealTimeProtection, SignatureAgeDays
```

> Read-only. If a third-party antivirus has taken over, the Defender cmdlets may be unavailable — the command tells you rather than failing.

---

#### `Get-PersistenceAudit` · `persist` · *Admin needed for full visibility*

Sweeps the places malware installs itself to survive a reboot — Run/RunOnce keys, Startup folders, auto-start services, and non-Microsoft scheduled tasks — and flags any entry whose executable is unsigned, tampered, or missing on disk. Returns every entry as an object.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Top` | int | `40` | Maximum flagged entries to print. All entries are still returned as objects. |

```powershell
persist
persist -Top 100
Get-PersistenceAudit | Where-Object Flag | Export-Csv C:\Reports\autoruns.csv -NoTypeInformation
```

> A heuristic sweep, not a verdict. A signed third-party autostart is flagged for awareness, not because it is malicious.

---

#### `Get-FirewallAudit` · `fwaudit`

Reports whether each firewall profile (Domain/Private/Public) is enabled and its default actions, then flags enabled inbound Allow rules that expose sensitive services — RDP, SMB, WinRM, SSH, SQL, VNC — to any remote address. Returns the risky rules as objects.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-IncludeAllowRules` | switch | off | Also list every enabled inbound Allow rule, not just risky ones. |

```powershell
fwaudit
Get-FirewallAudit -IncludeAllowRules
Get-FirewallAudit | Export-Csv C:\Reports\firewall-risky.csv -NoTypeInformation
```

> Read-only. The scan reads every enabled inbound Allow rule, so it can take a few seconds on a machine with many rules.

---

### Reporting & Documentation

---

#### `Get-TicketContext` · `ticket`

The one command to run at the start of any support call. Collects last boot time, disk space, CPU, recent errors, and patch status — everything you would otherwise ask the user for.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | `$env:COMPUTERNAME` | Query another machine (needs WinRM). |

```powershell
ticket
ticket -ComputerName "accounting-pc"
```

---

#### `Get-Events` · `ge`

Shows recent WiTechTools activity from the Windows Application event log — a record of which toolkit commands ran and what they reported.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-MaxEvents` | int | `30` | How many entries to show. |

```powershell
ge
ge -MaxEvents 50
```

---

#### `Get-AssetInventory` · `asset`

Full inventory of a machine — OS, CPU, RAM, BIOS, motherboard, disks, network adapters, and installed software — displayed as a summary and exported to CSV for asset tracking.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ComputerName` | string | `$env:COMPUTERNAME` | Inventory a remote machine (needs WinRM). |
| `-ExportPath` | string | Documents | Where to write the CSV. |
| `-SoftwareTop` | int | `30` | How many installed programs to include. |

```powershell
asset
asset -ComputerName "FRONTDESK-02" -SoftwareTop 50
Get-AssetInventory -ExportPath "\\fileserver\IT\Inventory\office-pc.csv"
```

---

#### `New-IncidentReport` · `incident`

Builds a timestamped HTML report with system context and recent toolkit activity — suitable for attaching to a ticket or sending to management.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Title` | string | `IT Incident` | Report title. |
| `-ReportedBy` | string | `$env:USERNAME` | Your name. |
| `-OutputPath` | string | Desktop | Where to save the HTML file. |
| `-EventCount` | int | `25` | How many recent toolkit events to embed. |

```powershell
New-IncidentReport -Title "System Lockup"
incident -Title "CPU Spike" -ReportedBy "TechSupport" -OutputPath "C:\Reports"
```

---

#### `Export-SystemSnapshot` · `snap`

Saves the machine's current state — processes, services, network configuration — to a JSON file, or compares the machine against a snapshot taken earlier.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Path` | string | default location | Where to save the snapshot. |
| `-Compare` | string | none | Path to an earlier snapshot to compare against. |
| `-Open` | switch | off | Open the file when done. |

```powershell
snap
snap -Compare "C:\Snapshots\baseline.json"
Export-SystemSnapshot -Path "C:\WiTechLogs\snapshot.json" -Open
```

> Take a snapshot on every machine you set up. Comparing later turns "it used to work" into a specific list of what changed.

---

#### `Invoke-SystemStateDiff`

Same idea as `snap`, focused on processes, services, and listening ports, with a simpler two-step workflow.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-BaselinePath` | string | `%TEMP%\system_baseline.json` | Where the baseline lives. |
| `-Compare` | switch | off | Compare against the baseline instead of creating one. |

```powershell
Invoke-SystemStateDiff                       # capture a baseline
Invoke-SystemStateDiff -Compare              # what changed since?
Invoke-SystemStateDiff -BaselinePath "C:\WiTechLogs\baseline.json"
```

> The default baseline lives in TEMP and can be cleaned up by Windows or by `ctmp`. Use `-BaselinePath` for anything you need to keep.

---

#### `Export-EventLogs` · `elogs` · **Admin required (enforced)**

Exports the Application, System, and Security event logs to a dated ZIP — for escalation, or before rebuilding a machine.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-OutputPath` | string | Desktop | Where to write the ZIP. |
| `-Logs` | string[] | Application, System, Security | Which logs to include. |
| `-Days` | int | `7` | How far back to export. |

```powershell
elogs
elogs -Logs @("Application", "System") -Days 14 -OutputPath "C:\Temp"
Export-EventLogs -Days 1
```

> Event logs can contain usernames and machine names. Handle the ZIP accordingly.

---

#### `Invoke-PatchReport` · `patch`

Lists recently installed Windows updates and warns if the machine has not been patched in too long.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Last` | int | `20` | How many updates to list. |
| `-StaleDays` | int | `30` | Days without a patch before flagging as stale. |

```powershell
patch
patch -Last 10 -StaleDays 14
```

---

### Active Directory

Both commands need the **RSAT Active Directory** tools installed and a domain-joined machine.

---

#### `Get-ADUserInfo` · `aduser`

Shows an AD account's status: enabled or disabled, locked out, last logon, password expiry, and group memberships. The first stop for any account problem.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Username` | string | *required* | Username or UPN. Accepts pipeline input. |

```powershell
aduser jdoe
Get-ADUserInfo -Username "jdoe@domain.com"
"jdoe" | Get-ADUserInfo
```

---

#### `Unlock-ADUser` · `unlock`

Unlocks a locked-out account and resets its bad-password count.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Username` | string | *required* | Username to unlock. Accepts pipeline input. |

```powershell
unlock jdoe
"jdoe" | Unlock-ADUser
```

> Unlocking treats the symptom. If it locks again within minutes, something is retrying an old password — check phones, mapped drives, and scheduled tasks, and use `gfl` to find the source.

---

### Files, Media & Module Admin

---

#### `Get-Keys` · `gk` · `getkeys`

Reads product keys from one or more CSV files and appends them to a master key registry.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-path` | string[] | `C:\witech\keys.csv` | Source CSV, or several. |
| `-dest` | string | `C:\witech\keys-complete.csv` | Master registry to append to. |

```powershell
gk
gk -path "C:\Temp\keys.csv" -dest "D:\Backup\allkeys.csv"
gk -path @("C:\keys1.csv", "C:\keys2.csv")
```

> Appends rather than overwrites, so the master file accumulates. This is intentional.

---

#### `Set-Directory` · `sd` · `setdir`

Creates a working folder and moves into it. If the first name is taken, it uses the fallback.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Name` | string | `./temp/` | Preferred folder. |
| `-Name2` | string | `./temp-2/` | Fallback if the first exists. |

```powershell
sd
sd -Name "C:\WorkTemp"
Set-Directory -Name "./sandbox" -Name2 "./sandbox-fallback"
```

---

#### `Set-MapDrive`

Maps a network share to a drive letter.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-DriveLetter` | string | `Z:` | Letter to map to. |
| `-NetworkPath` | string | `\\server\share` | UNC path of the share. |

```powershell
Set-MapDrive -DriveLetter "S:" -NetworkPath "\\dc1\sales"
Set-MapDrive -NetworkPath "\\backup-nas\archives"
```

---

#### `Set-ScanDocs` · *Admin needed*

Sets up a scan-to-folder share for a copier or scanner. Creates the folders, provisions a **standard, non-administrative** local account for the device to authenticate as, and shares the folder with Change access and NTFS Modify rights only.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Username` | string | `scans` | Local account for the scanner. |
| `-ShareName` | string | `ScannedDocuments` | Name of the SMB share. |

```powershell
Set-ScanDocs
Set-ScanDocs -Username "copier" -Verbose
```

> Deliberately grants the least privilege that works — a scanner account should never be an administrator. The elevated helper script is staged in an access-restricted folder rather than the world-readable TEMP directory.

---

#### `Set-ImageToPng` · `cpng`

Converts images and video frames to PNG using FFmpeg.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SourceDirectory` | string | `.` (current folder) | Folder to read from. |
| `-OutputDirectory` | string | `./png` | Folder to write to. |

```powershell
cpng
Set-ImageToPng -SourceDirectory "C:\Pictures"
cpng -OutputDirectory "C:\Pngs"
```

> Requires FFmpeg on your `PATH`.

---

#### `Set-ImageToJpg` · `cjpg`

Converts images and video frames to JPG using FFmpeg.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-SourceDirectory` | string | `.` (current folder) | Folder to read from. |
| `-OutputDirectory` | string | `./jpg` | Folder to write to. |
| `-Quality` | int | `85` | 1–100. Higher is better quality and a larger file. |

```powershell
cjpg
Set-ImageToJpg -SourceDirectory "C:\Pictures" -Quality 90
Set-ImageToJpg -Quality 50
```

> Requires FFmpeg on your `PATH`.

---

#### `Sync-Module` · `sync`

Mirrors this module to the team network share so every technician gets the same version.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Destination` | string | `$env:WITECH_SYNC_PATH`, else `T:\Dev\Powershell\Modules\WiTechTools` | Target share. |

```powershell
sync
sync -Destination "\\fileserver\IT\Modules\WiTechTools"
```

> Copies only — it never deletes files on the share. Backups, test files, and `.git` internals are excluded. If the share is unreachable it gives up after about 4 seconds rather than hanging.

---

#### `Import-WitechModule`

Reloads the module from disk. Use it after editing the module, or if commands are behaving oddly.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-ModulePath` | string | the loaded module's own folder | Load from somewhere else. |
| `-Sign` | switch | off | Also code-sign the module after loading. |

```powershell
Import-WitechModule
Import-WitechModule -ModulePath "D:\Dev\WiTechTools"
Import-WitechModule -Sign
```

> `Import-Module WiTechTools -Force` does the same job in one line and is usually quicker to type.

---

#### `Update-WiTechModuleSignature` · *Admin needed*

Creates a self-signed code-signing certificate (CN=TechSupport), trusts it on this machine, and signs a file with it.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-FilePath` | string | see [Known Issues](#known-issues) | File to sign. **Supply this explicitly.** |
| `-CertPath` | string | `%USERPROFILE%\Documents\MyCodeSigningCert.cer` | Where to export the certificate. |

```powershell
Update-WiTechModuleSignature -FilePath "C:\scripts\myscript.ps1"
```

> **The default `-FilePath` is wrong on most machines** — always pass `-FilePath` explicitly. See [Known Issues](#known-issues). Trusting the certificate writes to the machine-wide trust store and needs admin. A self-signed certificate is only trusted on machines where it has been installed.

---

#### `Get-WiTechCommands` · `cmds`

Prints every WiTechTools command with its shortcut. When you cannot remember a command name, start here.

*No parameters.*

```powershell
cmds
```

---

## Logging

WiTechTools records activity in the **Windows Application event log** under the source `WiTech`. Each command writes its own event ID, so you can trace which toolkit commands ran on a machine and what they reported.

```powershell
ge                # recent toolkit activity
ge -MaxEvents 100 # look further back
```

The `WiTech` source is registered automatically the first time the module is imported from an elevated terminal. Until that happens, logging is silently skipped — commands still work normally.

---

## Troubleshooting

**"The term 'gnet' is not recognized"**
The module is not loaded in this terminal. Run `Import-Module WiTechTools`, and add it to your `$PROFILE` so it loads every time.

**"The module requires a minimum PowerShell version of 7.0"**
You are in Windows PowerShell 5.1. Close it and open **PowerShell 7** (`pwsh`).

**A command prints "Must be run as Administrator"**
That command enforces elevation. Close your terminal, right-click PowerShell 7, and choose *Run as administrator*.

**A command fails with confusing permission errors**
It probably needs admin but does not check for it — anything marked **Yes\*** in the tables above. Try again elevated.

**Output looks like `←[36m` instead of colours**
Your terminal does not support ANSI colour. Use Windows Terminal or the PowerShell 7 console rather than the legacy console host.

**`Get-Network` is slow on its first run**
It looks up hardware vendors online the first time it sees each manufacturer, then caches them. Later scans on the same network are much faster.

**Remote `-ComputerName` commands fail**
The target needs WinRM enabled and reachable, and you need rights on it. Test with `Test-WSMan -ComputerName <name>`.

**AD commands fail with "not recognized"**
The RSAT Active Directory tools are not installed on this machine.

---

## Known Issues

None currently open.

Two long-standing issues were resolved in 4.3.0:

- `Update-WiTechModuleSignature` had default paths that did not exist on a standard install. Both now resolve correctly, including when Documents is redirected to OneDrive.
- `Get-ComputerInfo` used to be a WiTechTools wrapper that replaced the built-in Windows cmdlet and did not support its parameters. It has been removed, so `Get-ComputerInfo` is once again Microsoft's cmdlet, with full support for `-Property` and the rest. Use `gci2` (`Get-WiTechSystemInfo`) for the WiTech hardware summary.

---

## Running the Tests

```powershell
Install-Module Pester -Scope CurrentUser -Force    # Pester 5.x
Invoke-Pester -Path .\tests -Output Detailed
```

The suite validates the manifest, imports the module in a clean session, checks that every declared command and shortcut actually exists (and that nothing exists undeclared), verifies help completeness, runs PSScriptAnalyzer, and unit-tests the logic behind state diffing, entropy scanning, key export, and snapshot comparison.

Run it before `sync`.

**GitHub Actions** runs the same suite on every push and pull request against `main`, plus a weekly scheduled run so a platform update can't quietly break the module between pushes. See `.github/workflows/test.yml`.

**A pre-push hook enforces this locally**, since GitHub's required-status-check enforcement (branch protection / rulesets) is gated behind a paid plan for a private repository. On a fresh clone, enable it once:

```powershell
git config core.hooksPath .githooks
```

After that, `git push` to `main` runs the suite first and refuses the push if anything fails. Deliberately bypass it with `git push --no-verify` if you genuinely need to push a known-broken branch.

---

## Project Layout

```
WiTechTools/
├── WiTechTools.psd1               # Manifest: version, exported commands, shortcuts
├── WiTechTools.psm1               # Loader — imports everything in Public\
├── PSScriptAnalyzerSettings.psd1  # Linting rules for this project
├── README.md                      # This guide
├── Public/                        # One file per area
│   ├── AD.ps1                     # Active Directory
│   ├── FileTools.ps1              # Files, media, module administration
│   ├── Helpers.ps1                # Shared internals: tables, logging, device typing
│   ├── Network.ps1                # Everyday networking
│   ├── NetworkDiagnostics.ps1     # Deeper network troubleshooting
│   ├── Reporting.ps1              # Reports, snapshots, inventory
│   ├── Security.ps1               # Threat hunting and auditing
│   └── System.ps1                 # Health, performance, repair
└── tests/                         # Pester 5 test suite
```

`Helpers.ps1` also holds internal functions that are deliberately **not** exported — table formatting, event logging, the device classifier used by `Get-Network`, and the shared cursor handling behind every self-refreshing display (`traffic`, `pwatch`, `wifiscan -Live`), which is what keeps each refresh redrawing in place instead of scrolling the previous frame up. They are available inside the module but do not appear in `cmds`.

---

*Built and maintained by the WiTech team.*
