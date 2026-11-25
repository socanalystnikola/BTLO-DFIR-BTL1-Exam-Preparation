# Suspicious USB Stick

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Category | Digital Forensics / Malware Analysis |
| Environment | Kali Linux |
| Training Context | BTL1 (Blue Team Level 1) exam preparation |
| Completion Date | 24 September 2026, 10:20 |
| Evidence | screenshot.png |

## Overview

This challenge investigates a suspicious USB drive discovered on an organization's premises. The objective is to examine its contents, identify potentially malicious files, and determine whether a PDF document contains suspicious functionality.

The investigation focuses on the USB autorun configuration, PDF file identification, static content inspection, and detection of potentially dangerous PDF actions.

## Tools and Techniques

- cat - inspect text-based configuration files
- sha256sum - calculate file hashes
- head and od - inspect file signatures and magic bytes
- strings and grep - extract and search readable strings
- grep -a and wc -l - identify and count PDF objects or keywords
- VirusTotal - check a file hash against security vendors
- Python - perform byte-level searches in a file

## Investigation Workflow

### 1. Extracting the USB Evidence

The password-protected archive was extracted in Kali Linux using:

    unzip USB.zip -d USB

The archive password provided by the challenge was btlo.

The investigation was performed in the extracted USB/autorun directory.

### 2. Inspecting the Autorun Configuration

The file autorun.inf was examined using:

    cat autorun.inf

Relevant contents:

    [autorun]
    open=README.pdf
    icon=autorun.ico

The configuration specifies README.pdf as the file to open.

### 3. Calculating the PDF SHA256 Hash

The SHA256 hash was calculated using:

    sha256sum README.pdf

The recorded VirusTotal result in the challenge notes was:

    43/63 security vendors flagged this file as malicious

This result indicates that the PDF was flagged by multiple security vendors.

### 4. Verifying the PDF Magic Number

The first four bytes were inspected using:

    head -c 4 README.pdf | od -A x -t x1z

Observed bytes:

    25 50 44 46

These bytes represent the %PDF file signature. The file therefore has the expected PDF magic number.

### 5. Identifying the Target Operating System

Readable strings were extracted and searched using:

    strings README.pdf | grep -iE "windows|linux|macos|darwin"

The output contained references to cmd.exe and c:\windows\system32.

These references indicate Windows-specific functionality.

### 6. Identifying the Referenced Executable

The PDF strings were searched for executable names:

    strings README.pdf | grep -iE "\.exe"

The relevant result was:

    /F{cmd.exe}

The executable referenced in the PDF was cmd.exe.

### 7. Searching for Suspicious OpenAction Elements

The PDF was searched for the /OpenAction keyword:

    grep -a -o "/OpenAction" README.pdf | wc -l

The recorded output was:

    1

This indicates that one /OpenAction occurrence was identified.

An alternative byte-level check using Python is:

    with open("README.pdf", "rb") as pdf:
        data = pdf.read()

    print(data.count(b"/OpenAction"))


## Challenge Questions and Answers

| No. | Question | Answer |
|---|---|---|
| 1 | What file is autorun.inf running? | README.pdf |
| 2 | Does the PDF file pass the VirusTotal scan? | False |
| 3 | Does the file have the correct magic number? | True |
| 4 | What operating system can the file exploit? | Windows |
| 5 | What Windows executable is mentioned in the PDF? | cmd.exe |
| 6 | How many suspicious /OpenAction elements are present? | 1 |

## DFIR Findings

The investigation identified several indicators that warrant security attention:

- The USB autorun configuration references a PDF document.
- The PDF was reported as malicious by 43 of 63 VirusTotal security vendors in the challenge notes.
- The file begins with the expected PDF signature, confirming that its header is consistent with a PDF document.
- Static strings contain Windows-specific references, including cmd.exe and the Windows system directory.
- One /OpenAction occurrence was identified in the PDF.

The /OpenAction entry is relevant because PDF documents can use document actions to trigger behavior when opened. Its presence alone does not prove that code was successfully executed.

Other PDF elements that may deserve examination include /JS, /JavaScript, /AA, /Launch, and /EmbeddedFile.

## Security Relevance

This challenge demonstrates how removable media can be used as an initial access vector.

A suspicious USB device may contain configuration files that direct a user toward a malicious document. PDF files should therefore be inspected for embedded scripts, automatic actions, launch references, and other potentially unsafe structures.

A SOC analyst should preserve the original evidence, calculate cryptographic hashes, inspect file metadata and structure, and correlate static findings with endpoint telemetry where available.

## Limitations

- The VirusTotal detection ratio is reproduced from the original challenge notes and was not independently rechecked during this documentation process.
- The presence of /OpenAction does not, by itself, establish successful execution.
- No execution of the suspicious PDF was performed as part of this documented static-analysis workflow.
- The findings describe the challenge evidence and should not be interpreted as proof of compromise on a real endpoint.

## Conclusion

The suspicious USB evidence contained an autorun configuration referencing a PDF document with multiple malicious indicators. Static inspection identified Windows-specific command references and an /OpenAction entry, while the recorded VirusTotal result showed significant vendor detection.

The challenge reinforces practical DFIR skills involving removable-media investigation, file-signature validation, hashing, string extraction, and basic PDF threat analysis.

## Evidence

The original BTLO challenge completion screenshot is included as:

- screenshot.png
