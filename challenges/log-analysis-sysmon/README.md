# Log Analysis - Sysmon

## Challenge Information

| Field | Details |
|---|---|
| Challenge | Log Analysis - Sysmon |
| Category | Log Analysis / Malware Analysis |
| Platform | Blue Team Labs Online (BTLO) |
| Lab Environment | Kali Linux |
| Evidence Type | Sysmon JSON event log |
| Completion Date | 24 September 2026, 07:40 |
| Status | Completed |

## Overview

This challenge focuses on investigating Windows Sysmon events stored in a JSON file to reconstruct attacker activity.

The investigation covers an HTA file used to gain initial access, PowerShell-based malware downloading, environment variable manipulation, LOLBIN abuse, malware execution, dependency analysis, privilege escalation tooling, and reverse shell activity.

The objective is to use log-analysis techniques to identify suspicious processes, command lines, downloaded files, DLL dependencies, and network indicators.

## Evidence

- `screenshot.png` - BTLO challenge completion screenshot.
- `sysmon-events.json` - Sysmon JSON evidence file referenced in the challenge notes. The original evidence file is not included in this repository.

The findings below are documented from the challenge investigation notes. The original JSON evidence was not independently re-acquired and re-analysed during repository preparation.

## Investigation Methodology

### 1. Initial File Inspection

The first step is to identify the evidence file format and inspect its initial records.

    cd ~/Downloads
    file sysmon-events.json
    head -5 sysmon-events.json

These commands help confirm the file type and provide an initial view of the event structure.

### 2. Initial Access - HTA File

The investigation identified `updater.hta` in the attack chain.

    grep -i "\.hta" sysmon-events.json

The challenge notes indicate that `mshta.exe` executed `updater.hta` from the Downloads directory.

HTA files are HTML Applications that can be executed through Microsoft HTML Application Host (`mshta.exe`). Their execution can be abused to run attacker-controlled scripts and commands.

**Finding:** `updater.hta`

### 3. PowerShell Download Activity

The next step was to identify the PowerShell command used to retrieve the malware and the destination port.

    grep -i "CommandLine" sysmon-events.json | grep -iE "http|download"

The notes identify `Invoke-WebRequest` as the PowerShell cmdlet used to download `supply.exe`.

The download URL was:

    http://192.168.1.11:6969/supply.exe

**Findings:**
- PowerShell cmdlet: `Invoke-WebRequest`
- Destination IP: `192.168.1.11`
- Destination port: `6969`
- Downloaded file: `supply.exe`

### 4. Environment Variable Manipulation

The investigation identified manipulation of the Windows `COMSPEC` environment variable.

    grep -i "CommandLine" sysmon-events.json | grep -i "set "

The challenge notes report that the variable was redirected to:

    comspec=C:\windows\temp\supply.exe

`COMSPEC` normally identifies the command interpreter used by Windows, typically `cmd.exe`. Redirecting it to an unexpected executable is suspicious and may indicate an attempt to interfere with normal command execution.

**Finding:** `COMSPEC` was redirected to `C:\windows\temp\supply.exe`.

### 5. LOLBIN Abuse

The investigation identified `ftp.exe` as the legitimate Windows binary abused during the attack.

    grep -i "CommandLine" sysmon-events.json | grep -iE "ftp|powershell|cmd|wscript"

LOLBIN stands for Living Off The Land Binary. It describes a legitimate operating-system binary that an attacker abuses to perform malicious actions.

**Finding:** `ftp.exe`

### 6. First Command Executed by the Malware

The challenge notes identify `ipconfig` as the first command executed by `supply.exe`.

    grep -i "supply.exe" sysmon-events.json | grep -i "CommandLine"

`ipconfig` displays network configuration information. Its execution may indicate basic host and network reconnaissance.

**Finding:** `ipconfig`

### 7. Malware Language Identification

DLL dependency events were reviewed to identify the runtime associated with the malware.

    grep -i "\.dll" sysmon-events.json | sort -u | head -30

The notes identify `python27.dll` among the dependencies loaded by `supply.exe`.

This is consistent with a binary that embeds or relies on the Python 2.7 runtime.

**Finding:** Python

The presence of `python27.dll` supports the Python attribution. It does not, by itself, prove which packaging tool was used.

### 8. Additional Download - JuicyPotato

The investigation identified a subsequent download from GitHub.

    grep -i "CommandLine" sysmon-events.json | grep -iE "http"

The URL recorded in the challenge notes is:

    https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe

JuicyPotato is a Windows privilege escalation tool associated with abuse of certain Windows token impersonation privileges.

**Finding:** `JuicyPotato.exe`

### 9. Reverse Shell Activity

The final investigation step focused on the network connection attempted by `nc.exe`.

    grep -i "CommandLine" sysmon-events.json | grep -iE "nc|netcat"

The notes report a connection attempt to `192.168.1.11` on port `9898`.

**Findings:**
- Utility: `nc.exe`
- Destination IP: `192.168.1.11`
- Destination port: `9898`

This is consistent with attempted reverse shell activity. The command-line evidence described in the notes does not independently prove that the connection succeeded.

## Questions and Answers

| # | Question | Answer |
|---:|---|---|
| 1 | What is the file that gave access to the attacker? | `updater.hta` |
| 2 | What is the PowerShell cmdlet used to download the malware file and what is the port? | `Invoke-WebRequest`, `6969` |
| 3 | What is the name of the environment variable set by the attacker? | `comspec=C:\windows\temp\supply.exe` |
| 4 | What is the process used as a LOLBIN to execute malicious commands? | `ftp.exe` |
| 5 | What is the first command executed by the malware? | `ipconfig` |
| 6 | What language is the malware written in? | `python` |
| 7 | What is the full URL of the downloaded file? | `https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe` |
| 8 | What port does the attacker attempt to use for the reverse shell? | `9898` |

## Consolidated Command Reference

The following commands provide a neutral starting point for analysing the evidence without directly including the expected answers in the search terms.

**Initial inspection**

    file sysmon-events.json
    head -5 sysmon-events.json

**HTA execution indicators**

    grep -i "\.hta" sysmon-events.json

**Download-related command lines**

    grep -i "CommandLine" sysmon-events.json | grep -iE "http|download"

**Environment variable changes**

    grep -i "CommandLine" sysmon-events.json | grep -i "set "

**Potential LOLBIN activity**

    grep -i "CommandLine" sysmon-events.json | grep -iE "ftp|powershell|cmd|wscript"

**Malware-related command lines**

    grep -i "supply.exe" sysmon-events.json | grep -i "CommandLine"

**DLL dependency indicators**

    grep -i "\.dll" sysmon-events.json | sort -u | head -30

**Potential network utility activity**

    grep -i "CommandLine" sysmon-events.json | grep -iE "nc|netcat"

These searches are useful for initial triage. For a complete investigation, JSON-aware parsing with `jq` is preferable when the event structure permits it. Results should be correlated by timestamp, process ID, parent process ID, image path, and command line rather than interpreted as isolated strings.

## DFIR Findings

The challenge notes describe the following attack sequence:

1. `mshta.exe` executes `updater.hta`, providing the initial execution path.
2. PowerShell uses `Invoke-WebRequest` to download `supply.exe` from `192.168.1.11:6969`.
3. The attacker modifies `COMSPEC` to point to the downloaded executable.
4. `ftp.exe` is abused as a LOLBIN.
5. `supply.exe` executes `ipconfig`, consistent with network reconnaissance.
6. The presence of `python27.dll` indicates a Python runtime dependency.
7. The attacker downloads `JuicyPotato.exe`.
8. `nc.exe` attempts to establish a reverse shell connection to `192.168.1.11:9898`.

This sequence provides a useful investigative model for correlating Sysmon process and network events.

## Key Indicators

| Indicator | Value | Context |
|---|---|---|
| Initial access file | `updater.hta` | HTA execution |
| Initial execution process | `mshta.exe` | HTML Application Host |
| Downloaded malware | `supply.exe` | PowerShell download |
| Download cmdlet | `Invoke-WebRequest` | File retrieval |
| Download source | `192.168.1.11:6969` | Internal lab address and port |
| Modified variable | `COMSPEC` | Environment variable manipulation |
| LOLBIN | `ftp.exe` | Living Off The Land Binary |
| Reconnaissance command | `ipconfig` | Network configuration discovery |
| Runtime dependency | `python27.dll` | Python 2.7 runtime |
| Additional tool | `JuicyPotato.exe` | Privilege escalation tooling |
| Reverse shell utility | `nc.exe` | Netcat |
| Reverse shell destination | `192.168.1.11:9898` | Attempted connection |

## Learning Outcomes

- Searching Sysmon JSON logs with `grep`.
- Identifying suspicious HTA execution through `mshta.exe`.
- Recognising PowerShell download activity.
- Understanding suspicious `COMSPEC` manipulation.
- Identifying LOLBIN abuse.
- Recognising basic host and network reconnaissance.
- Using DLL dependencies as clues for identifying malware runtimes.
- Identifying privilege escalation tooling in process command lines.
- Recognising possible reverse shell activity.
- Correlating process execution and network indicators during incident investigation.

## Limitations and Evidence Notes

- Findings and answers are transcribed from the original challenge notes.
- The original `sysmon-events.json` file was not independently re-acquired or re-analysed while preparing this repository entry.
- The exact timestamps, process IDs, parent process relationships, and event-level sequence have not been independently verified here.
- The presence of `python27.dll` supports a Python runtime attribution but does not independently establish the malware's complete development or packaging process.
- The reported reverse shell command indicates an attempted connection; successful communication is not established by the notes alone.
- The reported use of JuicyPotato does not independently prove that privilege escalation succeeded.
- No malware was executed as part of preparing this documentation.

## Conclusion

This challenge demonstrates how Sysmon telemetry can be used to reconstruct suspicious Windows activity through process execution, command-line arguments, DLL dependencies, and network indicators.

The investigation highlights the importance of correlating multiple events to understand the attack sequence rather than relying on a single suspicious process or command.

The practical techniques are directly relevant to BTL1 preparation, particularly Windows log analysis, malware investigation, and incident timeline reconstruction.
