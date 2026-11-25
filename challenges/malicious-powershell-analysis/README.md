# Malicious PowerShell Analysis

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Challenge | Malicious PowerShell Analysis |
| Category | Malware Analysis / PowerShell Analysis |
| Environment | Kali Linux |
| Training Context | BTL1 (Blue Team Level 1) preparation |
| Completion Date | 24 September 2026, 12:10 |
| Malware Family | Emotet |

## Overview

This challenge focuses on the static analysis of an obfuscated PowerShell payload delivered as a Base64-encoded command.

The objective was to decode the PowerShell content, identify its behavior, reconstruct obfuscated strings, extract indicators of compromise (IoCs), and determine the malware family associated with the activity.

The analysis identified:

- PowerShell Base64 encoding using UTF-16LE.
- Obfuscated configuration of the TLS 1.2 security protocol.
- Construction of a directory path under `\HOME`.
- A DLL payload named `A69S.dll`.
- Execution through the legitimate Windows utility `rundll32.exe`.
- A remote URI hosted on `wm.mcdevelop.net`.
- Emotet as the challenge's identified malware family.

## Tools and Environment

| Tool | Purpose |
|---|---|
| Kali Linux terminal | File navigation and command execution |
| Python 3 | Base64 decoding and UTF-16LE conversion |
| grep | Searching decoded content for relevant strings |
| cat | Inspecting text files |
| Mousepad / nano | Viewing and editing scripts |
| CyberChef | Alternative method for decoding encoded content |

## Preparation

The challenge files were located in the working directory:

`~/Downloads/BTLO PowerShELL Analysis`

The initial files included:

- `BTLO.txt`
- `ps_script.txt`
- `decode_ps.py`

The file `ps_script.txt` contained the encoded PowerShell content, including the Base64 data following the `-ENCOD` argument.

Initial inspection:

- `cd ~/Downloads/BTLO\ PowerShELL\ Analysis`
- `ls -la`
- `cat ps_script.txt`

## Investigation Workflow

### 1. Decode the PowerShell payload

PowerShell encoded commands use Base64 encoding with UTF-16LE text representation.

The Python decoding procedure performed the following operations:

1. Read the input file.
2. Locate long Base64-like strings using a regular expression.
3. Select the longest matching string.
4. Decode the Base64 data.
5. Convert the decoded bytes from UTF-16LE.
6. Save the resulting PowerShell content to `decoded_output.txt`.

The regular expression used was:

`[A-Za-z0-9+/=]{100,}`

The decoding results recorded in the challenge notes:

| Item | Result |
|---|---:|
| Original input length | 5,397 characters |
| Extracted Base64 string length | 5,352 characters |
| Decoded data size | 4,014 bytes |
| Decoded text length | 2,007 characters |
| Output file | `decoded_output.txt` |

The decoded output provided readable PowerShell code for further static analysis.

### 2. Identify the security protocol

Neutral search command:

`grep -iE "protocol|security|tls|ssl" decoded_output.txt`

Relevant obfuscated expression:

`('T'+('ls'+'12'))`

String reconstruction:

- `T`
- `ls`
- `12`

Combined value:

`Tls12`

The script assigns this value to the PowerShell `SecurityProtocol` property.

**Finding: TLS 1.2**

### 3. Reconstruct the created directory

Neutral search command:

`grep -iE "HOME|Db_bh|ReP" decoded_output.txt`

The script constructs a directory path using string concatenation and a replacement operation.

Relevant elements:

- `('UO'+'H')` produces `UOH`.
- `[char]92` represents the backslash character (`\`).
- The `Replace` operation converts the `UOH` marker into a backslash.

The reconstructed directory path is:

`\\HOME\Db_bh30\Yf5be5g\`

**Finding: `\HOME\Db_bh30\Yf5be5g\`**

### 4. Identify the downloaded file

Neutral search command:

`grep -iE "\.dll|Swrp6tc" decoded_output.txt`

Relevant obfuscated expressions:

- `(('A6'+'9')+'S')` produces `A69S`.
- `('.'+'dl'+'l')` produces `.dll`.

Combined filename:

`A69S.dll`

**Finding: `A69S.dll`**

### 5. Identify the execution utility

Neutral search command:

`grep -iE "rundll|ToStRing" decoded_output.txt`

Relevant expression:

`&('r'+'undl'+'l32')`

String reconstruction:

`r` + `undl` + `l32` = `rundll32`

The PowerShell call operator (`&`) invokes the reconstructed command.

`rundll32.exe` is a legitimate Windows utility that can execute DLL-related functionality. Its use in this context is suspicious because the script invokes it with the downloaded DLL.

**Finding: `rundll32`**

### 6. Extract the remote domain

Neutral search command:

`grep -iE "6F2gd|wm.mcdeve" decoded_output.txt`

Relevant string fragments:

- `wm.mcdeve`
- `lop.net`
- `/content/`
- `6F2gd/`

Reconstructed URI:

`wm.mcdevelop.net/content/6F2gd/`

**Finding: `wm.mcdevelop.net`**

### 7. Identify the malware family

The domain was researched as part of the challenge investigation.

The challenge notes associate `wm.mcdevelop.net` with Emotet. The observed script behavior also includes downloading a DLL and invoking it through `rundll32`.

**Finding: Emotet**

This is the challenge's malware-family identification. The domain association should be treated as an investigated IoC, not as independent proof of malware execution on a victim system.

## Consolidated Answers

| Question | Answer |
|---|---|
| Q1. What security protocol is used for communication with the malicious domain? | TLS 1.2 |
| Q2. What directory does the obfuscated PowerShell create? | `\HOME\Db_bh30\Yf5be5g\` |
| Q3. What file is being downloaded? | `A69S.dll` |
| Q4. What is used to execute the downloaded file? | `rundll32` |
| Q5. What is the domain name of the URI ending in `/6F2gd/`? | `wm.mcdevelop.net` |
| Q6. What is the name of the malware? | Emotet |

## Obfuscation Analysis

| Obfuscated expression | Reconstructed value | Technique |
|---|---|---|
| `'T'+'ls'+'12'` | `Tls12` | String concatenation |
| `'UO'+'H'` | `UOH` | String concatenation |
| `Replace('UOH', '\')` | `\` | String replacement |
| `'A6'+'9'+'S'` | `A69S` | String concatenation |
| `'.'+'dl'+'l'` | `.dll` | String concatenation |
| `'r'+'undl'+'l32'` | `rundll32` | String concatenation |
| `'wm.mcdeve'+'lop.net'` | `wm.mcdevelop.net` | String concatenation |

## Neutral Command Reference

The following commands were used to search the decoded script without directly providing the answers:

| Investigation area | Command |
|---|---|
| Security protocol | `grep -iE "protocol|security|tls|ssl" decoded_output.txt` |
| Directory construction | `grep -iE "HOME|Db_bh|ReP" decoded_output.txt` |
| DLL filename | `grep -iE "\.dll|Swrp6tc" decoded_output.txt` |
| Execution utility | `grep -iE "rundll|ToStRing" decoded_output.txt` |
| Remote domain and URI | `grep -iE "6F2gd|wm.mcdeve" decoded_output.txt` |

## DFIR Findings and Relevance

The decoded script contains several indicators and behaviors relevant to a SOC investigation:

| Indicator / Behavior | DFIR relevance |
|---|---|
| Base64-encoded PowerShell | May conceal malicious commands from basic inspection |
| UTF-16LE encoding | Important when decoding PowerShell `-EncodedCommand` content |
| String concatenation | Obfuscates security-sensitive values and indicators |
| TLS 1.2 configuration | Identifies the configured communication protocol |
| `A69S.dll` | Potential downloaded malware payload |
| `rundll32.exe` | Legitimate Windows utility that can be abused for DLL execution |
| `wm.mcdevelop.net` | Network indicator requiring reputation and historical investigation |
| Emotet | Malware-family identification from the challenge investigation |

## Key Learning Points

- PowerShell `-EncodedCommand` content can be decoded using Base64 and UTF-16LE conversion.
- String concatenation and replacement operations can conceal commands, filenames, paths, and domains.
- Static analysis can reveal suspicious behavior without executing the payload.
- `rundll32.exe` is a legitimate Windows binary that can be abused by malware.
- Extracted domains and filenames can be used as starting points for IOC research and threat hunting.
- Neutral search commands help analysts discover relevant code before interpreting it.

## Limitations

This documentation is based on the challenge notes and the recorded decoding results.

The original challenge payload and a raw network capture are not included in this repository. Therefore:

- The downloaded DLL was not independently acquired or hashed as part of this documentation.
- No live connection to the identified domain was performed.
- No execution of the payload was performed for this write-up.
- The Emotet attribution reflects the challenge investigation and should not be interpreted as independent forensic confirmation.

The findings describe the behavior reconstructed from the decoded PowerShell content, not independently observed execution on a compromised endpoint.

## Conclusion

The investigation reconstructed an obfuscated PowerShell script that configures TLS 1.2, constructs a directory path, downloads `A69S.dll` from `wm.mcdevelop.net`, and invokes the DLL using `rundll32`.

The challenge identifies the associated malware family as Emotet.

This exercise demonstrates a practical static-analysis workflow for encoded PowerShell, string obfuscation, suspicious DLL downloads, LOLBin abuse, and IOC extraction.

## Evidence

The original BTLO challenge screenshot is included in this directory:

- `screenshot.png`
