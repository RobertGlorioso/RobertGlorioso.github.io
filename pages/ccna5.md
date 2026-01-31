# CCNA week 5 notes

- [CCNA 200-301]( #ccna )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )
- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )

### CCNA

There is a big lab to do dealing with vlan, STP and etherchannels in CPT. The [video](https://youtu.be/RX2YzW7CayU?si=FJV0xJY5pGDK4eNs) goes through all the config intracacies in CPT and I have my summary here as well:

![per-vlan-spanning-trees](/assets/img/ccna5/vlan-stp-summary.png)

I could not get my etherchannels up unfortunately, but Wendell explains how to get it to work in the video. Might be due to configuring via `int range` instead of each interface seperately, but CPT is wonky in this regard. I may come back to this one eventually.

![etherchannel down](/assets/img/ccna5/etherchannel-broken-cpt.png)

The next lab dealt with configuring static routes that was found on this Wendell's blog [page](https://www.certskills.com/clab119/). It had some interesting aspects to note like the router did not need the pc it was connected to to have a static route configured. Which, if I had to guess, this is due to it being in the same subnet as the router (LAN), and ARP learns that route for it. The initial pings took 2 attempts which would correlate with going through 2 different subnets for the table to fill the neighboring device's MAC-addresses into the ARP table for the first time:

![ping fills arp](/assets/img/ccna5/ping-arp-loading.png)

Another note is that I configured each route with that router's interface instead of the next hop router address. Both pieces of info are conveniently displayed in the hover-over action on CPT:

![ip and interface](/assets/img/ccna5/route-interface-ip.png)

I suppose both work in this topology but might be best practice to use the next hop address instead in case cables are moved?

Third lab this week had me working with switch virtual interfaces (SVI). This are the interVLAN routed ports in the layer 3 switches. So some notes, as a layer 3 switch, it needed to have the `ip routing` global command set. All seemed well after setting up ip routes:

![routes connected](/assets/img/ccna5/connected-route-vlans.png)

But the pings were not coming through. This screenshot showed a problem I had overlooked:

![vlans are bad](/assets/img/ccna5/problem-with-vlans.png)

After that was fixed the pings went through. Understanding that the pc's are following the gateway which is set in a subnet that matches the vlan the pc is in was crucial:

![hosts use gateways in subnet](/assets/img/ccna5/host-subnet-gateway.png)

Another CPT lab that is a ROAS config like the lab in CML class last week. Turns out I did forget the switch's interface config of `switchport trunk encapsulation dot1q`. However, that, it turns out, is needed only on older switch models and CML. My CPT version did not have it available.

![command not available](/assets/img/ccna5/old-config-command.png)

The ['ad hoc' lab](https://youtu.be/9qa2cLaZZHQ?si=JjJtFIZwgTHfTD2p) this week practiced troubleshooting with traceroute. In the following screenshot, having traceroute executing shows that while the problem was there the first 2 attempts fail to get a response from the initial route. TTL had incremented to 3 as the problem was fixed and router at 3rd hop responded, and the fourth shows it reaches destination.

![traceroute-completed](/assets/img/ccna5/traceroutes-completing.png)

### CMLLAB

This week's lab will implement a topology featuring STP using Cisco Modelling Labs. Using the packet capture feature allowed to examine BPDUs in close detail:

![ethernet frame bpdu](/assets/img/ccna5/bpdu-ethernet-frame.png)

I was tasked with allowing bpdufilter to do its magic and block those frames from forwarding on that access port. Packet capture made it clear there was no more incoming frames from the access switch being sent downstream.

A loop was created by bridging together the access nodes in the topology and setting each interface to be in portfast mode. The network would no longer function correctly shortly after turning on those interfaces. The packet counters rose quickly and pings stopped getting replies.

![looping network](/assets/img/ccna5/spt-loop.png)

### DEVNAE

I am taking a closer look at Ansible this week. I tackled a lab where I could write a custom script for this topology

![lab topology for ospf](/assets/img/ccna5/ansible-lab-top.png)

I attempted an OSPF implementation, but figuring it out in the playbook is a little tricky. My first attempt:

```yaml
- name: PLAY 1 - MANAGING INTERFACES 
   hosts: all 
   connection: network_cli 
   gather_facts: no 

   tasks: 

    - name:  TASK 1 - MANAGE INTERFACES 
      ios_config: 
        parents: interface Loopback0 
        commands: 
          - ip address 172.16.1.{{ loopback_id }} 255.255.255.0 
          - description Configured_with_Ansible 
    - name:  TASK 2 - MANAGE ospf int 1
      ios_config: 
        parents: interface g1
        commands: 
          - ip address 10.1.1.{{ ip4 }} 255.255.255.0 
          - description Configured_with_Ansible 
    
    <ommiting for brevity>

    - name:  TASK 5 - MANAGE ospf int 4
      ios_config: 
        parents: interface g4
        commands: 
          - ip address 10.1.4.{{ ip4 }} 255.255.255.0 
          - description Configured_with_Ansible 
    - name: TASK 6 - ospf conf
      ios_config:
        parents: router ospf 1
        commands:
          - network 10.1.0.0 0.0.255.255 area 1
```

This breaks the lab as each router had a different interface acting as the port for MGMT and changing the IP messed with the existing SSH config for ansible. Rather than update that ansible.cfg file I will try defining host variables. The host variable will be different for each host, so I can choose which set of interfaces to update a list in the inventory, passing it through to the playbook via a `loop`. After struggling with weird errors due to indenting the loop wrong, the new playbook I came up with is this:

```yaml
---
- name: PLAY 1 - MANAGING INTERFACES 
   hosts: all 
   connection: network_cli 
   gather_facts: no 

   tasks: 

    - name:  TASK 1 - MANAGE INTERFACES 
      ios_config: 
        parents: interface Loopback0 
        commands: 
          - ip address 172.16.1.{{ loopback_id }} 255.255.255.0 
          - description Configured_with_Ansible

---
- name: PLAY 2 - CHANGE ROUTES 
  hosts: all 
  connection: network_cli 
  gather_facts: no 

  tasks: 
    - name: Debug my_list variable
      ansible.builtin.debug:
        var: item.name
      loop: "{{ ints }}"
    
    - name: TASK 1 - set ips
      ios_config:
        parents: interface {{ item.name }}
        commands:
          - ip address {{ item.ip }} 255.255.255.0
      loop: "{{ ints }}"

    - name: TASK 2 - static routes
      ios_config:
        commands:
          - ip route {{ item.ip1 }} 255.255.255.255 {{ item.ip2 }}
      loop: "{{ nexts }}"
    
---
- name: PLAY 3 - OSPF
  hosts: all 
  connection: network_cli 
  gather_facts: no 
  
  tasks:
    - name: TASK 1 - ospf conf
      ios_config:
        parents: router ospf 1
        commands:
          - network 10.0.0.0 0.255.255.255 area 0
```

And here is where I defined those `nexts` and `ints` viarables seen in the curly braces. When the playbook is run the dicts in the variables are assigned to the `item` variable, and values are accessed through whatever key is used. For example, for the first task, the `item.ip` value will be 10.0.32.3 when the third router is in play.

```
[cpe]
R1 location=SF loopback_id=1 ints='[{"name":"g2","ip":"10.0.12.1"},{"name":"g3","ip":"10.0.13.1"}]' nexts='[{"ip1":"10.0.23.2","ip2":"10.0.21.2"},{"ip1":"10.0.32.3","ip2":"10.0.31.3"}]'
R3 location=LA loopback_id=3 ints='[{"name":"g1","ip":"10.0.31.3"},{"name":"g2","ip":"10.0.32.3"}]' nexts='[{"ip1":"10.0.12.1","ip2":"10.0.13.1"},{"ip1":"10.0.23.2","ip2":"10.0.21.2"}]'

[core]
R2 loopback_id=2 ints='[{"name":"g1","ip":"10.0.21.2"},{"name":"g3","ip":"10.0.23.2"}]' nexts='[{"ip1":"10.0.13.1","ip2":"10.0.12.1"},{"ip1":"10.0.31.3","ip2":"10.0.32.3"}]'
```

However, I'm not getting the neighbors to join up. There seems to be something wrong with the LSUs being sent between the routers and none of the R1,R2,R3 routers IPs appear in the output of `show ip ospf neighbors`. Looking at the traffic coming from any interface I see the error messages 'bad pkt received' coming from hello messages.

![bad packets](/assets/img/ccna5/bad-pkt-ospf.png)

Even though I can ping on all interfaces, I know it's something related to subnetting, since the initial config doesn't have the ospf problems that I get when I messed around with the IPs. There are bad sources incrementing everywhere like here:

![bad hellos](/assets/img/ccna5/bad-hello-counter.png)

I needed to stare at the topology diagram a bit more to finally get it. So, I had two different subnets in each link in the new config. That is wrong. This is the issue with the bad source packets, so I fixed up a new ip route to test it and it worked:

```
[cpe]
R1 location=SF loopback_id=1 ints='[{"name":"g2","ip":"10.10.0.1"},{"name":"g3","ip":"10.30.0.1"}]' nexts='[{"ip1":"10.20.0.2","ip2":"10.10.0.2"},{"ip1":"10.20.0.3","ip2":"10.30.0.3"}]'
R3 location=LA loopback_id=3 ints='[{"name":"g1","ip":"10.30.0.3"},{"name":"g2","ip":"10.20.0.3"}]' nexts='[{"ip1":"10.10.0.1","ip2":"10.30.0.1"},{"ip1":"10.10.0.2","ip2":"10.20.0.2"}]'

[core]
R2 loopback_id=2 ints='[{"name":"g1","ip":"10.10.0.2"},{"name":"g3","ip":"10.20.0.2"}]' nexts='[{"ip1":"10.30.0.1","ip2":"10.10.0.1"},{"ip1":"10.30.0.3","ip2":"10.20.0.3"}]'
```

![good hellos](/assets/img/ccna5/good-pkt-ospf.png)

The mind maps set for this week include ch. 9-11 and are STP, Etherchannels, and Subnets.

```yaml
STP:
  - preventing loops (broadcast storm)/MAC table corruption
  - one switch becomes root
  - convergence occurs after a topology change
  port roles:
    - root port
    - designated port
    - alternate port replaces root port in rapid convergence
    - backup port replaces designated port in rapid convergence
  port states:
    stable: 
      forwarding:
        - root switch forwards all frames
        - other switch port with cheapest path to root switch are a root port
        - other switch elect a port based on cost to get to root to be designated port
        - learns mac addresses from BPDUs
      blocking:
        - occurs on other ports not designated/root
        - called discarding in rapid
    transitory:
      - blocking ports transition to forwarding after root port fails (convergence)
      - rapid convergence adds orth port roles that allow switches to skip the listening phase
      listening:
        - former blocking port becomes listening before forwarding to avoid temporary loops while convergence occurs
        - MAC tables are updated to remove those that no longer are being received
      learning:
        - after forward delay timer ends to learn which switches are now sending here
        - in rapid, alternate ports are placed directly to this state
  BPDU:
    - sent between all switches running spanning-tree
    bridge ids:
      - priority number (4096x + vlan id) and MAC address
      - root and sender BID
    costs:
      - multiple of 4096
      - lowest determines root port elections
    timers:
      - config on root switch
      - hello timer how frequent (default every 2 sec)
      - max age timer how long switch waits for hellos
      - forward delay time between listening to learning to forwarding
  config:
    - pvst+ (per vlan spanning tree)
    rapid-pvst+:
      - alternate ports eliminate learning
      - blocking becomes discarding?
    port costs:
      - default based on speed capabilities
      - primary / secondary roots
    portfast:
      - moves port to forwarding state
      - speeds up convergence
      - dont use on bridging ports (access only)
    BPDU Guard:
      - block port if BPDUs are sent to the port
      - used with access ports only along with portfast
      - BPDU Filter an alternate feature that allows STP but disables portfast under certain conditions
    Root Guard:
      - reacts to BPDUs that would elect a new root switch
      - disables ports which receive superior BPDU than current root
    Loop Guard:
      - when unidirectional paths are in play due to failing fibre link
      - disables a possible designated port on a root / alternate port when unidirectional loop is found
```

```yaml
Etherchannels:
  - layer 2 or 3
  - load balanced, parallel links
  - updates bandwidth value, cost for STP/IP routes
  Dynamic protocols:
    LaCP:
      - IEEE standard
      - modes active/passive
    PaGP:
      - Cisco propriatary
      - modes desirable/auto
  Config:
    Int Range:
      - All interfaces must have consistent, speed, duplex, trunk/access
      - channel-group <id> mode <active|passive|desirable|auto|on>
  Show:
    - running-config <int name> lists portchannel info
    interface <name> etherchannel:
      - lists ea interface in a portchannel
    portchannel <etherchannel|summary|detail>:   
      - lists more or less detailed info
```

```yaml
Subnets:
  IP Address parts:
    - network first (classful)
    - subnet middle
    - hosts last
    - network + subnet = prefix
  DDN:
    - dotted decimal notation represent single IP address
    - 4 0 - 255 values "octets"
  Masks:
    - up to 30 1's the rest 0's
    - number of 1's is the mask, ex /25
    - /25 converts to 255.255.255.127 (111111111.111111111.11111111.10000000)
    - defines where hosts bits resides and how many hosts / subnet
    classful masks:
      - A is /8
      - B is /16
      - C is /24
  Classful IP:
    Class A:
      - first octet 1-126
      - largest networks
      - 8 host bits, 16 subnet bits, 8 network bits
      - private IP starts with 10
    Class B:
      - first octet 128-191
      - medium networks
      - 16 network bits, 8 subnet bits, 8 host bits
      - private IP starts with 172.16 - 172.31
    Class C:
      - first octet 192-223
      - smaller networks
      - 24 network bits, 8 subnet + host bits
      - private IP starts with 192.168.0 - 192.168.255
    - Class D 224 - 239 first octet for multicast
    - Class E 240 and up first octet for research
  Classless IP:
    - variable length subnet masks allow flexible host provisions
    - prefix and host only defined by masks
    - does not use the A, B, C defaults
    - subnets split into CIDR blocks
```