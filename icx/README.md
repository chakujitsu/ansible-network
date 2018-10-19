
This code add a new network_os type for connecting to Ruckus ICX Fastiron OS devices, along with two fastiron modules, fastiron_command and fastiron_config.

Tested on Ansible version 2.7 against Fastiron 8.0.7b.

The connection type required is network_cli with the specified network os of "fastiron"

This was all cloned and copied from the Ansible approved module "ironware", which was formerly of Brocade, and has the same SSH connection handler with the exception of slight differences, such as the method in which paging is disabled.

network_cli usage example:
```
- name: Test ICX Commands
  hosts:
    - lab-icx
  connection: network_cli
  gather_facts: no
  vars:
    ansible_user: username
    ansible_ssh_pass: password
    ansible_network_os: fastiron
  roles:
    - icx-testing
```

fastiron_command:
Used to send a command to the device and capture the output.

Example to get the output of the 'show version' command:
```
- name: Send fastiron Command
  fastiron_command:
    commands:
      - show version
  register: results
```

fastiron_config:
Used to send configuration terminal commands to the device.

Example to configure the hostname:
```
- name: Configure hostname
  fastiron_config:
    config:
      - hostname {{ hostname }}
```

The following files were used below:

Ironware Connection handlers:
/ansible/plugins/terminal/ironware.py
/ansible/plugins/action/ironware.py
/ansible/plugins/cliconf/ironware.py

Ironware module_util folder:
/ansible/module_utils/network/ironware/__init__.py
/ansible/module_utils/network/ironware/ironware.py


Ironware module folder:
/ansible/modules/network/ironware/__init__.py
/ansible/modules/network/ironware/ironware_command.py
/ansible/modules/network/ironware/ironware_config.py
/ansible/modules/network/ironware/ironware_facts.py

The above were cloned to:
Fastiron Connection handlers:
/ansible/plugins/terminal/fastiron.py
/ansible/plugins/action/fastiron.py
/ansible/plugins/cliconf/fastiron.py

Fastiron module_util folder:
/ansible/module_utils/network/fastiron/__init__.py
/ansible/module_utils/network/fastiron/fastiron.py

Fastiron module folder:
/ansible/modules/network/fastiron/__init__.py
/ansible/modules/network/fastiron/fastiron_command.py
/ansible/modules/network/fastiron/fastiron_config.py

Installation:
# Copy cli, terminal and action files
cp ./build/icx/terminal/fastiron.py /ansible/plugins/terminal/fastiron.py
cp ./build/icx/action/fastiron.py /ansible/plugins/action/fastiron.py
cp ./build/icx/cliconf/fastiron.py /ansible/plugins/cliconf/fastiron.py

# Copy module_utility files
mkdir -p /ansible/module_utils/network/fastiron/
cp ./build/icx/module_utils/fastiron/* /ansible/module_utils/network/fastiron/

# Copy modules to custom path
mkdir -p path-to-custom-modules/plugins/modules/fastiron/
cp ./build/icx/modules/fastiron/* path-to-custom-modules/plugins/modules/fastiron/
