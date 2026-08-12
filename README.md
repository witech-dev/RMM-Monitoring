# Alert Reference Guide — Disk Health & Print Spooler Monitoring

**Last updated:** August 12, 2026
**Companion scripts:** `AutoDiskRepair.ps1` (disk repair), `SpoolerRepair.ps1` (print spooler repair)

This guide explains every alert code our RMM monitoring can produce, what each one means in plain English, and what (if anything) you need to do about it. When an alert comes in, find its code in the **60-Second Triage Table** below, then jump to the detailed section if you need more background.

---

## 1. How the System Works (Plain English)

Windows keeps a running diary of everything that happens on a computer, called the **event log**. When something goes wrong — a hard drive stumbles, a service crashes — Windows writes a numbered note in that diary. Our RMM monitors read the diary constantly and raise an alert when certain bad numbers appear.

Here's the part that makes our setup different from plain alerting: **the alerts trigger robots that try to fix the problem first.** The flow looks like this:

```
Windows logs a problem code
        │
        ▼
RMM monitor sees it → raises an alert → runs the repair script
        │
        ▼
The repair script does its work, then writes ITS OWN code back
into the event log saying how it went:
        │
        ├── "All fixed" code  → the alert closes itself. Nobody is bothered.
        ├── "In progress" code → alert stays open until the fix completes.
        └── "I give up" code  → a SECOND monitor sees this and raises a
                                HIGH-priority alert: a human is needed.
```

So there are **two families of codes** you'll see in alerts:

1. **Windows' own warning codes** (7, 11, 51, 52, 55, 153, etc.) — Windows saying "something looks wrong." These start the process.
2. **Our robots' report codes** (8000-series for printing, 9000-series for disks) — our scripts saying "here's what I found and what I did." These end the process, one way or the other.

**The rule of thumb:** if the only codes you see are green ones from the tables below, the system handled it. The codes that page you at High priority (9002, 9003, 8002) exist precisely because the robot already tried everything safe and failed — don't re-run the automation on those; a person needs to look.

---

## 2. 60-Second Triage Table

Every code in the system, one line each. 🟢 = no action needed, 🟡 = keep an eye on it / minor action, 🔴 = act now.

| Code | Who logs it | One-line meaning | Severity | What you do |
|---|---|---|---|---|
| **7** | Windows (`disk`) | Drive found a physically damaged spot (bad block) | 🟡 | Auto-repair runs. Watch for repeats — repeats mean the drive is wearing out. |
| **11** | Windows (`disk`) | Controller error — drive, cable, or controller misbehaving | 🟡 | Auto-repair runs. If it repeats, check/replace the SATA cable first, then suspect the drive. |
| **51** | Windows (`disk`) | Error while swapping memory to/from disk (paging) | 🟡 | Auto-repair runs. Occasional ones are harmless; a flood means drive trouble. |
| **52** | Windows (`disk`) | **The drive itself predicts it will fail (SMART warning)** | 🔴 | Back up the machine and replace the drive. This is the drive's own death announcement — believe it. |
| **153** | Windows (`disk`) | A disk operation had to be retried | 🟡 | Auto-repair runs. Occasional = busy disk. Frequent = failing drive, cable, or overloaded VM host. |
| **50** | Windows (`Ntfs`) | Windows couldn't save data it promised to write — **data was lost** | 🟡 | Auto-repair runs. Note which machine; repeats alongside `disk` codes = dying drive. |
| **55** | Windows (`Ntfs`) | File system structure is corrupted — the classic "disk corruption" event | 🟡 | Auto-repair runs (this is the main code the whole disk system was built for). |
| **57** | Windows (`Ntfs`) | Failed to flush data to the transaction log | 🟡 | Auto-repair runs. Same watch-for-repeats rule as 50. |
| **98** | Windows (`Ntfs`) | Volume health report — the Error/Warning version means the volume needs attention | 🟡 | Auto-repair runs. (The harmless "volume is healthy" version is filtered out by our monitor settings.) |
| **9000** | Our robot (`AutoDiskRepair`) | ✅ Disk verified clean — repair succeeded or nothing was wrong | 🟢 | Nothing. This code closes the disk alert automatically. |
| **9001** | Our robot (`AutoDiskRepair`) | Repair scheduled — machine must REBOOT for the deep repair to run | 🟡 | Reboot the machine (or let the maintenance window do it). Alert stays open until the post-reboot check passes. |
| **9002** | Our robot (`AutoDiskRepair`) | 🚨 **Drive hardware is failing. Robot refused to continue.** | 🔴 | Back up / image the machine NOW, replace the drive. Do not keep running repairs on it. |
| **9003** | Our robot (`AutoDiskRepair`) | Deep repair ran at reboot but the disk is STILL corrupted | 🔴 | Back up, then investigate by hand. Usually means hardware is failing in a way SMART hasn't flagged yet. |
| **8000** | Our robot (`SpoolerRepair`) | ✅ Print spooler restarted and verified working | 🟢 | Nothing. This code closes the spooler alert automatically. |
| **8001** | Our robot (`SpoolerRepair`) | Print queue was wiped to fix the spooler — queued jobs were deleted | 🟡 | Users may need to reprint. If one machine gets this repeatedly, hunt for the bad printer/driver (see §5). |
| **8002** | Our robot (`SpoolerRepair`) | 🚨 **Spooler keeps crashing even with a clean queue. Left stopped.** | 🔴 | Almost always a faulty printer driver. See the troubleshooting steps in §5. |
| **8003** | Our robot (`SpoolerRepair`) | Spooler is disabled ON PURPOSE on this machine (security hardening) | 🟡 | The machine is fine. Fix the monitoring: unassign the spooler monitor from this device. |

---

## 3. Disk Monitoring — The Windows Warning Codes

These are the codes **Windows itself** writes when a drive is struggling. They all live in the **System** event log. Any of them (two within an hour, per our threshold) triggers the alert that runs `AutoDiskRepair.ps1`.

A useful mental model: a hard drive failing is like a road wearing out. First you get the occasional pothole (7), then drivers start swerving and retrying (153, 51), then the road crew starts losing cargo (50, 57), then the map itself no longer matches the road (55, 98). And sometimes the road inspector simply condemns it in advance (52).

### Source: `disk` — the hardware layer

These come from the low-level driver that talks to the physical drive. They're about the *hardware*.

- **Event 7 — Bad block.** *"The device has a bad block."* The drive tried to read or write a physical spot on the disk and found it damaged. Drives can quietly remap a few bad blocks around — that's normal aging — but every one Windows actually *reports* means the drive stumbled in a way software noticed. One or two per year: fine. Several per month: the drive is dying; plan a replacement even if the repair script keeps reporting success.

- **Event 11 — Controller error.** *"The driver detected a controller error."* The conversation between Windows and the drive broke down. Three usual suspects, cheapest first: a loose or failing **SATA/power cable**, the disk **controller** on the motherboard, or the **drive** itself. If a machine logs these repeatedly, reseat/replace the cable before condemning the drive — it's a $5 fix that solves a surprising number of these.

- **Event 51 — Paging error.** Windows constantly moves data between RAM and disk (called *paging*). This code means one of those transfers hit an error. It's the noisiest code we monitor — busy disks, sleep/resume cycles, and USB drives all cause occasional harmless ones, which is why our threshold requires two events within an hour before alerting. A steady stream of 51s, especially with 7s or 153s alongside, is a failing drive.

- **Event 52 — SMART predicted failure. THE BIG ONE.** 🔴 Every modern drive runs a built-in self-test system called **SMART** (Self-Monitoring, Analysis and Reporting Technology). Event 52 means the drive's own self-test concluded: *"I am going to fail. Back up your data immediately."* Drives don't say this lightly — treat it as a formal death notice. Sometimes you get weeks of warning, sometimes days. **Do not wait, do not run repairs** — back up or image the machine and replace the drive. (The repair script protects you here: it checks drive health before doing anything and will refuse to run repairs on a drive in this state, raising code 9002 instead.)

- **Event 153 — IO operation retried.** A read or write didn't complete on the first try and had to be re-issued. Think of it as the drive saying "sorry, what?" — once in a while is nothing (especially on virtual machines or busy servers, where storage latency causes this), but frequent retries are the classic early sign of a drive, cable, or controller on the way out.

### Source: `Ntfs` — the filing-system layer

**NTFS** is the filing system Windows uses to organize data on the drive — the index cards that record which file lives where. These codes mean the *organization* of the data is damaged, which is usually (but not always) caused by the hardware problems above, a power loss mid-write, or a crash.

- **Event 50 — Delayed write failed, data lost.** Windows holds data in memory briefly before writing it to disk (it's faster). This code means that write failed and **the data in question is gone**. A user's document may be damaged. Occasional occurrences happen with removable drives; on an internal drive, repeats mean real trouble.

- **Event 55 — File system corruption detected.** The flagship corruption event, and the original reason this whole monitoring system exists. NTFS found that its own records are damaged — the index cards no longer match reality. This is exactly what the repair script's chkdsk step fixes. One event 55, repaired, verified clean (9000), never seen again: fine. Recurring 55s on the same machine: the corruption has a cause, and it's almost always failing hardware — check what `disk`-source codes that machine has been logging.

- **Event 57 — Failed to flush transaction log.** NTFS keeps a journal of changes so it can recover cleanly from crashes. This code means it couldn't save that journal to disk. Same family and same response as 50.

- **Event 98 — Volume health report.** NTFS periodically reports the health of each volume. The **Error/Warning** versions say the volume needs repair. There is also a completely harmless **Information** version ("Volume C: is healthy. No action is needed.") that Windows logs routinely at startup — our monitors are configured to ignore Information-level events specifically so that friendly version can't raise false alarms. If you ever rebuild these monitors, keeping that type filter is essential.

---

## 4. Disk Monitoring — The Robot's Report Codes (9000-series)

When the alert fires, `AutoDiskRepair.ps1` runs on the machine. In plain English, it:

1. **Asks the drive if it's dying** (SMART health check). If yes → stops immediately and reports 9002. Running heavy repairs on a dying drive can finish it off, so the robot refuses.
2. **Scans the file system online** (no downtime, users unaffected).
3. **Repairs Windows' own system files** (the DISM and SFC tools — think of these as restoring Windows' factory-original files from a known-good source).
4. **If corruption was found:** schedules the deep repair (**chkdsk**) to run at the next reboot — Windows can't deeply repair the drive it's actively running from, the same way you can't rebuild a road while driving on it. It also plants a one-time task that re-checks the disk after that reboot and reports the final result.

**Burst protection (added Aug 12, 2026):** a single disk hiccup often writes many events in the *same second*, and the RMM raises one alert — and launches one copy of the script — per matching event (observed: 10 launches in one second). The RMM's event monitors offer no de-duplication setting, so the script guards itself: if another copy is already running, or a run completed within the last 60 minutes, the duplicate exits immediately (exit code 0, no events written, disk untouched). **A stack of same-second "Hard Disk Repair" entries in a device's activity feed during a burst is therefore expected** — only one of them does any work.

Its report codes (Application log, source `AutoDiskRepair`):

- **9000 — Volume verified clean.** 🟢 The all-clear. Either nothing was actually wrong, or the repair worked and a fresh scan confirmed the disk is healthy. **This code automatically closes the disk alert** — if you never saw the alert, this is why. No action.

- **9001 — Repair scheduled, reboot required.** 🟡 Corruption was found and the deep repair is queued for the next restart. **The fix has NOT happened yet.** Action: restart the machine (after hours is fine — coordinate with the user). Two things to know: the reboot will take noticeably longer than normal because the repair runs during startup — on a large traditional hard drive it can run **for hours** and the machine may look stuck. It isn't. Warn the user, don't power-cycle it. After the reboot, the robot re-checks the disk and reports either 9000 (fixed — alert closes) or 9003 (still broken — see below).

- **9002 — Drive hardware failing, robot stood down.** 🔴 The SMART health check failed. The robot deliberately did *not* attempt repairs because stressing a dying drive accelerates the failure. **Action: treat as urgent.** Back up or image the machine immediately, replace the drive, restore. Every day of delay is a gamble on the drive's remaining life. This code raises its own High-priority alert and never closes automatically — that's intentional.

- **9003 — Still corrupted after the deep repair.** 🔴 The chkdsk ran at reboot, and the follow-up scan *still* found corruption. When repairs don't hold, the corruption has an active cause — usually hardware failing in a way SMART hasn't flagged yet, occasionally a bad RAM stick corrupting data on its way to disk. **Action:** back up the machine first (protect the data before experimenting), then investigate: run a manual `chkdsk` and read its output, check the machine's recent `disk`-source events, consider a RAM test, and lean toward drive replacement if anything else supports it.

**Script exit codes** (shown in the RMM's script-run history, not in alerts): `0` = clean/fixed, `1` = repair scheduled awaiting reboot, `2` = failing hardware (matches 9002), `3` = the script itself hit an error (check the script output).

---

## 5. Print Spooler Monitoring

The **print spooler** is the Windows service that manages all printing — when it stops, nobody on that machine can print. This monitor isn't watching event codes; it watches the **service itself**, and alerts when the spooler has been down for 5+ minutes. The alert runs `SpoolerRepair.ps1`, which works in escalating tiers:

1. **First failure:** restart the service and verify it stays up.
2. **Second failure within 30 minutes:** a plain restart didn't hold, which almost always means a **corrupted print job** is stuck in the queue and crashes the spooler every time it starts (a "poisoned" job). The robot wipes the print queue, then restarts.
3. **Third failure within 30 minutes:** even a clean queue didn't help. The robot **stops trying and leaves the service stopped** so the alert stays open — at this point it's a defective printer driver, and endless auto-restarts would just hide the problem.

Its report codes (Application log, source `SpoolerRepair`):

- **8000 — Spooler restarted and verified.** 🟢 Fixed. Closes the alert automatically. No action.

- **8001 — Print queue was purged.** 🟡 The robot deleted everything waiting in the print queue to recover the spooler. It worked — but **users' pending print jobs were deleted** and they'll need to reprint. No ticket needed for a one-off. **The pattern to watch:** the same machine logging 8001 weekly means some specific printer, driver, or document type keeps generating poison jobs — find it (ask the user what they printed just before each incident) and fix the driver rather than letting the robot keep mopping up.

- **8002 — Spooler crash-looping, left stopped.** 🔴 Restarts failed, a clean queue failed; a human is needed and **printing on this machine is down until you act.** This is a faulty-driver problem in nearly every case. Troubleshooting order:
  1. Open Event Viewer → Windows Logs → Application, and look for **"Application Error" events naming `spoolsv.exe`** — the "faulting module" line usually names the DLL of the guilty driver.
  2. Ask what changed: a newly added printer, a driver update, a new label/receipt printer are the usual suspects.
  3. Remove or roll back that driver (Print Management console → All Drivers), then start the spooler manually.
  4. If it holds, done — the next successful robot run reports 8000 and closes the escalation alert.

- **8003 — Spooler is disabled on purpose.** 🟡 Some machines (domain controllers, most servers) have the spooler **deliberately disabled** as a security measure (the "PrintNightmare" vulnerability made this standard practice). The robot detected that this machine is one of them and correctly refused to fight the policy. **The machine is fine — the monitoring is wrong.** Action: unassign the spooler monitor from this device so it stops alerting.

**Script exit codes:** `0` = spooler running, `1` = crash-loop, human needed (matches 8002), `2` = disabled by policy (matches 8003), `3` = script error.

---

## 6. Monitor Configuration Reference

Everything needed to rebuild or audit the monitors. All event monitors use event types **Critical + Error + Warning only** (never Information — see Event 98 above for why), except the auto-resolution rules, which specifically match Information.

### Disk system

| Setting | Trigger A (hardware) | Trigger B (file system) | Escalation ("needs a human") |
|---|---|---|---|
| Event Log Name | `System` | `System` | `Application` |
| Event Source Name | `disk` | `Ntfs` | `AutoDiskRepair` |
| Event codes | 7, 11, 51, 52, 153 | 50, 55, 57, 98 | 9002, 9003 |
| Event types | Critical, Error, Warning | Critical, Error, Warning | Error |
| Threshold | 2 times in 60 min | 2 times in 60 min | 1 time in 60 min |
| Priority | Moderate | Moderate | **High** |
| Response | `AutoDiskRepair.ps1` (timeout ≥ 90 min) | `AutoDiskRepair.ps1` (timeout ≥ 90 min) | none |
| Auto-resolution | `AutoDiskRepair` code 9000, Information, 1 in 60 min | same | same |

Optional third monitor if desired: `System` / `disk` / code **52** alone at **1 time in 60 min** — a SMART death notice should never wait for a second event. (With the 2-in-60 threshold on Trigger A, a lone 52 waits for company; in practice dying drives are chatty, but the dedicated monitor removes the gamble.)

Burst behavior: these monitors raise **one alert per matching event** with no de-duplication option, so an event burst launches many simultaneous copies of the response script. The de-duplication lives in `AutoDiskRepair.ps1` itself (single-instance mutex + 60-minute cooldown; duplicates exit 0 without writing events). If the script is ever rebuilt, that guard must be kept.

### Print spooler system

| Setting | Service monitor | Escalation ("needs a human") |
|---|---|---|
| Watches | Service `Spooler` — *is Not Running* (or *is Stopped*) for 5 min | Event log |
| Event Log / Source | — | `Application` / `SpoolerRepair` |
| Event codes | — | 8002, 8003 |
| Event types | — | Error, Warning |
| After boot delay | 15 minutes | — |
| Priority | Moderate | **High** |
| Auto resolve | After 1 minute (no longer applicable) | `SpoolerRepair` code 8000, Information, 1 in 60 min |
| Response | `SpoolerRepair.ps1` (timeout 5 min) | none |
| Targets | Workstations & print servers only — never DCs or spooler-disabled servers | same devices |

---

## 7. Codes We Deliberately Left Out (and why)

The original disk monitor watched codes **55 98 153 7 51 9 33 57**. Two were dropped on purpose — if you're ever tempted to re-add them, here's the history:

- **Event 9** (controller timeout) — a real signal, but it's logged by whatever storage *driver* each machine uses (`storahci`, `iaStorA`, `stornvme`, RAID vendors…), so no single source-filtered monitor can catch it, and unfiltered it collides with unrelated components that also use ID 9. Timeouts severe enough to matter cascade into codes 153/7, which Trigger A catches anyway. Re-add it only as its own monitor scoped to a specific driver source if the fleet standardizes on one.
- **Event 33** — with no source filter this matched unrelated housekeeping events (shadow-copy cleanup and others) and was likely a meaningful chunk of the old monitor's false alarms. There is no well-known disk-health event at ID 33.
- **Event 137** (`Ntfs` transaction errors) — genuine but notoriously noisy on machines running backup software (it often fires during routine snapshot operations). Left out unless evidence shows it appearing alongside `disk`-source codes.
- **Event 157** (`disk` — surprise removal) — legitimate for flaky cables/enclosures, but it also fires every time someone yanks a USB drive without ejecting. Add to Trigger A only if the fleet has few removable drives.

---

## 8. Glossary

| Term | Plain-English meaning |
|---|---|
| **Event log** | Windows' built-in diary of everything that happens. Viewable with the Event Viewer app. |
| **Event source** | Which component wrote a diary entry (e.g., `disk` = the disk driver, `Ntfs` = the filing system, `AutoDiskRepair` = our robot). |
| **Event code / ID** | The number identifying what kind of entry it is. Codes are only meaningful *together with* their source — different sources reuse the same numbers. |
| **SMART** | Every drive's built-in self-test. When SMART predicts failure, the drive is formally telling you it's dying. |
| **NTFS** | The filing system Windows uses to track which file lives where on the drive. "Corruption" usually means this index is damaged, not the files themselves. |
| **chkdsk** | Windows' disk repair tool ("check disk"). Deep repairs must run during startup, before Windows is fully awake — which is why some fixes require a reboot. |
| **DISM / SFC** | Tools that verify and restore Windows' own system files from known-good copies. |
| **Print spooler** | The Windows service that manages the printing queue. Spooler down = no printing on that machine. |
| **Poisoned print job** | A corrupt document in the queue that crashes the spooler every time it starts. Cured by wiping the queue. |
| **Auto-resolution** | The RMM rule that closes an alert automatically when the "all clear" code (9000 / 8000) appears. |
| **Escalation monitor** | The second monitor watching for the robots' "I give up" codes (9002, 9003, 8002) — the ones that mean a human must step in. |
