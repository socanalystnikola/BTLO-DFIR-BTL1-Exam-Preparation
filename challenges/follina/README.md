# Follina - CVE-2022-30190

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Category | Malware Analysis |
| Environment | Kali Linux |
| Training Context | BTL1 (Blue Team Level 1) preparation |
| Completion Date | 23 September 2026, 11:37 |
| Vulnerability | CVE-2022-30190 |

## Overview

This challenge focuses on the static analysis of a suspicious Microsoft Office document associated with the Follina vulnerability (CVE-2022-30190).

The investigation covers file identification, hash calculation, document extraction, XML relationship inspection, and identification of suspicious indicators.

The sample should not be executed on a production system.

## Tools and Utilities

- Kali Linux
- VirusTotal
- sha1sum
- file
- oletools / oleobj
- unzip
- grep
- Text editor

## Sample Preparation

The challenge archive password was:

`btlo`

The extracted document was named:

`sample.doc`

Although the filename uses the `.doc` extension, the challenge identified the file as an Office Open XML document. File identification should be based on its actual content and structure, not only its extension.

## Consolidated Results

| Question | Finding |
|---|---|
| SHA1 hash | 06727ffda60359236a8029e0b3e8a0fd11c23313 |
| File type | Office Open XML Document |
| Suspicious URL | https://www.xmlformats.com/office/word/2022/wordprocessingDrawing/RDF842l.html! |
| Relationship file | document.xml.rels |
| Reported size threshold | 4096 bytes |
| Reported process | msdt.exe |

## Technical Analysis

CVE-2022-30190, commonly known as Follina, is a Microsoft Support Diagnostic Tool (MSDT) vulnerability associated with specially crafted Office documents.

The investigation highlights several DFIR concepts:

- File hashing supports sample identification and evidence integrity checks.
- File signatures and internal structure can reveal a file type that differs from its extension.
- Office Open XML documents contain XML files and relationship definitions.
- External URLs embedded in document relationships can provide useful indicators of suspicious activity.
- Process names such as msdt.exe can guide endpoint investigation and event-log correlation.

## Limitations

The findings are based on the available challenge notes and documented investigation workflow. The original sample was not supplied with this repository for independent re-analysis.

- The SHA1 hash and challenge answers are transcribed from the available notes.
- The suspicious URL and relationship-file finding are recorded challenge results.
- The 4096-byte threshold and msdt.exe process observation were not independently verified using raw execution telemetry.
- No original process-monitoring capture or complete endpoint event-log evidence was available for validation.

The sample should be handled only in an isolated malware-analysis environment. Do not execute suspicious documents on a production workstation.

## Conclusion

The Follina challenge demonstrates how static examination of an Office document can reveal useful indicators for a DFIR investigation.

Hash calculation, file-type identification, archive extraction, XML relationship inspection, and identification of suspicious external references provide a structured starting point for investigating potentially malicious Office documents.

The exercise supports BTL1 preparation by reinforcing evidence handling, basic malware-analysis techniques, and the importance of distinguishing documented findings from independently verified evidence.

## Evidence

- screenshot.png - BTLO challenge screenshot.

