network-baseline-config
=========

Note: This deployment model is utilized in conjunction with Jenkins running the jobs via a container in which is built by the Ansible repo.

This role is used to generate a baseline configuration file to configure a new device for field deployment. This role utilize a Jinja2 templates, which are filled by variables defined by host and group variable files. This role is run locally on the machine in which the it was executed to generate an initial baseline config, and then stored into the ansible/configs/ directory. Depending on the vendor, after production deployment, these variable files are used to enforce a desired state by other roles specific to that platform. (See roles prepended with IOS for examples of enforcing a desired state).

Currently, this role is used to generate configuration baselines for the following:

- Ruckus ICX Edge Switch
- Ruckus ICX Core Switch/Layer 3
- Cisco 2960-x Edge Switch
- Cisco 3750-x Edge Switch
- Fortigate 140E for retail branch architecture


# Simplified Steps
------------

1. Clone the Ansible repo to your local machine. Create a site group_vars file if it doesn't exist for the site which you are generating the config for. You can clone the site_vars template file located at/ansible/environments/network/group_vars/template for a baseline of all required parameters. This file must be named after the site abbreviation, and is a text file with no extension. Populate all the variables in the group_vars file you just created. Examples can are shown in document below.  <br>
```
vi /ansible/environments/network/group_vars/sitename
```
2. Create device host_var file. You can clone the host_vars template file located at /ansible/environments/network/host_vars/template. This must be named after a resolvable DNS entry (cannot be internal). Populate all the variables in the header, and desired blocks (access, trunks, and port-channels).  <br>
```
vi /ansible/environments/network/host_vars/device.site.corp.lab.com
```
3. Add your group and host to the inventory file. The group name will be exactly the same name as your group_vars file, and your hosts in that group will also be the exact same name as outlined in step 2. <br>
```
vi /ansible/environments/network/network-inventory
```
4. After your files are created and populated. See the documentation below for all variables that can be configured for configuration generation.  After variables have been populated and saved, add and commit your new/modified files to the LAB Ansible git repository. You will then push your files to Github to have Jenkins generate the configurations and automatically. (if using Jenkins for CI/CD)<br>
```
git add -A
git commit -m 'commit description'
git push
```
5. Jenkins will need to be manually told to generate the configurations after the new Ansible container is built. You can run the job manually in Jenkins, titled 'ansible-playbook-network-baseline-config' or you choose to run playbook from your machine. To do so, issue the following command: <br>
```
/ansible/ansible-playbook -i ./environments/network/ ./playbooks/network/network-baseline-config.yml
```

6. If you are having Jenkins generate the configuration, you can configure the Jenkins job to push your newly generated configurations to your Git repo. You can navigate to the repo, and view your configuration in the /ansible/configs/ folder. <br>


# Requirements
------------

The variable file creation is required as below:

Host declared in the 'network-inventory file', under the site abbreviation. In the below example, usw31-01.lab.corp.lab.com was added to the network-invetory file under the group name YGW. A group_vars file is required for the site, as well as the host_vars file for the device you are generating a configuration for.

```
---
# Network Inventory
network-all:
  children:
    labdc1:
      hosts:
        avrack.labdc1.corp.lab.com:
        core.labdc1.corp.lab.com:
        ntnx.labdc1.corp.lab.com:
        usw11-01.labdc1.corp.lab.com:
        usw11-02.labdc1.corp.lab.com:
        usw11-03.labdc1.corp.lab.com:
        usw11-04.labdc1.corp.lab.com:
        usw11-05.labdc1.corp.lab.com:
        usw12-01.labdc1.corp.lab.com:
    labdc2:
      hosts:
        core.labdc2.corp.lab.com:
        ntnx.labdc2.corp.lab.com:
        tor02.labdc2.corp.lab.com:
        tor03.labdc2.corp.lab.com:
        tor06-07.labdc2.corp.lab.com:
        tor09-10.labdc2.corp.lab.com:
        usw1.labdc2.corp.lab.com:
        usw2.labdc2.corp.lab.com:
        usw3.labdc2.corp.lab.com:
        usw4.labdc2.corp.lab.com:
        theater.labdc2.corp.lab.com:
    lab:
      hosts:
        madeupswitch.lab.com:
        #10.8.20.237:
```

**host_vars file**: This is the name of the device you are provisioning, which was declared in the inventory file. The filename is hostname, and this file is used to generate the baseline and manage the device after deployment. Because Ansible is ran from the aws.lab.com domain, a seperate (non .internal) hostname must resolve to the device, because internal addresses are not forwardable between domains. The naming standard for the host_var file and DNS entry are as follows: devicename.site.corp.lab.com. This file contains variables that declare configurations that are unique to the device (such as hostname, interface configuration and so forth).


**group_vars file**: This file will be named as the site abbreviate after the site in which devices are located. This file contains variables that are unique to the site.

```
├── environments
│   ├── network
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   ├── labdc1
│   │   │   ├── labdc2
│   │   │   ├── lab
│   │   │   └── network-all
```

# Role Variables
--------------

## **all:**

configs_dir: This variable dictates where the role will save a generated configuration. If running from LAB, the assigned value will work. If running from a directory other than the declared variable, this will need to be changed to suit your setup. There is a commented out variable that I (Trent) use for the path where I keep my repositories locally on my machine. Because this role is all performed locally, you may need to adjust this path to your current Ansible directory path if running locally. If being executed in Jenkins, you can leave the path as it is.

## **group_vars:**

The group_vars site file is used to declare site specific variables. Any VLANs that are used at this site are declared here, along with other variables that are used in other roles.

**site_name**: The name of the site, as referenced by LAB IT. (required) <br>
**site_abbrv:** Abbreviated site name, used to generate variables that has a character limit (required) <br>
**site_supernet**: (Fortigate Only) String Value of the site subnet that is local to the device. Required if EPLAN == True. This value will be used in the OSPF statement if utilizing EPLAN.  <br>
**site_supernet_mask**: (Fortigate Only) String Value of the site subnet mask that is local to the device. Required if EPLAN == True. This value will be used in the OSPF statement if utilizing EPLAN. <br>
**primary_dns**: Primary DNS server for the switch to use. If left undeclared, network-all values are used. (optional in site group_var file) <br>
**secondary_dns**: Secondary DNS server for the switch to use. If left undeclared, network-all values are used. (optional in site group_var file) <br>
**mgmt_default_gateway**: IP address for all management devices at this site to use on the Management VLAN. (required) <br>
**yit_radius_servers**: Radius servers the switch uses for AAA. If left undeclared, network-all values are used. (optional in site group_var file)<br>
**site_vlans**: This dictionary declares the VLANS by name, which is in turned use to configure the VLAN IDs, name, whether or not to tag with voice, and whether or not to enable DHCP snooping on. VLANs must be declared here, in order to declare access interfaces. <br>
**dictionary-key**: VLAN name. This key is referenced in the host_vars file by name. Please declare in all caps (required) <br>
  **vlan_id**: String value indicating the VLAN number (required) <br>
  **voice_vlan**: Boolean value that will tag access interfaces with the voice VLAN (optional) <br>
  **dhcp_snoop**: Boolean value that will enable DHCP snooping on the VLAN in which it is true (optional) <br>
  **ve_ip**: (OPTIONAL; ICX/Fortigate Only) Creates a VE interface with the specified IP on a layer 3 ICX switch <br>
  **ve_mask**: (OPTIONAL; ICX/Fortigate Only) Creates a VE interface with the specified netmask on a layer 3 ICX switch <br>
  **ve_helper1**: (OPTIONAL; ICX/Fortigate Only) Creates a VE interface with the specified primary IP helper on a layer 3 ICX switch <br>
  **ve_helper2**: (OPTIONAL; ICX Only) Creates a VE interface with the specified secondary IP helper on a layer 3 ICX switch <br>
  **WAN**: (Fortigate Only) Boolean value to indicate if this VLAN is used for a WAN Interface. <br>
    **admin_distance**: (Fortigate Only) Integer value to indicate preference of default route. Required when WAN == True, and required regardless of number of WAN circuits.  <br>
  **ve_gw**: (Fortigate Only) String value indicating the associated default gateway with this WAN circuit. Required when WAN == True. <br>
  **EPLAN**: (Fortigate Only) Boolean value to indicate if this VLAN is used to uplink to the LAB EPLAN. This will configure OSPF and backup default routes for internet traffic in the event of a WAN failure.  <br>

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

An example of the site group_vars file for a location with only switches configured by Ansible is as follows:

```
---
site_name: "LAB REMOTE SITE"
site_abbrv: "lab"
primary_dns: "10.9.21.161"
secondary_dns: "10.11.21.161"
mgmt_default_gateway: "10.14.21.254"
yit_radius_servers:
  - 10.8.23.31
  - 10.11.21.31
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

An example of the site group_vars file for a site with ICX Layer 3 switches:

```
---
site_name: "LAB REMOTE SITE"
site_abbrv: "lab"
primary_dns: "10.30.21.161"
secondary_dns: "10.9.21.161"
yit_radius_servers:
  - 10.30.21.21
  - 10.8.23.31
site_vlans:
  USERS:
    vlan_id: "300"
    voice_vlan: true
    ve_ip: "10.30.0.254"
    ve_mask: "255.255.254.0"
    ve_helper1: "10.30.21.161"
    ve_helper2: "10.9.21.161"
  VOICE:
    vlan_id: "314"
    ve_ip: "10.30.14.254"
    ve_mask: "255.255.254.0"
    ve_helper1: "10.30.21.161"
    ve_helper2: "10.9.21.161"
  GUEST:
    vlan_id: "310"
  MGMT:
    vlan_id: "321"
    ve_ip: "10.30.20.254"
    ve_mask: "255.255.254.0"
    ve_helper1: "10.30.21.161"
    ve_helper2: "10.9.21.161"
...
```
An example of the site group_vars file for a retail branch with a FGT140E:

```
---
site_name: "LAB-RETAIL-EXAMPLE"
site_abbrv: "LRE"
site_supernet: '10.32.0.0'
site_supernet_mask: '255.255.248.0'
primary_dns: "10.8.23.161"
secondary_dns: "10.9.21.161"
yit_radius_servers:
  - 10.8.23.31
  - 10.11.21.31
site_vlans:
  USERS:
    vlan_id: "100"
    voice_vlan: true
    ve_ip: "10.32.0.254"
    ve_mask: "255.255.255.0"
    ve_helper1: "10.9.21.161"
  VOICE:
    vlan_id: "101"
    ve_ip: "10.32.1.254"
    ve_mask: "255.255.255.0"
    ve_helper1: "10.9.21.161"
  GUEST:
    vlan_id: "102"
    ve_ip: "10.32.2.254"
    ve_mask: "255.255.255.0"
  MGMT:
    vlan_id: "103"
    ve_ip: "10.32.3.254"
    ve_mask: "255.255.255.0"
    ve_helper1: "10.9.21.161"
  SECURITY:
    vlan_id: "104"
  AV:
    vlan_id: "105"
  CL-WAN:
    vlan_id: "200"
    WAN: True
    ve_ip: "1.1.1.2"
    ve_mask: "255.255.255.0"
    ve_gw: "1.1.1.1"
    admin_distance: 10
  ATT-WAN:
    vlan_id: "201"
    WAN: True
    ve_ip: "2.2.2.2"
    ve_mask: "255.255.255.0"
    ve_gw: "2.2.2.1"
    admin_distance: 15
  OSPF-A0:
    vlan_id: "999"
    EPLAN: True
    ve_ip: "10.255.21.32"
    ve_mask: "255.255.254.0"
...

```

## **host_vars:**

**hostname**: String that is the device hostname (required) <br>
**mgmt_vlan_ip**: String that is the device IP address (required) <br>
**mgmt_vlan_mask**: String that is the device subnet mask (required) <br>
**mgmt_default_gateway**: String that is the device's default gateway <br>
**mgmt_layer3**: Boolean value for ICX switches which enables the usage of VE interfaces <br>
**ztp_mac**: MAC address of an ICX switch for Zero Touch Provisioning
**ztp_prefix**: Hardware model of an ICX switch for Zero Touch Provisioning
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
#usw31-01.lab.lab.internal variables
hostname: "usw31-01.lab"
#Management SVI ip address
mgmt_vlan_ip: "10.14.21.253"
mgmt_vlan_mask: "255.255.255.0"
model: "2960-x"
```

An example for the host_vars file header for a Layer 3 ICX switch with Zero Touch Provisioning:

```
---
hostname: "usw11-01.lab.lab"
ztp_mac: "d4c1.9e49.f594"
ztp_prefix: "ICX7150-48ZP"
mgmt_vlan_ip: "10.30.20.249"
mgmt_vlan_mask: "255.255.254.0"
mgmt_default_gateway: "10.30.20.254"
mgmt_layer3: true
model: "icx"
```

**all_ports**: This dictionary declares the access interfaces for each VLAN declared at the site. Each VLAN is declared in in group_vars dictionary called "site_vlans". In order to add a VLAN configured for an access interface, it must be declared at the group level. After deployment into production, changing the admin_up boolean to false will turn the port down. Custom descriptions can be configured here as well, however any non unique description is encouraged to use the descriptions declared in the 'all' variable file.

**VLAN**: Key that must match a VLAN that is declared in the site group_vars file.
**name**: Interface type  (required) <br>
**module:** Interface Module/Switch (optional for Cisco, required for ICX) <br>
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
    description: "LAB-EPLAN"
    admin_up: true
  WAN:
    name: GigabitEthernet
    module: 1
    card: 0
    ports:
      - 48
    description: "COX-WAN"
    admin_up: true

#############END USER PORTS############
```

**all_trunks**: This dictionary declares the trunk interfaces on the device. Multiple trunks can be defined within this dictionary.

The "TRUNK-IDENTIFIER" key is used to identify what the trunk is configured for. (Required)<br>
**name**: Interface type (required) <br>
**module:** Interface Module/Switch (optional for Cisco, required for ICX) <br>
**card**: Interface card (required) <br>
**ports**: Interfaces to be configured. Acceptable input is range(ex. 45-47), list (ex.45,47) or singular(ex. 45). Multiple Values can be input in a new line, see the example below the template.  (required) <br>
description: Interface description. Acceptable input is a string (ex. "description") or a variable (ex. {{ user_access_port_description }}). It is preferred that the string variable follows a format that is consistent with the device it is uplinked to. That format is <connected device> <connected device interface>. For example, "core.labdc2 Te1/1/1" (required) <br>
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
    description: "TO RUCKUS AP"
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
    description: "COX MODEM EPLAN"
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
    description: "fw.lab 1"
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
    description: "fw.lab 2"
    uplink: true
    admin_up: true
    vlans:
      - WAN
      - EPLAN

############END TRUNK PORTS#############
```

**port_channels**: This dictionary declares the port-channels, and the associated interfaces. If this interface is an uplink towards the core, please define it as so. If the port channel is the upstream path to a DHCP server, set snooping_trust to be true.

**ID**: Integer value to identifies the port-channel number. (required)<br>
**lag_name:** String value that is used to name the LAG if the device type is ICX. (required for ICX),<br>
**name:** Interface type <br>
**module:** Interface Module/Switch (optional for Cisco, required for ICX) <br>
**card**: Interface card (required) <br>
**ports**: List of interfaces that will be members of the port-channel. Format must match exactly of the name of the interface in the configuration (required)<br>
**description**: String or variable that will describe the interfaces. (required)<br>
**snooping_trust**: Boolean value that will permit DHCP offers on this port-channel. (optional)<br>
**uplink**: Boolean value that is not implemented, but eventually will be utilized (optional)<br>
**native_vlan**: VLAN to be used as the native, untagged VLAN. If left un-delcared, default of 1 will be used. (optional)<br>
**bpdu_guard**: Boolean value to enable BPDU guard on the interface. (optional)<br>
**vlans**: List of interfaces that will be allowed on the port-channel. Must be named exactly as the VLAN keys in the site group_vars site_vlans dictionary (required)<br>

The template for the port channels block is as follows:

```
###########BEGIN PORT-CHANNELS############

port_channels:
  1:
    lag_name: "LAG-NAME"
    name: GigabitEthernet
    module: 1
    card: 1
    ports:
      - 1-4
    description: "Core Uplink"
    snooping_trust: true
    uplink: true
    admin_up: true
    vlans:
      - VLAN1
      - VLAN2

###########END PORT-CHANNELS############
```

An example of the port-channels block is as follows:

```
###########BEGIN PORT-CHANNELS############
#Port Channel
port_channels:
  1:
    lag_name: Core-lag3
    name: Ethernet
    module: 1
    card: 2
    ports:
      - 1-4
    description: "Core Uplink"
    snooping_trust: true
    uplink: true
    admin_up: true
    vlans:
      - USERS
      - VOICE
      - MGMT
      - GUEST
###########END PORT-CHANNELS############

```


#**Author Information**
------------------

T-Money-Mac-Network-McDizzle (Trent Nielsen) is the author of this role.
