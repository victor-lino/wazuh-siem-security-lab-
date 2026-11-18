
# Wazuh SIEM Lab – Architecture

This directory contains the logical architecture diagram of the Wazuh SIEM laboratory.

## Environment Overview

The lab was built using VMware and includes the following components:

- **SEC01:** Ubuntu Server running Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard.
- **DC01:** Windows Server domain controller running the Wazuh agent.
- **WIN-TARGET01:** Windows endpoint running the Wazuh agent.
- **Kali Linux:** Security testing machine used to generate controlled activity.
- **SECURITYLAB:** Isolated virtual network (`192.168.250.0/24`).

## Monitoring Architecture

Windows endpoints send security telemetry through their Wazuh agents to the Wazuh Manager.

The Manager processes incoming events and forwards the data to the Indexer, where it becomes available for investigation through the Dashboard.

Kali Linux is used to perform controlled security tests against the target environment.

## Diagram

The `architecture.mmd` file contains the Mermaid source code for the logical architecture diagram.

You can visualize it using the [Mermaid Live Editor](https://mermaid.live/).

> Note: This diagram represents the logical monitoring architecture and does not claim to document every VMware virtual network interface, firewall rule, or physical network route.
