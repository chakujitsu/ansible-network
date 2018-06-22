network-baseline-config
=========

This role is used to generate a baseline configuration file to configure a new device for field deployment. This role utilizes a Jinja2 template, which is filled by variables defined by a host and group variable files. This role is run locally on the machine in which the it was executed to generate an initial baseline config, and then stored into the ansible/configs/ directory. After production deployment, the variable files created are used to enforce a desired state by other roles.


# Simplified Steps
------------

1. Create site group_vars file if it doesn't exist. You can clone the site_vars template file located at/ansible/environments/network/group_vars/template. This must be named after the site abbreviation. Fill in group_vars block shown in document below.  <br>
```
vi /ansible/environments/network/group_vars/sitename
```
2. Create device host_var file. You can clone the host_vars template file located at /ansible/environments/network/host_vars/template. This must be named after a resolvable DNS entry (cannot be internal). Fill in header, and desired blocks (access, trunks, and port-channels).  <br>
```
vi /ansible/environments/network/host_vars/device.site.domain.com
```
3. Add group and host to inventory file. (See example below if you are a dummy and don't know how) <br>
```
vi /ansible/environments/network/network-inventory
```
4. Commit your files to Github.<br>
```
git add -A
git commit -m 'commit description'
git push
```
5. Run the network-generate-config playbook if desired on local machine. <br>
```
/ansible/ansible-playbook -i ./environments/network/ ./playbooks/network/network-baseline-config.yml
```
Otherwise the config will be generated into the ansible/configs/ folder and then pushed to Github. <br>
6. View your configuration in the /ansible/configs/ folder, or on github, and provision your device. <br>


# Requirements
------------

The variable file creation is required as below:

Host declared in the 'network-inventory file', under the site abbreviation. A group_vars file is required for the site, as well as the host_vars file for the device you are generating a configuration for.

```
---
# Network Inventory
network-all:
  children:
    lab:
      hosts:
        test-3750-04-02.lab.internal:
    lab2:
      hosts:
        madeupswitch.lab.internal:

```

**host_vars file**: This is the name of the device you are provisioning, which was declared in the inventory file. The filename is hostname, and this file is used to generate the baseline and manage the device after deployment. The naming standard for the host_var file and DNS entry should match a resolvable DNS entry. This file contains variables that declare configurations that are unique to the device (such as hostname, interface configuration and so forth).


**group_vars file**: This file will be named as the site abbreviate after the site in which devices are located. This file contains variables that are unique to the site.

```
├── environments
│   ├── network
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   ├── lab

```

# Role Variables
--------------

## **all:**

configs_dir: This variable dictates where the role will save a generated configuration. If running from a directory other than the declared variable, this will need to be changed to suit your setup. There is a commented out variable that I use for the path where I keep my repositories locally on my machine. Because this role is all performed locally, you may need to adjust this path to your current Ansible directory path if running locally. If being executed in the default directory (/etc/ansible), you can leave the path as it is.

## **group_vars:**

The group_vars site file is used to declare site specific variables. Any VLANs that are used at this site are declared here, along with other variables that are used in other roles.

**site_name**: The name of the site.(required) <br>
**site_abbrv:** Abbreviated site name, used to generate variables that has a character limit (required) <br>
**primary_dns**: Primary DNS server for the switch to use. If left undeclared, network-all values are used. (optional in site group_var file) <br>
**secondary_dns**: Secondary DNS server for the switch to use. If left undeclared, network-all values are used. (optional in site group_var file) <br>
**mgmt_default_gateway**: IP address for all management devices at this site to use on the Management VLAN. (required) <br>
**yit_radius_servers**: Radius servers the switch uses for AAA. If left undeclared, network-all values are used. (optional in site group_var file)<br>
**site_vlans**: This dictionary declares the VLANS by name, which is in turned use to configure the VLAN IDs, name, whether or not to tag with voice, and whether or not to enable DHCP snooping on. VLANs must be declared here, in order to declare access interfaces. <br>
**dictionary-key**: VLAN name. This key is referenced in the host_vars file by name. Please declare in all caps (required) <br>
  **vlan_id**: String value indicating the VLAN number (required) <br>
  **voice_vlan**: Boolean value that will tag access interfaces with the voice VLAN (optional) <br>
  **dhcp_snoop**: Boolean value that will enable DHCP snooping on the VLAN in which it is true (optional) <br>


The template for the site group_vars file is as follows:

```
---
site_name: "Site-Name"
site_abbrv: "SN"
primary_dns: "1.1.1.1"
secondary_dns: "2.2.2.2"
mgmt_default_gateway: "10.255.255.254"
yit_radius_servers:
  - 10.8.8.8
  - 10.9.9.9
site_vlans:
  VLAN1:
    vlan_id: "1"
    voice_vlan: true
    dhcp_snoop: true
  VLAN2:
    vlan_id: "2"
    dhcp_snoop: true
  VLAN3:
    vlan_id: "3"
    dhcp_snoop: true
  VLAN4:
    vlan_id: "4"
  VLAN5:
    vlan_id: "5"
...
```

An example of the site group_vars file is as follows:

```
---
site_name: "LAB-DC"
site_abbrv: "LAB"
primary_dns: "1.1.1.1"
secondary_dns: "2.2.2.2"
mgmt_default_gateway: "3.3.3.1"
yit_radius_servers:
  - 4.4.4.4
  - 5.5.5.5
site_vlans:
  USERS:
    vlan_id: "100"
    voice_vlan: true
    dhcp_snoop: true
  VOICE:
    vlan_id: "114"
    dhcp_snoop: true
  MGMT:
    vlan_id: "121"
    dhcp_snoop: true
  EPLAN:
    vlan_id: "999"
  WAN:
    vlan_id: "1465"
...
```


## **host_vars:**

**hostname**: String that is the device hostname (required)<br>
**mgmt_vlan_ip**: String that is the device IP address (required)<br>
**mgmt_vlan_mask**: String that is the device subnet mask (required) <br>
**model**: String that is the model of the switch. See the role for all supported model types. (required) <br>

The template for the host_vars file header is as follows:

```
---
#ansible.dns.hostname variables
hostname: "devicename.site"
#Management SVI ip address
mgmt_vlan_ip: "10.1.1.1"
mgmt_vlan_mask: "255.255.255.0"
model: "model"
```

An example for the host_vars file header is as follows:

```
---
hostname: "switch.lab"
#Management SVI ip address
mgmt_vlan_ip: "10.1.21.253"
mgmt_vlan_mask: "255.255.255.0"
model: "2960-x"
```

**all_ports**: This dictionary declares the access interfaces for each VLAN declared at the site. Each VLAN is declared in in group_vars dictionary called "site_vlans". In order to add a VLAN configured for an access interface, it must be declared at the group level. After deployment into production, changing the admin_up boolean to false will turn the port down. Custom descriptions can be configured here as well, however any non unique description is encouraged to use the descriptions declared in the 'all' variable file.

**VLAN**: Key that must match a VLAN that is declared in the site group_vars file.
**name**: Interface speed  (required) <br>
**module**: Interface Module/Switch (optional) <br>
**card**: Interface card (required) <br>
**ports**: Interfaces to be configured. Acceptable input is range(ex. 45-47), list (ex.45,47) or singular(ex. 45). Multiple Values can be input in a new line, see the example below the template.  (required) <br>
**description**: Interface description. Acceptable input is a string (ex. "description") or a variable (ex. {{ user_access_port_description }}). It is preferred that the variables in the group_vars/all file are used if a non-unique identifier is required (such as access ports). (required) <br>
**bpdu_guard**: Boolean value to enable BPDU guard on the interface. (optional)<br>
**admin_up**: Boolean value to turn the interface up or not. (required)<br>

The template for the access ports block is as follows:

```
#############BEGIN ACCESS PORTS############
all_ports:
  VLAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 1-44
    description: "{{ user_access_port_description }}"
    admin_up: true

#############END USER PORTS############

```

An example of the access ports block is as follows:

```
#############BEGIN ACCESS PORTS############
all_ports:
  USERS:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 1-44
    description: "{{ user_access_port_description }}"
    bpdu_guard: true
    admin_up: true
  EPLAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 47
    description: "PRIVATE-EPLAN"
    admin_up: true
  WAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 48
    description: "ISP-WAN"
    admin_up: true

#############END USER PORTS############

```

**all_trunks**: This dictionary declares the trunk interfaces on the device. Multiple trunks can be defined within this dictionary.

The "TRUNK-IDENTIFIER" key is used to identify what the trunk is configured for. (Required)<br>
**name**: Interface speed  (required) <br>
**module**: Interface Module/Switch (optional) <br>
**card**: Interface card (required) <br>
**ports**: Interfaces to be configured. Acceptable input is range(ex. 45-47), list (ex.45,47) or singular(ex. 45). Multiple Values can be input in a new line, see the example below the template.  (required) <br>
description: Interface description. Acceptable input is a string (ex. "description") or a variable (ex. {{ user_access_port_description }}). It is preferred that the string variable follows a format that is consistent with the device it is uplinked to. That format is <connected device> <connected device interface>. For example, "core.hdos Te1/1/1" (required) <br>
**uplink**: Boolean value to indicate if this interface uplinks to another network switch. This feature is currently not implemented. (optional) <br>
**admin_up**: Boolean value to turn the interface up or not. (required) <br>
**snooping_trust**: Boolean value to trust DHCP offers originating from this interface. (optional)<br>
**native_vlan**: VLAN to be used as the native, untagged VLAN. If left un-delcared, default of 1 will be used. (optional)<br>
**bpdu_guard**: Boolean value to enable BPDU guard on the interface. (optional)<br>
**vlans**: VLANs to be allowed accross the trunk. These VLANS must be declared in the site group_vars file, otherwise the role will fail. Reference the VLANS by name, and the value will be auto populated (required) <br>



The template for the trunk ports block is as follows:

```
############BEGIN TRUNK PORTS###########

all_trunks:
  TRUNK-IDENTIFIER:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 45-46
    description: "description"
    uplink: true
    admin_up: true
    snooping_trust: true
    vlans:
      - USERS
      - MGMT
      - GUEST

############END TRUNK PORTS#############
```

An example of the trunk ports block is as follows:

```
############BEGIN TRUNK PORTS###########

all_trunks:
  APS:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 45-46
    description: "TO AP"
    uplink: true
    admin_up: true
    native_vlan: MGMT
    vlans:
      - USERS
      - MGMT
      - GUEST
  COX-EPLAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 47
    description: "MODEM EPLAN"
    uplink: true
    admin_up: true
    vlans:
      - EPLAN
  FW-LAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 49
    description: "FIREWALL-UPLINK 1"
    uplink: true
    admin_up: true
    vlans:
      - USERS
      - MGMT
      - GUEST
      - VOICE
  FW-WAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 50
    description: "FIREWALL-DOWNLINK 2"
    uplink: true
    admin_up: true
    vlans:
      - WAN
      - EPLAN

############END TRUNK PORTS#############
```

**port_channels**: This dictionary declares the port-channels, and the associated interfaces. If this interface is an uplink towards the core, please define it as so. If the port channel is the upstream path to a DHCP server, set snooping_trust to be true.

**ID**: Integer value to identifies the port-channel number. (required)<br>
**interfaces**: List of interfaces that will be members of the port-channel. Format must match exactly of the name of the interface in the configuration (required)<br>
**description**: String or variable that will describe the interfaces. (required)<br>
**snooping_trust**: Boolean value that will permit DHCP offers on this port-channel. (optional)<br>
**uplink**: Boolean value that is not implemented, but eventually will be utilized (optional)<br>
**native_vlan**: VLAN to be used as the native, untagged VLAN. If left un-delcared, default of 1 will be used. (optional)<br>
**bpdu_guard**: Boolean value to enable BPDU guard on the interface. (optional)<br>
**vlans**: List of interfaces that will be allowed on the port-channel. Must be named exactly as the VLAN keys in the site group_vars site_vlans dictionary (required)<br>

The template for the trunk ports block is as follows:

```
###########BEGIN PORT-CHANNELS############

port_channels:
  ID:
    interfaces:
      - Interface 1
      - Interface 2
    description: "description"
    snooping_trust: true
    uplink: true
    vlans:
      - VLAN1
      - VLAN2

##########END PORT-CHANNELS############
```

An example of the trunk ports block is as follows:

```
###########BEGIN PORT-CHANNELS############

port_channels:
  1:
    interfaces:
      - GigabitEthernet1/0/49
      - GigabitEthernet1/0/50
    description: "LAN TRUNK"
    snooping_trust: true
    uplink: true
    vlans:
      - USERS
      - MGMT
      - GUEST
      - VOICE
  2:
    interfaces:
      - GigabitEthernet1/0/51
      - GigabitEthernet1/0/52
    description: "WAN TRUNK"
    snooping_trust: true
    vlans:
      - WAN
      - EPLAN

##########END PORT-CHANNELS############
```


#**Author Information**
------------------

T-Money-Mac-Network-McDizzle (Trent Nielsen) is the author of this role.
