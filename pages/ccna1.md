# CCNA notes week 1
- [Understanding Cisco Data Center Foundations | DCFNDU]( #dcfndu )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )
- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )

### DCFNDU

This course will forms the backbone of knowledge about network engineering. It is all about hardware/software functionality: data center architecture, servers, switches, routers, storage, supervisors, access redundancy, virtualizaion, fiber, NX OS. Where they operate in the data center: what OSI layer each functions at, what protocols are used for network discovery and traffic management. Also what types of model architectures used.

Architectures:
- know 2-tier, 3-tier architectures
- SAN systems vs Cisco unified data center
- hyperconverged architecture

Switches:
- for the aggregation level (layer 2,3)
- upstream from routers (access layer) downstream from servers (core layer)
- port channels vs vPCs
- protocols for access redundancy/failover: HSRP, VRRP, GLBP

Virtualization:
- Switching (vSwitch): data plane, control plane, management, CoPP, fault recovery, virtual overlay, 
- Routing (vRF): tables (RIB/FIB), stacking interfaces (like vLAN), context aware services
- virtual machines: VMware, hypervisor, vMotion, High Availablity, fault tolerance, DRS, DPM

![vPC lab](/assets/img/ccna1/vpc_configs.png)

### CMLLAB

This course familiarizes with the Cisco Modeling Lab. A web app designed to mimick a datacenter with customizable topologies and virtual routers, switches, & servers to play with. Concepts include the CML architecture, simulation of devices (nodes) and images (OS, software), and network testing.

Architecture:
- Virtualized network controlled by CML API
- Web app UI to simulate workstations

Simulator:
- Adds nodes, links between them to represent network infrastructure
- Images uploaded to nodes can run cisco OS and software to represent real running network

![cml workstation](/assets/img/ccna1/cml_workstation.png)

### DEVNAE

This course introduces the concept of automating data center controls and operations via API and data models. Important for understanding the common problems in development pipelines and solutions to enhance teamwork across the developer and operations teams.