This ansible-network repo is a work in progress and currently used to manage baseline configurations and access port configurations.

##### Planned Implementations:

1. Dynamic Interface Descriptions for switch trunk uplinks, based on CDP information.
2. Static Routing and "Core" campus switch configurations for SVIs and Layer 3 addresses.
3. Dynamic routing with OSPF and EIGRP.


Note: This has been tested on 12.X, however some of the ios-baseline tasks do
not work, such as tftp-block size. Therefore they have been commented out.


#### IOS-Baseline Playbook


The current playbook that manage basline paramaters are below, with most variables declared in the network-all variable file:

1. ios-aaa-config
 * Used to set radius AAA authentication and AAA accounting to the radius servers defined in the network-all variable file.
2. ios-banners
  * Used to set login banners upon SSH connection, and login. All other banners are removed after connection.
3. ios-ntp
  * Sets NTP server, which is defined by the ntp_servers variable.
4. ios-system
  * Sets hostname, management IP address,DNS Servers, enables DNS lookup, enables cdp and lldp, disables http server, enables PVST mode sets SSH defaults, encrypts cleartext passwords and sets block size for tftp (currently commented out for testing). Device specific variables (hostname, IP address, etc) are located in the host_vars device variable file (in this case test-3750-04-02.lab.internal)
5. ios-snmp
  * Configures SNMPv2 for servers defined in roles vars/main.yml variable file, along with enabling traps and setting the source interface. **Note:** This needs a revision, as it is a little bit dated. It was my first attempt at a role and needs some better regex and modification to capture specific datasets (server address, along with community strings), rather than replacing entire lines. It works, but not very flexible.
6. ios-syslog
  * Configures syslog variable, as well as device loggin/buffer sizes, while setting the source interface for syslog traps.

#### IOS-Interfaces Playbook


This playbook generates interface configurations for access ports, along with configuring VLANs that should be used on the switch. **Note:** This role is a work in progress, port channel configuration is pending. Trunk ports and access portsworking as intended. 

1. The "site" group_var file (in this case "lab"),  is where all the site VLAN variables are declared by name. This playbook enforces vlans based on these named variables. These named variables are then used to generate interface configurations (see below).

2. The host_var file for the 3750 (in this case "test-3750-04-02.lab.internal"), is where are the device specific interface configurations are declared. A jinja2 template is then used to loop the VLAN dictionary in order to generate the config file, then pushed to the device.  Be sure to declare the path for the configuration files in the network-inventory folder, to a folder that exists on the machine you execute it on.  

See the test-3750-04-02.lab.internal host_vars file for an example on how to declare interfaces and turn them up/down. If custom interface descriptions are required, then you can remove the jinja2 variable from the host_var file, and input your own. See madeupswitch for an example of this.

#### Network-Baseline-Config

This playbook generates a configuration for dropping on a device during an initial deployment. See the role readme.md for more info. 
