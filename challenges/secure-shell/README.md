# Secure Shell (SSH) Log Analysis

## Challenge Information

| Field | Details |
|---|---|
| Challenge | Secure Shell |
| Category | Log Analysis |
| Completion Date | 25 September 2026 |
| Completion Time | 15:13 |
| Environment | Linux |
| Purpose | BTL1 (Blue Team Level 1) exam preparation |

## Overview

This challenge focuses on analysing SSH server logs to identify connection activity, authentication attempts, successful logins, and relevant system information.

The investigation uses command-line tools available in Linux to extract and interpret information from the supplied `sshlog.log` file.

## Objectives

- Identify the most frequent IP addresses in the log.
- Determine which account successfully authenticated.
- Count successful password authentications.
- Identify the first recorded connection from the relevant source.
- Determine the SSH logging verbosity level.
- Investigate Windows OpenSSH log-file locations.

## Tools and Commands

- `grep` – search log entries using text and regular expressions.
- `awk` – extract selected fields from log entries.
- `cut` – extract specific fields.
- `sort` – sort extracted results.
- `uniq -c` – count repeated values.
- `wc -l` – count lines.
- `head` – display the beginning of a file.
- Google Search – research Windows OpenSSH logging locations.

## Investigation

### 1. Initial Log Inspection

The following commands were used to inspect the supplied log:

    cd ~/Downloads/1
    file sshlog.log
    wc -l sshlog.log
    head -20 sshlog.log

These commands identify the file type, count the log lines, and display the first 20 entries.

### 2. Identify the Most Frequent IP Addresses

Command:

    grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" sshlog.log | sort | uniq -c | sort -rn | head -10

Recorded results:

| Occurrences | IP address |
|---:|---|
| 14,624 | 192.168.1.17 |
| 3,730 | 192.168.1.20 |

The source notes identify `192.168.1.17` as the attacker IP address. It is a private IP address, indicating that the activity originated from an internal network address.

Recorded challenge answer:

    internal:192.168.1.17


### 3. Identify Successfully Authenticated Accounts

Command:

    grep -i "Accepted password" sshlog.log | awk '{print $9}' | sort -u

Recorded result:

    sophia

The investigation identified `sophia` as the account associated with successful password authentication. Other account names, including `admin`, `root`, and `test`, appeared in the log but were not identified as successfully authenticated accounts.

Recorded challenge answer:

    1:Sophia

### 4. Count Successful Password Authentications

Command:

    grep -i "Accepted password" sshlog.log | wc -l

Recorded result:

    2

The log contains two entries matching the successful password authentication pattern.

### 5. Identify the First Recorded Connection

Command:

    grep -i "Connection from" sshlog.log | head -10

The first displayed connection entry was:

    2021-04-29 23:52:25.989 Connection from 192.168.1.17 port 49338 on 192.168.1.20 port 22

This entry records a connection from `192.168.1.17` to `192.168.1.20` on TCP port `22`, the standard SSH port.

### 6. Determine the SSH Logging Verbosity

Command:

    grep -iE "log level|LogLevel|DEBUG|INFO|WARN|ERROR" sshlog.log | head -10

The inspected entries included debug-level messages such as `debug1`, `debug2`, and `debug3`.

The source notes record the challenge answer as:

    Debug 3


### 7. Investigate Windows OpenSSH Log Locations

The investigation included a separate research step to identify where Windows OpenSSH may store its SSH server logs.

The source notes record the following path:

    C:\ProgramData\ssh\logs\sshd.log

This path was identified through external research. It was not extracted or verified from the supplied `sshlog.log` evidence.

Windows OpenSSH can also send logging information to Windows Event Viewer. Whether a separate log file is created depends on the logging configuration.

### 8. Relevant SSH Log Patterns

| Log pattern | Meaning |
|---|---|
| `Failed password` | Failed password authentication attempt |
| `Accepted password` | Successful password authentication |
| `Connection from` | Incoming SSH connection |
| `debug1`, `debug2`, `debug3` | Debug-level logging messages |
| `does not exist` | Invalid or non-existent username indication |

## Key Findings

- `192.168.1.17` was identified in the source notes as the relevant internal attacker IP.
- The account `sophia` was identified as successfully authenticated.
- Two successful password authentication entries were recorded.
- The first displayed connection entry originated from `192.168.1.17` and targeted `192.168.1.20` on TCP port `22`.
- The source notes record the SSH logging verbosity answer as `Debug 3`.
- Windows OpenSSH logging locations were investigated separately from the supplied log evidence.

## Skills Developed

- Searching and filtering SSH logs with `grep`.
- Extracting fields with `awk`.
- Counting authentication events with `wc -l`.
- Identifying repeated IP addresses using `sort` and `uniq -c`.
- Recognising successful and failed SSH authentication patterns.
- Interpreting connection records and debug-level messages.
- Researching Windows OpenSSH logging behaviour.

## Conclusion

This challenge demonstrated a practical command-line approach to SSH log analysis. By filtering and counting relevant log entries, it was possible to identify recurring IP addresses, successful authentication activity, connection details, and logging verbosity.

The exercise reinforced the importance of distinguishing evidence directly obtained from log files from information established through external research.

## Disclaimer

This documentation describes a controlled educational challenge performed for defensive security training and BTL1 (Blue Team Level 1) exam preparation. Findings are based on the supplied challenge material and recorded investigation notes.

