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
