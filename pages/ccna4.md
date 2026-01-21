# CCNA week 4 notes

- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )
- [CCNA 200-301]( #ccna )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )

### DEVNAE

A lab is administered to implement YANG driven telemetry via gRPC real-time streaming. The telemetry is set up from the config terminal on the device and a data visualization software called Grafana displays the telemetry in a bar graph (by the time of the screenshot the connection broke - might have been the big ping instructed in the lab?). There is a bit of xpath used here to filter out the xml parts in the NETCONF that we will visualised. This was an interesting lab that I will have to come back to at some other time.

![netconf subscription setup](/assets/img/ccna4/streaming-netconf-setup.png)

![streaming to grafana](/assets/img/ccna4/grafana-telemetry.png)

The network automation essentials class continues with an explanation of secret/password storage methods in Python, Linux, Ansible, and Hashicorp.

Python:
- configparser module to parse info from a .ini file into a dictionary data structure
- interfaces with Hashicorp vaults via hvac

Linux:
- environment variables can be used to hide sensitive info in scripts that may be shared in a git repository
- storing info in a .env file and putting that inside a .gitignore is a safe alternative to hardcoding passwords and infrastructure data

Ansible:
- ansible-vault is a function to allow encryption via AES-256 with a vault password stored outside the project repository

Hashicorp:
- hashicorp-vault is another utility that is centralized and info is retrieved via REST api 

### CCNA

One [lab](https://youtu.be/l-Tehu4sXzc?si=X6Ibby3nOoDCpwxS) this week involved troubleshooting ports on switches. An interesting aspect of this lab is it allows you to follow TCP/IP model guidelines to diagnose the issues. I originally completed the lab in Cisco Packet Tracer (CPT); it involved checking for duplex and speed mismatches, as well as some other things like port security that I had to watch the video for:

![port security config](/assets/img/ccna4/int-troubleshoot-portsecurity.png)

I recreated the lab in CML for practice:

![CML troubleshooting lab](/assets/img/ccna4/cml-troubleshoot.png)

But some of the config options in CML and CPT seem to differ, in this case regarding `no negotiation auto` not being a interface config option on CPT:

![CPT config differs](/assets/img/ccna4/no-no-negotiation-auto.png)

I believe when speed/duplex is set on CPT it disables the autonegotiation. This actually might be some differences in the OSes running on the CML and simulated on CPT. I have the lab from CML saved [here](/assets/cmllab1.yml).

Another [lab](https://www.certskills.com/clab115/) for this week involves setting up trunking ports on switches. This lab was done in the CPT. There was a few notable things about this lab. When I thought I had configured everything correctly some trunks still weren't showing on Switch 3:

![non trunking vlan3](/assets/img/ccna4/not-trunking.png)

![switch3 access port](/assets/img/ccna4/access-port.png)

It does not say in the lab to change this interface's config for DTP. It is just stuck on auto, but I guess that is the point. The verification here is that DTP with one side set to auto and the other an access is not a trunk. I guess I was overthinking it (again).

A second issue I noticed was there was a BPDU setting enabled on Switch 2 blocking the port, as that deals with spanning trees I was going to let that one pass. The labs of the future will deal with these settings.

![bpdu-guard-blocked](/assets/img/ccna4/bpdu-guard-blocked.png)

A third lab had me dive in to STP/RSTP and mess around with roots and costs. I noticed that configuring BPDU guard and portfast could potentially break the network by allowing loops if not configured on an access port, so a good note would be the command `show run int [interface-name]` does work to show that info. Just, not on CPT, but there it can been seen in the full `show run`:

![command not on CPT](/assets/img/ccna4/command-not-on-CPT.png)

### CMLLAB

In the lab I found a topology on the referenced [github](https://github.com/CiscoLearning/ciscolive-brkcrt-2059/tree/main/sample-labs/1-ccna-vlan-config). In this assignment I was required to do the following

* Implement VLAN assignments on two switches
* Configure a dot1q trunk between those switches
* Configure a dot1q trunk between one switch and a router that will handle the routing for the VLANs
* Configure VLAN trunking on that router to act as a gateway between VLANs
* Verify that connectivity exists between hosts in the lab on each VLAN, and that the connectivity goes through the router

This was something I could practice my knowledge gained from the OCG. Here to show my work:

* Implement VLAN assignments on two switches
![config vlans](/assets/img/ccna4/config-vlans.png)
* Configure a dot1q trunk between those switches (the g0/3 config below)
* Configure a dot1q trunk between one switch and a router that will handle the routing for the VLANs (the g0/0 config below)
![trunk between switches](/assets/img/ccna4/trunk-between-switches.png)
* Configure VLAN trunking on that router to act as a gateway between VLANs
![router trunking](/assets/img/ccna4/router-trunks.png)
* Verify that connectivity exists between hosts in the lab on each VLAN, and that the connectivity goes through the router
![verified connection](/assets/img/ccna4/verify-ip-connected.png)
![ping working](/assets/img/ccna4/ping-to-vlan20.png)

The last screenshot for verifying had some initial pings fail, which was due to configuring switch 1 and 2 backwards. The configurations were corrected as shown in screenshot above and the pings began to come through.

One note here to remind myself: to save changes to configs on CML, click "fetch" in the device's config tabs before downloading the lab. I have saved the before and after files for these labs ([here](/assets/cmllab0.yml) and [here](/assets/cmllab0done.yml))

Finally mind maps from chapters 6-8, covering Login configuration, Interfaces, VLANs:

```yaml
Logins Config:
  contexts:
    console:
      - line console 0
      - login (for password security)
    telnet:
      - line vty 0 (1-15)
      - login (for password security)
      - transport input <all|telnet>
    ssh:
      - line vty 0 (1-15)
      - transport input <all|ssh>
      - crypto key generate rsa <modulus modulus-value>
      - ip ssh version 2 (to allow only SSHv2)
  username:
    - to access enable for seperate users
  password:
    - config for either console/vty contexts or username
  enable:
    - enable secret <password> 
    - secures the privileged EXEC mode
  AAA server:
    - external server used to manage user/pw
```

```yaml
Interface:
  type:
    - Ethernet
    - FastEthernet
    - GigabitEthernet
    - TenGigabitEthernet
  port numbers:
    - designate port position after type, usually row/column (e.g. FastEthernet1/2)
    - can also designate ranges (e.g. TenGigabitEthernet1/0/11-15)
  context:
    - config-if
    - speed, duplex, auto-mdix, description, (no) shutdown
  verifying:
    show running-config:
      - displays each interface's config
      - can single out individuals by adding type+number to command
    show interface (type+number):
      status:
        line:
          - physical cable status (damage cable, unplugged, shutdown, etc)
          - up, down, or administratively down (when shutdown command is issued)
        protocol:
          - ethernet protocol status (mismatches, software has blocked port, switch on opposite end turned off)
          - up, down, err-disabled (for software that manages port security or STP)
        interface:
          - overall phys + eth
          - disabled, notconnect, err-disabled, or connected
  configuring:
    interface (type+number)(range):
      - in terminal context
      - (no) speed, duplex, description, shutdown, mdix auto (pinout detection), negtiation auto
  autonegotiation:
    - prevents mismatches (speed, duplex, pinout)
    - based of fast link pulses sent before any traffic over ethernet cables
    - transmits speed, duplex, and pinout capabilities
    parallel detection:
      - switch logic to figure out speed and duplex when one side has no autonegotiation
```

```yaml
VLAN:
  uses:
    - create multiple broadcast domains on one switch
    - seperate network entities (subnets)
  trunking:
    - ethernet protocol allows data transmitted to vlans
    - inserts a vlan tag id (1-4095) in header
    - ports operate as trunk, access, or dynamic
  verifying:
    show:
      vlan:
        - brief lists interfaces assigned to vlan
        - id lists trunking, access ports
        - native vlan is the default (vlan 1), can be set
      interfaces:
        - status lists vlan id on access port or just trunk
        - type+number switchport lists operational mode, voice/access vlans
      mac address-table:
        - MAC entries list vlans associated with the device
  configuring:
    vlan:
      - config-vlan to set name, id, (no) shutdown
    interface:
      switchport:
        - access for endpoint device connections
        - trunk (native vlan id) for ports between switches
        - daynamic (auto|desirable) allows negotiation of trunking, desirable initiates/auto waits
  vtp:
    - vlan trunking protocol
    - logic that shares vlan info among connected devices
    - config vtp mode (server|client|transparent|off)
  voice:
    - vlan trunk to have IP phones operate over ethernet
```

