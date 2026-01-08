## CCNA week 3 notes

- [Understanding Cisco Data Center Foundations | DCFNDU]( #dcfndu )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )
- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )
- [CCNA 200-301]( #ccna )

# DCFNDU

The end of this course is in sight and the last topic included the Cisco Application Centric Infrastructure which introduced another package of tools for automating networks and integrations with cloud infrastructure. The technology utilizes spine and leaf architecture to be easily scaled and monitored using APICs (Application Policy Infrastructure Controllers). Other teminology involved included policy models, EPG and contracts, VMM (virtual machine manager), and service graphs. This was a good resource for finding out about a lot of the technologies I can get involved with once I am further along in my career goal - I will focus more now on CCNA and passing the CCNA exam. I have Wendell Odom's textbook here now so I will be posting about it for the next few weeks.

# CMLLAB

Discussions include the cmlutils library in Python which allows for command-line interaction with CML labs and nodes in the topology. I also practiced developing a CI/CD pipeline in gitlab with NSO container to automate testing. That I will need to revisit later since while the initial config pipeline was successful, subsequent pushes broke something.

![build pipeline with gitlab for automated testing over CML](/assets/img/ccna3/NSO%20-%20gitlab%20-%20pipeline.png)
![scripts for gitlab ci/cd](/assets/img/ccna3/NSO%20-%20term.png)

# DEVNAE

The course continues with describing Ansible, an open source tool written in Python to orchestrate changes across networks. The YANG (yet another next generation) modelling language is also discussed. Related topics include deeper dives of NETCONF, RESTCONF, gNMI/gRPC when in use for network monitoring and the YANG suite.

Ansible:
- Utilizes yaml encoded files to organize assets and to configure those assets and to gather data
- Stores commands in a "playbook" file and devices in an "inventory" file and runs on the cli with "ansible-playbook 'inventory file' 'playbook-file'"

YANG:
- A model for encoding the components of a network and communicating with those components - alternative to CLI 
- Foundation for NETCONF, RESTCONF, gNMI and gRPC and uses xml encoding
- YANG types vary accross vendors - Cisco has a YANG for EIGRP for instance and others include IETF and OpenConfig from google
- Tools for dealing with YANG include validators, parsers, translators, editors, and generators
- Model driven telemetry (MDT) a more efficient mode of monitoring networks via streaming vs tradition methods via SNMP
- With python Jinja2 style templates, xml can be transmitted to numerous devices automatically

![router bgp neighbors with netconf and python](/assets/img/ccna3/bgp%20-%20netconf%20-%20python.png)

![gathering xml data with YANG suite](/assets/img/ccna3/YANG%20-%20gather%20xml%20interfaces%20data.png)
![YANG sutie](/assets/img/ccna3/YANG%20-%20suite.png)

# CCNA

This section will be devoted to Wendell Odom's CCNA guide and will be my main resource to help me prep for the CCNA exam. The book has 2 volumes, 8 sections as well as a companion youtube playlist, so there is lot to get through. I will be posting "mind maps" which Wendell is advocating for memory retrieval and other notes from the books.

The first five chapter topics follow: TCP/IP, LANs, IP Routing, CLI, and Switches.

```yaml
TCP/IP:
  Network:
    - USPS
    - Highways
  Model:
    vs OSI:
      - RFC ####
    Encapsulation:
      - packets
      - frames
      - segments
    Layers/abstractions:
      - physical // ethernet, 802.11
      - data-link // ethernet, 802.11
      - network // ICMP, IP
      - transport // TCP, UDP
      - application // HTTPS, SMTP
```

```yaml
LANs:
  - Connected devices / broadcast domain
  - layer 2, 3
    addressing:
      - IP
      - MAC
  - VS Hub
  Ethernet:
    - VS Fibre
    - UTP
    - RJ45 port / chassis
    - Duplex
  VS WAN:
    - cabling (e-line, leased line)
    - serial (coax)
```

```yaml
IP Routing:
  Addresses:
    - v4
    - v6
  Local / Remote:
    Subnets:
      Discovery:
        frames:
          - de/encapsulation
          - Unicast
          - Multicast
          - Broadcast
        - next hop
        - MAC address
        - tables
        - flooding    
```

```yaml
CLI:
  OS:
    - Linux
    - Windows
    - iOS
    - Red Hat
  enable:
    - exec / priveledged mode
  configure:
    show:
      - tab completion
      - ? 
      - <CR>
  - feature
```

```yaml
Switch:
  - FCS
  VLAN:
    Trunking:
      - Segmentation
  Layer 2:
    Learning:
      - MAC Address Table
  STP:
    - Loop prevention
  Frame Flooding:
    - Learning
    - FFFF.FFFF.FFFF
    - known address
    - unknown address
  Ports:
    - shut
    - no shut
```
