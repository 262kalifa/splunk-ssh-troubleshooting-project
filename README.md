# -splunk-ssh-troubleshooting-project
Description: A beginner SOC project focused on troubleshooting SSH connectivity and Splunk Universal Forwarder log forwarding issues using Linux logs and Splunk.
# Troubleshooting SSH Service Failure Using Linux Logs and Splunk

## Project Overview

This project documents the troubleshooting of SSH connectivity and Splunk log forwarding issues in a local cybersecurity lab environment.

The lab involved an Ubuntu Server VM, Kali Linux, a Windows host machine, Splunk Enterprise, and Splunk Universal Forwarder. The goal was to investigate why SSH was not working properly and why newly generated SSH authentication logs were not appearing in Splunk.

## Project Objective
The objective of this project was to:
- Troubleshoot SSH connectivity issues between Kali Linux and Ubuntu Server.
- Verify that the SSH service was running on Ubuntu.
- Monitor live SSH authentication logs using Linux commands.
- Investigate why new SSH logs were not appearing in Splunk.
- Check the Splunk Universal Forwarder status.
- Identify and fix an inactive Splunk forward-server configuration.
- Confirm that new SSH logs were successfully forwarded to Splunk.

## Lab Environment

- Ubuntu Server VM
- Kali Linux VM
- Windows host machine
- VirtualBox
- Splunk Enterprise installed on Windows
- Splunk Universal Forwarder installed on Ubuntu Server

## Tools Used

- Linux terminal
- SSH
- `systemctl`
- `tail`
- `ipconfig`
- `nc`
- Splunk Universal Forwarder
- Splunk Enterprise
- Splunk Search Processing Language

## Data Source

The main log source used in this project was:

```text
/var/log/auth.log

This log file contains Linux authentication events, including SSH login attempts, failed password attempts, invalid user attempts, successful logins, and session activity.

Troubleshooting Process
1. SSH Service Status Check

The SSH service was checked using:

sudo systemctl status ssh

The result showed that the SSH service was active and running. The server was also listening on port 22. This confirmed that the SSH problem was not caused by the SSH service being stopped.

2. SSH Connection Failure

An SSH connection attempt from Kali to the Ubuntu server returned:

Network is unreachable

This indicated that the issue was related to network connectivity or routing rather than SSH authentication.

3. Network Adapter Review

The Ubuntu Server VM was shut down so that the VirtualBox network adapter settings could be reviewed. The VM was configured to use a Host-only Adapter.

This was important because Kali, Ubuntu, and the Windows host needed to communicate on the same virtual network.

4. Live Authentication Log Monitoring

After SSH connectivity was restored, live SSH authentication logs were monitored using:

sudo tail -f /var/log/auth.log

The logs showed failed login attempts for an invalid user, repeated password failures, and a successful SSH login for user kalafi.

This confirmed that Ubuntu was generating SSH authentication logs locally.

5. Splunk Log Count Issue

After generating new SSH activity, the following Splunk query was used:

index=*
| stats count by host source sourcetype

The result showed that the event counts did not increase after new logs were created. This indicated that Ubuntu was generating logs locally, but Splunk was not receiving the newly created authentication events.

6. Splunk Universal Forwarder Status Check

The Splunk Universal Forwarder was checked using:

sudo /home/kalafi/splunkforwarder/bin/splunk status

The result showed that the forwarder was running. This confirmed that the problem was not caused by the forwarder service being stopped.

7. Forward-Server Status Check

The configured forward-server was checked using:

sudo /home/kalafi/splunkforwarder/bin/splunk list forward-server

The result showed:

Active forwards:
None

Configured but inactive forwards:
10.225.156.46:9997

This showed that the Splunk Universal Forwarder was running but not actively connected to Splunk Enterprise. The forwarder was still configured to send logs to an old receiver IP address.

8. Correct Splunk Receiver IP Identified

The Windows host IP address was checked using:

ipconfig

The correct Windows Host-only IP address was identified as:

192.168.56.1

Because the Ubuntu server was using the 192.168.56.x Host-only network, the correct Splunk receiver address was:

192.168.56.1:9997
9. Forward-Server Configuration Fixed

The old inactive forward-server was removed using:

sudo /home/kalafi/splunkforwarder/bin/splunk remove forward-server 10.225.156.46:9997

The correct Splunk receiver address was added using:

sudo /home/kalafi/splunkforwarder/bin/splunk add forward-server 192.168.56.1:9997

The Splunk Universal Forwarder was restarted using:

sudo /home/kalafi/splunkforwarder/bin/splunk restart

After restarting, the forward-server list showed an active forward to:

192.168.56.1:9997
10. Port 9997 Connectivity Test

Connectivity to the Splunk receiving port was tested using:

nc -vz 192.168.56.1 9997

The connection succeeded. This confirmed that Ubuntu could reach Splunk Enterprise on port 9997.

11. New Logs Confirmed in Splunk

After generating new SSH activity, the following Splunk query was used:

index=* host=lifa-servers source="/var/log/auth.log"
| stats count by host source sourcetype

The event count increased under:

host=lifa-servers
source=/var/log/auth.log
sourcetype=auth

This confirmed that new SSH authentication logs were successfully forwarded from Ubuntu to Splunk after the forward-server configuration was corrected.

Key SPL Queries
Check Hosts, Sources, and Sourcetypes
index=*
| stats count by host source sourcetype
| sort host source sourcetype
Check Authentication Logs from Ubuntu

index=* host=lifa-servers source="/var/log/auth.log"
| stats count by host source sourcetype
| sort - count
View SSH-Related Authentication Logs

index=* host=lifa-servers source="/var/log/auth.log" sourcetype=auth sshd
| table _time host source sourcetype _raw
| sort - _time
Search Failed and Successful SSH Login Events

index=* host=lifa-servers source="/var/log/auth.log" sourcetype=auth ("Failed password" OR "Accepted password" OR "Invalid user")
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip sourcetype _raw
| sort - _time

Screenshots
The Images folder contains screenshots showing:

SSH service status
SSH network unreachable error
VM network adapter setting
Live SSH authentication logs
Splunk log count before the fix
Splunk Universal Forwarder status
Inactive forward-server configuration
Windows Host-only IP address
Updated active forward-server
Successful port 9997 connectivity test
Splunk log count after the fix
Main Findings

The project showed that the SSH service itself was running correctly. The initial SSH issue was related to network connectivity.

After SSH activity was restored, a second issue was identified: Splunk was not receiving new authentication logs. The Splunk Universal Forwarder was running, but it was still configured with an old inactive Splunk receiver IP address.

The issue was fixed by identifying the correct Windows Host-only IP address, updating the forward-server to 192.168.56.1:9997, restarting the Splunk Universal Forwarder, and confirming that new logs were received in Splunk under sourcetype auth.

Skills Demonstrated

This project demonstrates beginner SOC and troubleshooting skills, including:

Linux service troubleshooting
SSH connectivity investigation
Linux authentication log analysis
Real-time log monitoring
Splunk Universal Forwarder troubleshooting
Network port testing
SPL searching and validation
Evidence-based documentation
Conclusion

This project demonstrated how Linux logs and Splunk can be used together to troubleshoot SSH and log forwarding problems.

The investigation showed that a service can be running locally but still fail because of network or forwarding configuration issues. By checking the SSH service, reviewing live authentication logs, testing network connectivity, and correcting the Splunk forward-server configuration, new SSH authentication logs were successfully forwarded to Splunk.
