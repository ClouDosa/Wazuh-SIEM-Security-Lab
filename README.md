# Wazuh SIEM Deployment and Detection Engineering Lab

## Overview

This project documents the deployment and validation of a Wazuh security monitoring environment built in a virtual lab. I installed and configured the Wazuh server components, connected Linux and Windows agents, troubleshot deployment issues, generated security events, and created a custom decoder and rule for a network device log.

The goal was not only to get Wazuh running, but to validate the complete path from endpoint activity to detection and alert generation.

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

![Ubuntu server](screenshots/01%20environment/logged%20into%20ubuntu%20server.webp)

### 2. Wazuh Server Deployment

I configured the Wazuh package repository, installed the Wazuh Indexer, Manager, Dashboard, and supporting services, and validated that the main Wazuh processes were running.

![Repository setup](screenshots/02%20wazuh%20deployment/task1%201%20ssh%20gpgkey%20repo%20setup.webp)

![Wazuh manager services](screenshots/02%20wazuh%20deployment/task1%201%20manager%20daemons%20status.webp)

I also confirmed access to the Wazuh Dashboard over HTTPS.

![Wazuh dashboard](screenshots/02%20wazuh%20deployment/task1%201%20dashboard%20live%20with%20port443%20config.webp)

![Dashboard overview](screenshots/02%20wazuh%20deployment/task1%201%20dashboard%20overview.webp)

### 3. Troubleshooting and Recovery

During the deployment I encountered storage and service issues instead of restarting the build from scratch. I investigated disk usage, identified the storage problem, extended the logical volume, and validated the additional space.

![Disk full troubleshooting](screenshots/03%20troubleshooting/task1%201%20disc%20full%20error.webp)

![LVM extension](screenshots/03%20troubleshooting/task1%201%20lvm%20extend.webp)

I also worked through a Filebeat certificate and service issue and checked system resources, shards, permissions, and Wazuh data paths while troubleshooting.

![Filebeat troubleshooting](screenshots/03%20troubleshooting/task1%201%20filebeat%20cert%20troubleshoot.webp)

### 4. Linux and Windows Agent Enrollment

I installed and started Wazuh agents, configured the required lab networking and port forwarding, and troubleshot an initial enrollment connection failure.

![Enrollment connection troubleshooting](screenshots/04%20agent%20enrollment/task1%202%20cant%20connect%20to%20enrollment%20service.webp)

After correcting the connection path, the agent registered successfully through `wazuh-authd`.

![Successful agent registration](screenshots/04%20agent%20enrollment/task1%202%20successful%20registration%20via%20wazuh%20authd.webp)

I validated the Windows agent service and confirmed that the enrolled agents appeared as active in Wazuh.

![Windows Wazuh agent](screenshots/04%20agent%20enrollment/task1%202%20windows%20agent%20service%20running.webp)

![Active agents](screenshots/04%20agent%20enrollment/task1%202%20all%20dashboard%20agents%20wazuh%20active.webp)

### 5. Detection Validation with Failed SSH Authentication

To verify that Wazuh was detecting endpoint security activity, I generated repeated failed SSH authentication attempts in the lab.

![Failed SSH attempts](screenshots/05%20detection%20validation/task1%203%20creating%20fake%20entries.webp)

I then confirmed that the Wazuh alerts file recorded the activity and triggered rule **5712** for the SSH authentication failures.

![Wazuh brute force alert](screenshots/05%20detection%20validation/task1%203%20bruteforce%20alert%20rule5712.webp)

### 6. Custom Decoder and Detection Rule

I created a custom Wazuh decoder for a structured network device log and tested it with `wazuh-logtest`. The first decoder attempt produced a regex and configuration error, so I corrected the decoder and tested it again until the expected fields were parsed successfully.

The decoded fields included values such as:

- `nas_ip`
- `mac_addr`
- `user_name`
- `domain`
- `host_resolved_identities`
- `networkDeviceProfileName`
- `netBiosName`
- `portID`
- `state`

![Custom decoder validation](screenshots/06%20custom%20decoder%20rule/task2%20decoder%20logtest%20success.webp)

I then created a custom rule that matched the decoded event and validated it with `wazuh-logtest`. The final test generated custom rule ID **100011**, level **8**, for the unexpected host resolved identity condition.

![Custom Wazuh rule](screenshots/06%20custom%20decoder%20rule/task2%20rules%20logtest%20success.webp)

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
Wazuh SIEM Security Lab/
├── README.md
└── screenshots/
    ├── 01 environment/
    ├── 02 wazuh deployment/
    ├── 03 troubleshooting/
    ├── 04 agent enrollment/
    ├── 05 detection validation/
    └── 06 custom decoder rule/
```

## Security Note

Authentication material and Wazuh agent keys are not published in this repository. Screenshots are included only where they demonstrate configuration, troubleshooting, enrollment status, or detection results without intentionally exposing reusable credentials.

## Future Improvements

- Add sanitized copies of the custom decoder XML and local rule XML
- Add an architecture diagram of the Wazuh server and monitored endpoints
- Automate portions of the deployment using Ansible
- Add additional detections mapped to MITRE ATT&CK techniques
