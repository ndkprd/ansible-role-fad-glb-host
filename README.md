# ansible-role-fortiadc-glb-host

## Description

Ansible role to setup/manage Fortinet's FortiADC GLB Host settings using their HTTP REST API.

## Usage

### Hosts Example

```
# /etc/ansible/hosts

[fortiadc]
fad1.ndkprd.com ansible_host=fad1.ndkprd.com fad_apitoken=mysupersecrettoken
fad2.ndkprd.com ansible_host=fad2.ndkprd.com fad_apitoken=mysupersecrettoken
```

### Needed Vars Example

```
---
# ./fad-glb-host-vars.yaml

fad_vdom: root

fad_glb_host:
  - hostname: repo # hostname without domain
    domain: devops.ndkprd.com # base domain without dot
    scope: public # for naming purpose naming only, I personally use "public" and "local"
    default_ipv4: 0.0.0.0 # default IP when none of virtual server work
    vs_pool:
      - mkey: DC1 # your identifier for the virtual server pool
        pool_name: waf.dc1.ndkprd.com # existing pool name, MUST EXIST FIRST
        weight: 100 # VS pool weight
      - mkey: DC2
        pool_name: waf.dc2.ndkprd.com
        weight: 1

  - hostname: sonarqube
    domain: devops.ndkprd.com
    scope: public
    default_ipv4: 0.0.0.0
    vs_pool:
      - mkey: DC1
        pool_name: waf.dc1.ndkprd.com
        weight: 100
      - mkey: DC2
        pool_name: waf.dc2.ndkprd.com
        weight: 1
```

### Playbook Example

```
---
# ./playbook.yaml

- name: Add global-load-balance host entry in FortiADC.      
  hosts: fortiadc
  become: true
  gather_facts: no
  vars_files:
    - ./fad-glb-host-vars.yaml

  roles:
    - ndkprd.fortiadc-glb-host
```

## License

MIT, use at your own risk.