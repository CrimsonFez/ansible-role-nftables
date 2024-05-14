Ansible Role: Nftables
======================

Ansible Role that installs and manages [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page).

Ruleset is written in single file, directory `/etc/nftables` is created for manual include-files. On Debian systemd unit-file is patched to allow normal ruleset reload.

Requirements
------------

Python, Ansible >= 2.9

Role Variables
--------------

|Variable|Deafule value|Description|
|-|-|-|
|`nftables_validate_ruleset`|`True`|Wether to run `nft -c -f {{ nftables_ruleset_path }}`|
|`nftables_disable_flush_ruleset`|`False`|Disable Flush ruleset on reload/restart service, instead flush individual tables. This is usefull for when running other applications that configure nftables.|
|`nftables_ruleset.include_files`|`[]`|List of include files path for generic level include statements|
|`nftables_ruleset.tables`|`[]`|List of tables|
|`nftables_ruleset.tables[*].name`|-|Table name, mandatory|
|`nftables_ruleset.tables[*].family`|`'ip'`|Table family: ip, arp, ip6, bridge, inet, netdev|
|`nftables_ruleset.tables[*].include`|`[]`|List of inlude file paths|
|`nftables_ruleset.tables[*].chains`|`[]`|List of chains in table|
|`nftables_ruleset.tables[*].chains[*].name`|-|Chain name, mandatory|
|`nftables_ruleset.tables[*].chains[*].type`|`'filter'`|Chain type: `'filter'`, `'route'`, `'nat'`|
|`nftables_ruleset.tables[*].chains[*].hook`|`'input'`|Chain hook|
|`nftables_ruleset.tables[*].chains[*].device`|-|Device associated with chain|
|`nftables_ruleset.tables[*].chains[*].policy`|`'accept'`|Chain policy action|
|`nftables_ruleset.tables[*].chains[*].priority`|`0`|Chain priority|
|`nftables_ruleset.tables[*].chains[*].rules`|`[]`|List of rule expressions|

Dependencies
------------

None.

Supported platforms
-------------------

* Debian
  * Buster (11)
  * Bullseye (12)

Example Playbook
----------------

```yaml
---
- hosts: all
  become: yes

  vars:
    nftables_ruleset:
      tables:
        - name: filter
          family: "ip"
          chains:
            - name: input
              type: "filter"
              hook: "input"
              policy: "drop"
              priority: 0
              rules:
                - "ct state established,related accept"
                - "iifname lo accept"
                - "icmp type echo-request accept"
                - "tcp dport {ssh} accept"
            - name: forward
              type: "filter"
              hook: "forward"
              policy: "drop"
              priority: 0
            - name: output
              type: "filter"
              hook: "output"
              policy: "accept"
              priority: 0

  roles:
    nftables
```

Testing
-------

This role uses [Molecule](https://molecule.readthedocs.io/en/stable/) for testing. Molecule is run against [Vagrant](https://www.vagrantup.com/docs)-managed VMs (see `default` scenario). By default, provider `libvirt` is used unless overriden by `MOLECULE_VAGRANT_PROVIDER` env variable.

Use `MOLECULE_VAGRANT_VM_CPUS` and `MOLECULE_VAGRANT_VM_MEM` env variables to set VM vCPUs and RAM count.

Dependencies:
* molecule
* [molecule-plugins[vagrant]](https://github.com/ansible-community/molecule-plugins)
* ansible

### Quick Install with pipenv
```shell
pipenv install
```

### Running with pipenv
```shell
pipenv shell
molecule test
```

License
-------

SPDX:MIT  
See full text in [LICENSE](LICENSE) file.

Author Information
------------------

This fork is maintained by David Kovari.  
This role was created in 2021 by Dmitry Danilov.
