# Reverse Engineering - A Classic Injection

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Category | Malware Analysis / Reverse Engineering |
| Environment | Kali Linux and isolated Windows analysis VM |
| Training Context | BTL1 preparation |
| Completion Date | 21 September 2026, 12:45 |

## Overview

This challenge examines a Windows executable that uses process injection to execute shellcode in a target process. The investigation combines static PE analysis, disassembly, string inspection, and dynamic process monitoring.

## Tools

- `strings`, `grep`, `awk` - extract and filter readable strings.
- `rabin2` - inspect PE metadata and imported functions.
- `radare2` - disassemble and analyse program behaviour.
- CyberChef - decode Base64-encoded PowerShell commands.
- Process Monitor (ProcMon) - monitor process creation and file-system activity in the Windows VM.

## Preparation

The supplied archive was extracted in Kali Linux. The password recorded in the challenge notes was `infected`.

## Investigation Workflow and Answers

### Q1. Compiler

PE metadata and executable strings were inspected using `rabin2 -I` and `strings`.

**Answer:** Microsoft Visual C++

### Q2. Sleep Time

The `main` function was examined in Radare2 around the Sleep API call. The value `0x2bf20` converts to 180000 milliseconds, or 180 seconds.

**Answer:** 3 minutes

### Q3. Password

The extracted strings were filtered to identify the password.

**Answer:** `btlo`

### Q4. Shellcode Size

The disassembly around `VirtualAllocEx` showed an allocation-size argument of `0x1d9`.

**Answer:** 473 bytes

### Q5. Process-Injection API

The relevant imported API functions were `VirtualAllocEx`, `WriteProcessMemory`, and `CreateRemoteThread`.

**Answer:** `WriteProcessMemory`

### Q6. Victim Process

The target process identified in the challenge notes was:

**Answer:** `nslookup.exe`

### Q7. Created File

The dynamic-analysis notes describe monitoring the sample in a Windows VM with ProcMon.

**Answer:** `C:\Windows\Temp\btlo.txt`

### Q8. Message in the Created File

The Base64-encoded PowerShell command was decoded using CyberChef.

**Answer:** `Welcome to BTLO!`

### Q9. Program Used by the Shellcode

The dynamic-analysis notes identify the program launched to create and write the file.

**Answer:** `powershell.exe`

## Consolidated Results

| Question | Finding |
|---|---|
| Q1 - Compiler | Microsoft Visual C++ |
| Q2 - Sleep time | 3 minutes |
| Q3 - Password | `btlo` |
| Q4 - Shellcode size | 473 bytes |
| Q5 - Injection API | `WriteProcessMemory` |
| Q6 - Target process | `nslookup.exe` |
| Q7 - Created file | `C:\Windows\Temp\btlo.txt` |
| Q8 - File message | `Welcome to BTLO!` |
| Q9 - Program used | `powershell.exe` |

## Analysis Notes and Limitations

The Q1-Q6 findings are documented from the static-analysis results in the supplied challenge notes.

The Q7-Q9 findings are transcribed from those notes, which describe a Windows VM and ProcMon workflow. The raw ProcMon capture and decoded command line were not included in the supplied text. Therefore, these are recorded challenge findings, not independently revalidated from the underlying dynamic-analysis artifacts.

The sample should only be examined in an isolated, controlled lab environment.

## Conclusion

The challenge demonstrates a process-injection workflow in which a Windows executable waits for three minutes, checks a password, and uses shellcode to interact with a target process. Static analysis identified the compiler, delay, password, shellcode allocation size, injection APIs, and target process. The recorded dynamic-analysis findings identify the created file, its message, and the PowerShell program involved.

## Evidence

- `screenshot.png` - BTLO challenge screenshot.
