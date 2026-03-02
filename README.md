# Virtual SOC Lab
## Suricata IDS Integrated with Elastic SIEM Stack

## Project Summary
This project demonstrates the design, deployment, and validation of a Virtual Security Operations Centre (SOC) lab using open-source security tools.

The lab objective was to:
- Deploy a Network Intrusion Detection System (Suricata)
- Integrate it with a SIEM platform (Elastic Stack)
- Simulate reconnaissance attacks
- Detect, forward, and visualize security alerts
- Validate an end-to-end threat detection workflow

The lab successfully detects reconnaissance scans (for example, Nmap SYN scans) and visualizes alerts in Kibana.

## Objectives
- Configure a secure virtual network environment
- Deploy Suricata as a network IDS
- Enable and manage threat detection rules
- Simulate attack traffic from an attacker VM
- Forward IDS logs to Elasticsearch using Filebeat
- Visualize and analyze alerts in Kibana
- Validate real-time detection capability

## Lab Architecture
### Architecture Overview
1. Attacker VM generates malicious or test network traffic.
2. Suricata IDS server monitors traffic and writes alerts/logs.
3. Filebeat collects Suricata logs and ships them to Elasticsearch.
4. Elasticsearch indexes and stores security events.
5. Kibana visualizes events for investigation and monitoring.

![Lab architecture](docs/doc-images/image1.png)

## Components
### Attacker Machine
- OS: Ubuntu 24.04
- Tool: Nmap
- Role: Simulate reconnaissance attacks

### IDS Server
- OS: Ubuntu 24.04
- Suricata
- Elasticsearch 8.x
- Kibana 8.x
- Filebeat

## Environment Setup
### Virtualization Platform
- Oracle VirtualBox
- Host-Only Networking mode
- Network range: `192.168.56.0/24`

### Machine Configuration
#### IDS Server
- RAM: 15 GB
- OS: Ubuntu 24.04 LTS
- Monitored interface: `enp0s8`

![IDS server setup](docs/doc-images/image2.png)

#### Attacker VM
- RAM: 10 GB
- OS: Ubuntu 24.04 LTS

![Attacker VM setup](docs/doc-images/image3.png)

## Network Configuration
### Adapter Configuration
- Adapter 1: Host-Only Adapter (Promiscuous Mode enabled)
- Adapter 2: NAT (internet access for package downloads)

### Assigned IP Addresses
- IDS Server: `192.168.56.103`
- Attacker VM: `192.168.56.102`

### Connectivity Verification
```bash
ping 192.168.56.103
tcpdump -i enp0s8
```

## Suricata Deployment
### Installation
```bash
sudo apt update
sudo apt install suricata -y
```

### Rule Management
Enabled Emerging Threats Open ruleset:
```bash
sudo suricata-update enable-source et/open
sudo suricata-update update-sources
sudo suricata-update
```

Loaded rules: ~48,000+ detection rules.

### Interface Configuration
Configured Suricata to monitor `enp0s8`.

Verification:
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -i enp0s8
```

![Suricata config test](docs/doc-images/image4.png)

### Service Validation
```bash
sudo systemctl status suricata
```

![Suricata service status](docs/doc-images/image5.png)

Confirmed active and running.

## Attack Simulation
Attack traffic generated from attacker VM:
```bash
nmap -sS 192.168.56.103
nmap -A 192.168.56.103
```

![Nmap attack simulation](docs/doc-images/image6.png)

### Observed Detection
- ET SCAN NMAP OS Detection Probe
- TCP SYN scan patterns
- Reconnaissance alerts

Verified alerts in:
- `/var/log/suricata/fast.log`
- `/var/log/suricata/eve.json`

![Suricata alert logs](docs/doc-images/image7.png)

## Elastic Stack Deployment
### Elasticsearch
Configured as a single-node cluster:
```yaml
discovery.type: single-node
```

Verified cluster status:
```bash
curl -k -u elastic:<password> https://localhost:9200
```

![Elasticsearch status](docs/doc-images/image8.png)

Cluster returned a healthy response.

### Kibana
Connected to Elasticsearch using an enrollment token.

![Kibana enrollment](docs/doc-images/image9.png)

Verified access via:
`http://192.168.56.103:5601`

Created data view:
`filebeat-*`

![Kibana data view](docs/doc-images/image10.png)
![Kibana events](docs/doc-images/image11.png)

## Filebeat Configuration
### Module Activation
```bash
sudo filebeat modules enable suricata
```

Enabled eve fileset:
```yaml
enabled: true
var.paths: ["/var/log/suricata/eve.json"]
```

### Setup
```bash
sudo filebeat setup
```

### Service Start
```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

![Filebeat service](docs/doc-images/image12.png)

Verified ingestion:
```bash
sudo filebeat test output
```

![Filebeat output test](docs/doc-images/image13.png)

## Log Flow Validation
Confirmed full pipeline:
1. Nmap scan executed
2. Suricata generated alert
3. `eve.json` updated
4. Filebeat forwarded log
5. Elasticsearch indexed event
6. Kibana displayed detection

![Pipeline validation 1](docs/doc-images/image14.png)
![Pipeline validation 2](docs/doc-images/image15.png)
![Pipeline validation 3](docs/doc-images/image16.png)

## Detection Analysis
Example alert:
- ET SCAN NMAP OS Detection Probe

Mapped to MITRE ATT&CK:
- T1595 - Active Scanning
- T1046 - Network Service Scanning

- Severity: Medium
- Classification: Attempted Information Leak

![Detection analysis 1](docs/doc-images/image17.png)
![Detection analysis 2](docs/doc-images/image18.png)

## Troubleshooting Performed
Issues resolved during setup:
- Suricata interface mismatch
- Missing ruleset loading
- Elasticsearch bootstrap error
- Kibana authentication issue
- Filebeat module misconfiguration (eve fileset disabled)

Diagnosis commands used:
```bash
systemctl status <service>
journalctl -u <service>
```

## Skills Demonstrated
- Network configuration
- IDS deployment and rule management
- SIEM integration
- Log pipeline troubleshooting
- Threat detection validation
- Elastic security architecture

## Outcome
Successfully implemented a working SOC lab capable of:
- Detecting reconnaissance attacks
- Centralizing IDS logs
- Visualizing security events
- Performing initial threat investigation

This lab simulates real-world SOC infrastructure used in enterprise environments.

## Future Enhancements
- Add Zeek for network analysis
- Add Wazuh for host-based detection
- Create automated detection rules in Kibana
- Build custom dashboards
- Implement alert email notifications
- Deploy using Docker
