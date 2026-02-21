# CCNA week 6 notes

- [CCNA 200-301]( #ccna )
- [Introduction to Network Simulations with Cisco Modeling Labs | CMLLAB]( #cmllab )
- [Understanding Cisco Network Automation Essentials | DEVNAE]( #devnae )

### CCNA

There is a CPT IP routing config lab today. My initial setup did work, but I couldn't figure out if a ping to the rest of the subnet work because I didn't set up any endpoints. Even though I could ping the router interfaces in the other LANs, turns out this is the wrong config.

![single ip routes](/assets/img/ccna6/wrong-ip-route.png)

A first note to remind myself is that ip routes with both interface and next hop is best practice. The more info the system has, the better, so ideally configs would look like this (first two wrong, last two correct):

![corrected and incorrect routes](/assets/img/ccna6/correct-ip-route-config.png)

A second note is that the mask will control which range of IPs the routes will take, so I will need to cover that detail when it comes to the exam. In particular, converting mask to DDN will be important. A mask of /32 will host a single IP address, a mask of /24 will host 255 IPs including broadcasts, a mask of /16 will host 2^16 - 1 hosts including broadcasts, and so on.

Oftentimes no warning or error is shown when an incorrect mask is config'd.  It is hard to tell if the DDN mask inputted will match the slash mask in the diagrams. A good method to verify is the `show ip route` command, since it will show the mask calculated from the config:

![show ip route](/assets/img/ccna6/show-ip-route-result.png)

Another lab showing how to configure routers has it so that to reach seperate hosts, every router must be configured correctly. This lab featured 3 branching routers and a core. 

![lab topology](/assets/img/ccna6/lab-118-topo.png)

Even though the branches had the routes to their subnets they were unreachable prior to the core router being configured.This screenshot shows before and after the core router has routes to each branch:

![core configured](/assets/img/ccna6/before-after-core-config.png)

The third lab this week dealt with DHCP relaying. After configuring the relay agents on the branch routers, each host gets an IP address in the specified subnet.

An interesting change I did was to re-configure the DHCP server's excluded addresses for the 2 subnet. However simply changing the config does not alter the PC's IP address. 

![new dhcp exclude range](/assets/img/ccna6/dhcp-excluded.png)

After cycling the power on the host in that subnet, the new IP chosen was one less than previous IP instead of one more than the highest excluded address. I expected the DHCP server to offer an IP address that was just outside of the exclusion range. I'm not sure if that's a quirk of CPT or if that's how the DORA process works.

![dhcp provision one less](/assets/img/ccna6/dhcp-ip-bad-provision.png)

So then I to tried to power cycle the DHCP server and the host and it wound up with an IP much closer to the highest excluded address (.44). That seems better. The turning it off and on really does work.

The last CPT lab this week will practice an OSPF implementation. Some interesting detail I am jotting down here is that there is logging showing up by default that I did not see with the CML lab. Manually changing the router id would add the new entry to the LSDBs, but not replace old ones; the logging here is also shown when that router id change occurs:

![new ospf neighbor](/assets/img/ccna6/ospf-neighbor-router-id-change.png)

The running-config shows this logging setting in the router section. 

![ospf logging](/assets/img/ccna6/running-config-ospf-loggin.png)

There is also a setting to place neighbors directly without doing the LSA exchanges, but I don't know what purpose this has. It does not show those manually entered routers under `show ip ospf neighbors`.

![neighbor not shown](/assets/img/ccna6/ospf-neighbor-config.png)


### CMLLAB

Continuing to learn more about STP in CML we can set up our own lab to try out RootGuard features. To inspect the nodes I am using the breakout tool to make terminal windows popup instead of using the tiny half-window in the lab page. Setup for this is simple and was discussed in week 2's notes

![cml breakout](/assets/img/ccna6/cml-breakout-terminal.png)

The topology after the setup was completed is shown below. There was two distinct topologies, left and right, which were linked creating one large topology.

![spt topology](/assets/img/ccna6/spt-topology.png)

The connection between the -9 and -0 switches have root guard enabled on their g0/3 interfaces. Shown below is that configuration with log messages show root guard being activated:

![enable root guard](/assets/img/ccna6/root-guard-enabled.png)

And here is the spanning-tree summary on one of those root guard enabled switches. Noted is the status of the root guard enabled interface g0/3 with type P2p *ROOT_INC (root inconsistent). The port has been blocked once the superior BPDU was sent over to that side.

![spt summary](/assets/img/ccna6/spt-summary-root-guard.png)

After the second connection is added between the -4 and -1 switches there is a change in the root bridge of the entire topology due to the lowered MAC address of the left side's root bridge (5254.0009.64fb):

![spt root changed](/assets/img/ccna6/spt-summary-root-changed.png)

However after the -4 and -1 link is removed, the original bridge (5254.000a.3b96) is now the root. The -0 switch which had it's root bridge changed show it is now the root once more:

![spt root returned](/assets/img/ccna6/spt-summary-root-returned.png)


### DEVNAE

PYang is a command line utility for working with YANG models inside Python. A module can be built which implements a class representing the YANG model that is passed to it using the commands:

```bash
export PYBINDPLUGIN=`/usr/bin/env python -c 'import pyangbind; import os; print ("%s/plugin" % os.path.dirname(pyangbind.__file__))' `
pyang $HOME/RFC/ietf-interfaces.yang --plugindir $PYBINDPLUGIN -f pybind -p $HOME/RFC/ > ietf_interfaces.py
```

This python script that gets output can be loaded and interactions with the interface module can be done through the python terminal. 

The yang module `ietf-interfaces` is included in the lab. To test this with another module we can download the module from the yang set created from the connected router as described in the lab's steps. I was able to view the routing module in this way. It shows it depends on the `ietf-interfaces` and `ietf-yang-types` modules. The -p flag passed to pyang tells it to look in the directory for other yang modules. However some errors are encountered when running the pyang script on the `ietf-routing` module downloaded from the yang suite.

![yang module error](/assets/img/ccna6/yang-module-error.png)

There is a github repository with updated modules. If I try this with the github `ietf-routing` module some other errors occur:

![pyang error 2](/assets/img/ccna6/pyang-error.png)

But the pass to `--lint` shows the module is valid validation. I will try to delete that area containing the problem lines 

![pyang loaded](/assets/img/ccna6/pyang-loaded.png)

And that did the trick. Turns out according to PYang docs, only openconfig modules are thorougly tested. But I am able to load the script `ietf_routing.py` generated script and interact with the routing class the bindings created.

![python routing class](/assets/img/ccna6/python-routing-class.png)

The mind maps set for this week include ch. 12-18 and are Magic Number and Routers.

```yaml
Magic Number:
  -Simple subnetting technique
  -Determines subnet sizes
  Steps:
    Identify subnet octet:
      - Memorize mask size relative to DDN
      - /0 - /7 first octet, /8 - /15 second octet, /16 - /23 third octet, /24 last
    Convert mask to DDN:
      - bit position left-to-right corresponds to power of 2 value (76543210)
      - helps identify subnet octet
    Subtract subnet octet from 256:
      - this is the magic number that gives the network bit
      - the magic number divides 256 evenly and that will give number of subnets
  Examples:
    100.50.68.34 /27:
      - /27 = 11111111.11111111.11111111.11100000 = 255.255.255.224
      - subnet octet is in 4th position
      - 256 - 224 = 32, 32 goes into 256 8 times so there are 8 subnets
      - 100.50.68.0, 100.50.68.32, 100.50.68.64, ... 100.50.68.224
      - the given ip resides in the 100.50.68.32 subnet
    7.127.0.2 /9:
      - /9 = 11111111.10000000.000000000.000000000 = 255.128.0.0
      - subnet octet is 2nd position
      - 256 - 128 = 128, 256/128 = 2 times 2 subnets
      - 7.0.0.0, 7.128.0.0 are the subnet ids
      - the given ip resides in the 7.0.0.0 subnet
```

```yaml
Routers:
  - layer 3 devices which act to move packets between different networks
  ARP:
    - how hosts and network devices learn routes to other devices
    - helps encapsulation of ip frame data in ethernet frame data (MAC)
    table components:
      - ip addresses
      - MAC addresses
      - outgoing interface
  verify:
    show ip arp:
      - displays arp table
      - also show arp
    show ip protocols:
      - info about dynamic protocols configured
    show ip int br:
      - displays interface/vlan and ip address
      - line statuses/protocol status (layer 1 + 2)
      - ip routing status
      - show protocol displays ip and mask
    show ip route:
      - displays ip routing grouped by networks
      - ip subnet and mask
      - code is for method of routing (dynamic/static)
  config:
    default routes:
      - ip routes that sends any packet not listed in routes to the next hop address listed
      - ip route 0.0.0.0 0.0.0.0 <NEXT HOP ADDRESS>
    static routes:
      - configured in global mode
      - permanent mode skips checks for interface status or next hop router availability to always include in routing tables
      - ip route <IP ADDRESS> <DDN MASK> <OUTGOING INTERFACE> <NEXT HOP ADDRESS>
      - floating static routes are high administrative distance routes used for backups
  ROAS:
    - on a router used for inter-VLAN communication
    - used if a layer 3 switch is not available or if router ports are limited
    - a single interface can be used for many IP routes
    subinterface:
      - replaces VLANs on routers
      - <INT TN>.<VLAN ID> ex: int g0/1.20 (interface gigabit 0/1 for the id 20 VLAN)
      - ip address assignable
      - shows up in other verification commands alongside normal interfaces
```
