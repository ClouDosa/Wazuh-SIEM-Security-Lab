# Wazuh SIEM Deployment and Detection Engineering Lab

## Overview

This project documents the deployment and validation of a Wazuh security monitoring environment built in a virtual lab. I installed and configured the Wazuh server components, connected Linux and Windows agents, troubleshot deployment issues, generated security events, and created a custom decoder and rule for a network device log.

The goal was not only to get Wazuh running, but to validate the complete path from endpoint activity to detection and alert generation.

## Architecture

The lab uses an Ubuntu Server virtual machine to host the Wazuh server components, including the Wazuh Manager, Indexer, Dashboard, and Filebeat. Linux and Windows endpoints run Wazuh agents and send security data back to the server for analysis.

The project validates two detection paths. The first uses endpoint activity, where repeated failed SSH authentication attempts are collected and matched to an existing Wazuh detection rule. The second uses a custom network device log, where a decoder extracts structured fields and local rules evaluate those fields to generate a custom alert.

```text
Linux / Windows endpoints
        |
        v
   Wazuh agents
        |
        v
   Wazuh Manager
        |
        +----> Built in detection rules
        |
        +----> Custom decoder and local rules
        |
        v
 Wazuh Indexer / Dashboard
```

## Lab Environment

- Oracle VirtualBox
- Ubuntu Server
- Wazuh 4.14.7
- Wazuh Indexer
- Wazuh Manager
- Wazuh Dashboard
- Filebeat
- Linux Wazuh agent
- Windows Wazuh agent
- SSH test traffic

> The screenshots use a private lab environment. Sensitive agent key material is intentionally excluded from this public repository.

## What I Built

### 1. Virtual Lab and Ubuntu Server

I created the virtualized environment in VirtualBox, installed Ubuntu Server, and verified the server was ready for the Wazuh deployment.

![Ubuntu server](Wazuh%20Technical%20Test/Logged%20into%20Ubuntu%20Server.png)

### 2. Wazuh Server Deployment

I configured the Wazuh package repository, installed the Wazuh Indexer, Manager, Dashboard, and supporting services, and validated that the main Wazuh processes were running.

![Wazuh manager services](Wazuh%20Technical%20Test/task1.1_manager-daemons-status.png)

I also confirmed access to the Wazuh Dashboard over HTTPS.

![Wazuh dashboard](Wazuh%20Technical%20Test/task1.1_dashboard-live-with-port443-config.png)

![Dashboard overview](Wazuh%20Technical%20Test/task1.1_dashboard-overview.png)

### 3. Troubleshooting and Recovery

During the deployment I encountered storage and service issues instead of restarting the build from scratch. I investigated disk usage, identified the storage problem, extended the logical volume, and validated the additional space.

![Disk full troubleshooting](Wazuh%20Technical%20Test/task1.1_disc-full-error.png)

![LVM extension](Wazuh%20Technical%20Test/task1.1_lvm-extend.png)

I also worked through a Filebeat certificate and service issue and checked system resources, shards, permissions, and Wazuh data paths while troubleshooting.

![Filebeat troubleshooting](Wazuh%20Technical%20Test/task1.1_filebeat-cert-troubleshoot.png)

### 4. Linux and Windows Agent Enrollment

I installed and started Wazuh agents, configured the required lab networking and port forwarding, and troubleshot an initial enrollment connection failure.

![Enrollment connection troubleshooting](Wazuh%20Technical%20Test/task1.2_cant%20connect%20to%20enrollment%20service.png)

After correcting the connection path, the agent registered successfully through `wazuh-authd`.

![Successful agent registration](Wazuh%20Technical%20Test/task1.2_successful-registration-via-wazuh-authd.png)

I validated the Windows agent service and confirmed that the enrolled agents appeared as active in Wazuh.

![Windows Wazuh agent](Wazuh%20Technical%20Test/task1.2_windows-agent-service-running.png)

![Active agents](Wazuh%20Technical%20Test/task1.2_all-dashboard-agents-wazuh-active.png)

### 5. Detection Validation with Failed SSH Authentication

To verify that Wazuh was detecting endpoint security activity, I generated repeated failed SSH authentication attempts in the lab.

![Failed SSH attempts](Wazuh%20Technical%20Test/task1.3_creating-fake-entries.png)

I then confirmed that the Wazuh alerts file recorded the activity and triggered rule **5712** for the SSH authentication failures.

![Wazuh brute force alert](Wazuh%20Technical%20Test/task1.3_bruteforce-alert-rule5712.png)

### 6. Custom Decoder and Detection Rule

I created a custom Wazuh decoder for a structured network device log and tested it with `wazuh-logtest`. The first decoder attempt produced a regex and configuration error, so I corrected the decoder and tested it again until the expected fields were parsed successfully.

The decoder identifies logs from `SF-IMS` and extracts the following fields:

- `state`
- `nas_ip`
- `mac_addr`
- `user_name`
- `domain`
- `host_resolved_identities`
- `networkDeviceProfileName`
- `netBiosName`
- `portID`

![Custom decoder validation](Wazuh%20Technical%20Test/task2_decoder_logtest_success.png)

I then created two local rules. Rule **100010** generates a level 5 event when the decoded domain matches the sanitized test value. Rule **100011** generates a level 8 event when the network device profile is `test` and the resolved identity does not match the expected sanitized value.

![Custom Wazuh rule](Wazuh%20Technical%20Test/task2_rules_logtest_success.png)

## Detection Engineering Files

The sanitized detection files used in the lab are included so the project can be reviewed beyond screenshots:

- [Custom network decoder](decoders/network_decoders.xml)
- [Custom local rules](rules/local_rules.xml)

The decoder converts the structured network event into named fields. The local rules evaluate those fields and generate the custom detections demonstrated in the screenshots.

## Validation Approach

I validated the lab at each stage instead of treating installation success as the final result. Server processes and dashboard access were checked after deployment. Agent status was verified after enrollment. Failed SSH activity was intentionally generated and traced to the resulting Wazuh alert. The custom decoder and rules were tested with `wazuh-logtest` until the expected fields and rule IDs were returned.

This gave me evidence that the complete monitoring path was working: endpoint or log activity was collected, parsed, evaluated, and turned into a visible security detection.

## Troubleshooting Performed

This lab included several issues that required investigation rather than a clean one pass installation:

- Storage exhaustion during the Wazuh deployment
- LVM storage extension and filesystem growth
- Filebeat certificate and service troubleshooting
- Wazuh Indexer and shard validation
- Permissions and `/var/ossec` path checks
- Agent enrollment connectivity troubleshooting
- VirtualBox port forwarding configuration
- Custom decoder regex and configuration correction
- Custom rule validation with `wazuh-logtest`

## Security Skills Demonstrated

- SIEM deployment and administration
- Linux server administration
- Wazuh architecture and services
- Endpoint agent deployment
- Log collection and analysis
- Security alert validation
- SSH authentication monitoring
- Custom decoder development
- Custom detection rule development
- Troubleshooting Linux storage, services, permissions, and certificates
- Virtualized network configuration

## Repository Structure

```text
Wazuh-SIEM-Security-Lab/
├── README.md
├── SCREENSHOT MAP.md
├── decoders/
│   └── network_decoders.xml
├── rules/
│   └── local_rules.xml
└── Wazuh Technical Test/
    └── PNG lab screenshots
```

## Project Evidence

The `Wazuh Technical Test` folder contains the original lab screenshots used throughout this README. `SCREENSHOT MAP.md` provides an additional reference for the evidence set, while the `decoders` and `rules` folders contain the sanitized detection engineering files.

## Security Note

Authentication material and Wazuh agent keys are not published in this repository. Screenshots are included only where they demonstrate configuration, troubleshooting, enrollment status, or detection results without intentionally exposing reusable credentials.

## Future Improvements

- Add a visual architecture diagram of the Wazuh server and monitored endpoints
- Automate portions of the deployment using Ansible
- Add additional detections mapped to MITRE ATT&CK techniques
