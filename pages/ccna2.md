## CCNA week 2 notes

- [Understanding Cisco Data Center Foundations | DCFNDU]( #dcfndu )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )
- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )

# DCFNDU

As I have plowed through the first week of the DCFNDU course I have noticed these topics aren't aligned with a lot of the CCNA topics. That is because this is more for CCNP, but no matter, I am completing this since it is free for this week. Continuing on the import topics are now data center storage, fiber channels, Cisco unified computing, and Cisco advanced services.

Data Center Storage:
- NAS: NFS, CIFS, runs over TCP/IP
- Block-Based storage: runs over network infrastructure, SANs, SCSI protocol, fiber channels (over ethernet)
- Virtual SAN: multiple SANs on a single fabric, Nexus and MDS switch supports it
- FCP: layered model (TCP), enables SCSI and NVMe over fiber channels, utilizes credit-based flow control
- Fiber Channel Zoning: alternative to physically seperated SANs, restricts devices' access to seperate vSAN in the same fabric, utilizes LUN mapping/masking to secure data stored on LUNs
- Switch modes: does not require domain IDs consumed for every switch (fabric mode), NPV (N Port Virtualization) enables virtualed ports for login to multiple fiber channels, requires an NPIV (N Port ID Virtualization) enabled core switch

Unified Computing:
- Servers: UCS Blade integrate LAN, SAN, management consoles, IMC
- Intersight: Cloud based and on prem monitoring, SaaS, auditing, tagging, endpoint management
- Stateless Computing, UUIDs, service profiles, UCS Manager

![UCS Manager](/assets/img/ccna2/UCS.png)

Advanced Services:
- Data Centers are automated through NX OS model driven approach enabling tools like Ansible or Terraform to apply broad changes to networks quickly and accurately
- nxapi uses NETCONF to interact with configurations via xml
- visore a managed object (tree like structures of UCS components) browser uses xml

![NX-API](/assets/img/ccna2/netconf_nx_api.png)

# CMLLAB

Cisco Modeling Labs provides ways to work with other users' created topologies. There are also APIs for interacting with topologies through Python and a "breakout tool" to telnet or vnc into to nodes running on the CML topology.

![telnet to a CML lab topology node](/assets/img/ccna2/telnet_cml_lab.png)

# DEVNAE

There are many networking tools that share functionality across different platforms: Windows, Linux, Cisco OS (IOS/NX-OS). They can be performed manually if the right CLI command is used or can be scripted via Python or Bash or other scripting language. Further automation can be achieved on scaled infrastructure with NETCONF, RESTCONF, or gRPC tools. Model driven approaches enable these APIs to work of network protocols like HTTP(S), SSH, and TLS. Git is used for version control and a part of the CI/CD pipeline.

APIs:
- REST used to standardize Create, Read, Update, Delete HTTP commands and is an example of Stateless client-server model
- Postman GUI to practice interacting with API
- Requests Verbs: GET, POST, PUT, PATCH, DELETE
- Request Headers: URL, content-type, value, cookies 
- Server Responses: 1XX - informations, 2XX - successful, 3XX - redirecting, 4XX - client error, 5XX - server error

Python:
- (Non)Interactive mode to run scripts or commands line by line
- PIP for library installing to device
- Netmiko library to SSH to network devices
- Requests library to send HTTP commands

Bash:
- Linux provides cli commands such as ping, ifconfig, and traceroute
- To store network changes like IP configurations there is a file named resolve.conf, and the data is stored in the /etc/network directory
- netplan command can apply changes in yaml files to persist the changes over system reboots

![netplan persisting ip configuration](/assets/img/ccna2/netplan_persisting_configs.png)
![resolve.conf yaml file](/assets/img/ccna2/resolveconf_persisting_configs.png)

Git:
- Stores multiple copies of code bases as changes are made and possibly commited to a remote repository
- Enables multiple people/teams working on the same code base
- Remote sites (like the one hosting this page) can be used to see who applied what change and administrate merges and/or branching in the code base through tools like pull requests