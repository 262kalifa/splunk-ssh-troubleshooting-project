# Splunk SSH Troubleshooting & Log Forwarding Investigation

## Overview

This project documents a hands-on troubleshooting investigation involving SSH connectivity, Linux authentication logging, and Splunk Universal Forwarder communication in a local cybersecurity lab.

The investigation began with an SSH connectivity failure between Kali Linux and an Ubuntu Server. After restoring network communication, a second issue was identified: new authentication events were being generated locally on Ubuntu but were not reaching Splunk Enterprise.

The project demonstrates a structured troubleshooting process across the network, service, log, and SIEM layers rather than assuming a single root cause.

## Objectives

- Troubleshoot SSH connectivity between Kali Linux and Ubuntu Server.
- Verify that the SSH service and authentication logging were functioning correctly.
- Confirm that new authentication events were being written to `/var/log/auth.log`.
- Investigate why new events were not appearing in Splunk.
- Validate Splunk Universal Forwarder status and receiver connectivity.
- Correct the forwarding configuration and verify successful log ingestion.
- Document the investigation with commands, SPL queries, screenshots, and a technical report.

## Lab Environment

| Component | Role |
|---|---|
| Ubuntu Server | SSH server and Splunk Universal Forwarder host |
| Kali Linux | Authorized SSH testing system |
| Windows host | Splunk Enterprise receiver |
| Oracle VirtualBox | Virtualization platform |
| Splunk Enterprise | Centralized log search and analysis |
| Splunk Universal Forwarder | Forwards Ubuntu authentication logs to Splunk |

The virtual systems communicated over a VirtualBox host-only network.

## Data Source

The primary log source was:

```text
/var/log/auth.log
```

This log contains Linux authentication activity such as SSH login attempts, invalid users, failed passwords, successful authentication, and session events.

## Investigation Workflow

### 1. Verify the SSH service

The SSH service was checked first:

```bash
sudo systemctl status ssh
```

The service was active and listening on port 22. This ruled out a stopped SSH service as the immediate cause of the connection failure.

### 2. Investigate the SSH connectivity failure

An SSH attempt from Kali returned:

```text
Network is unreachable
```

This pointed the investigation toward network configuration rather than authentication.

The VirtualBox adapter configuration was reviewed and corrected so the lab systems could communicate over the host-only network.

### 3. Verify local authentication telemetry

After restoring connectivity, Ubuntu authentication activity was monitored in real time:

```bash
sudo tail -f /var/log/auth.log
```

The server recorded failed login attempts, invalid-user activity, password failures, and a successful SSH login. This confirmed that Ubuntu was generating the expected local telemetry.

### 4. Compare local log generation with Splunk ingestion

A Splunk event-count search was used to check whether new activity was arriving:

```spl
index=*
| stats count by host source sourcetype
```

The local authentication log continued to change, but the Splunk event count did not increase. This narrowed the issue to the forwarding path rather than SSH itself.

### 5. Verify the Splunk Universal Forwarder

The forwarder service was checked:

```bash
sudo /home/kalafi/splunkforwarder/bin/splunk status
```

The forwarder was running.

The configured receiver was then inspected:

```bash
sudo /home/kalafi/splunkforwarder/bin/splunk list forward-server
```

The result showed no active forward and an outdated receiver configuration:

```text
Active forwards:
None

Configured but inactive forwards:
10.225.156.46:9997
```

This established that the forwarder process itself was healthy, but it was not connected to the correct Splunk receiver.

### 6. Correct the receiver configuration

The Windows host-only address was identified as:

```text
192.168.56.1
```

The old receiver was removed and the correct receiver was configured:

```bash
sudo /home/kalafi/splunkforwarder/bin/splunk remove forward-server 10.225.156.46:9997
sudo /home/kalafi/splunkforwarder/bin/splunk add forward-server 192.168.56.1:9997
sudo /home/kalafi/splunkforwarder/bin/splunk restart
```

The forward-server status then showed an active connection to:

```text
192.168.56.1:9997
```

### 7. Validate receiver-port connectivity

Connectivity to the Splunk receiving port was tested from Ubuntu:

```bash
nc -vz 192.168.56.1 9997
```

The connection succeeded, confirming that Ubuntu could reach the Splunk receiver on TCP port 9997.

### 8. Verify successful ingestion

New SSH activity was generated and Splunk was queried again:

```spl
index=* host=lifa-servers source="/var/log/auth.log"
| stats count by host source sourcetype
```

The event count increased for:

```text
host=lifa-servers
source=/var/log/auth.log
sourcetype=auth
```

This verified that new Ubuntu authentication events were successfully reaching Splunk after the forwarding configuration was corrected.

## Useful SPL Queries

### Review indexed hosts, sources, and sourcetypes

```spl
index=*
| stats count by host source sourcetype
| sort host source sourcetype
```

### Review Ubuntu authentication events

```spl
index=* host=lifa-servers source="/var/log/auth.log"
| stats count by host source sourcetype
| sort - count
```

### View SSH-related authentication logs

```spl
index=* host=lifa-servers source="/var/log/auth.log" sourcetype=auth sshd
| table _time host source sourcetype _raw
| sort - _time
```

### Extract source IPs from SSH authentication events

```spl
index=* host=lifa-servers source="/var/log/auth.log" sourcetype=auth ("Failed password" OR "Accepted password" OR "Invalid user")
| rex field=_raw "from (?<src_ip>\\d+\\.\\d+\\.\\d+\\.\\d+)"
| table _time host src_ip sourcetype _raw
| sort - _time
```

## Key Findings

- The SSH service was healthy; the initial connection problem was caused by network reachability.
- Ubuntu continued to generate authentication events locally after SSH connectivity was restored.
- The Splunk Universal Forwarder process was running, but its configured receiver was outdated and inactive.
- Correcting the receiver address and restarting the forwarder restored event forwarding.
- Successful port testing and increased event counts in Splunk provided independent validation that the forwarding path was working.

## Skills Demonstrated

- Linux service troubleshooting
- SSH connectivity investigation
- Linux authentication log analysis
- Real-time log monitoring
- Splunk Universal Forwarder administration
- TCP port validation
- SPL searching and field extraction
- Layered troubleshooting methodology
- Evidence-based technical documentation

## Troubleshooting Approach

A major lesson from this project was to validate each layer independently:

```text
Network → Service → Local Logs → Forwarder → Receiver Port → SIEM Ingestion
```

A service can be healthy while the network path is broken, and a forwarder can be running while still pointing to the wrong receiver. Testing each layer separately made it possible to isolate both problems without assuming that one failure explained everything.

## Project Report

A detailed report is included in the repository:

- [Splunk SSH Troubleshooting Project Report](./Splunk_SSH_Troubleshooting_Project_Report.pdf)

## Evidence

The repository includes screenshots documenting the troubleshooting process, including:

- SSH service state
- network reachability failure
- VirtualBox adapter configuration
- live `auth.log` activity
- Splunk event counts before and after remediation
- Universal Forwarder status
- inactive and corrected forward-server configuration
- TCP 9997 connectivity validation
- final Splunk verification

## Status

**Completed.** The SSH connectivity and Splunk log-forwarding issues were identified, remediated, and independently validated.
