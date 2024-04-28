# ansible-role-fortiadc-glb-host

## Description

Ansible role to setup/manage Fortinet's FortiADC GLB Host settings using their HTTP REST API.

Since FAD GLB Host have a recursive dependency (Host -> VSP -> Servers -> Data Center), this role will depends on those roles. You can skip them by using tags. See [About Tags](#about-tags) below for more details. 


## Usage

### Hosts Example

```
# /etc/ansible/hosts

[fortiadc]
fad1.ndkprd.com ansible_host=fad1.ndkprd.com fad_apitoken=mysupersecrettoken
```

### Playbook Example

```yaml
---

- name: Update/create FortiADC GLB Resources.
  hosts: all
  become: false
  gather_facts: false
  connection: local
  vars:
     # for testing-purpose only, to delete the created resources after. change to 'false' if you want the resource to stay.
    do_cleanup: true
    # global-load-balance data-center entries
    fad_glb_data_centers:
      - name: dc1.ndkprd.com
        location: ID
    # global-load-balance servers entries
    fad_glb_servers:
      - name: "dmz.dc1.ndkprd.com"
        data_center: "dc1.ndkprd.com"
        health_check_ctrl: enable
        health_check_list: "LB_HLTHCK_ICMP "
        health_check_relationship: OR
        server_type: Generic-Host
        auth_type: none
        address_type: ipv4 # FAD address type
        auto_sync: disable
        fad_ipv4: "0.0.0.0"
        fad_ipv6: "::"
        fad_pass: ""
        fad_port: "5858"
        server_members:
          - name: public-waf-1.dc1.ndkprd.com
            ipv4: 10.10.1.1
            ipv6: "::"
            address_type: ipv4
            gateway: ""
            health_check_ctrl: disable
            health_check_inherit: enable
            health_check_list: ""
            health_check_relationship: "OR"
    # global-load-balance virtual-server-pool entries
    fad_glb_vs_pools:
      - name: public-waf.dc1.ndkprd.com # VS Pools mkey
        check_server_status: enable # healthcheck
        check_virtual_server_existent: disable
        load_balance_method: wrr
        vs_pool_members:
          - id: 1001 # high number of ID for mkey
            is_backup: disable # if enable, when healthcheck failed it will goes to this server
            server: dmz.dc1.ndkprd.com # GLB Servers
            server_member_name: public-waf-1.dc1.ndkprd.com # GLB Servers member
            weight: 10

  roles:
    - ndkprd.fad_glb_host
```

### About Tags

I added quiet lots of debug task, mainly to check if the variable I set is correct. These tags basically just print out the var that the previous task set/register. You can skip them altogether by skipping tasks with `debug` tags.

For example, if you're using CLI, you can just go `ansible-playbook playbook.yaml --skip-tags debug`.

As mentioned above, this role recursively depends on my two other FortiADC GLB role: one for GLB Data Center, other for GLB Servers and its members. You can use/skip them with the following tags:

- ndkprd.fortiadc-glb-data-center -> `fad_glb_dc`;
- ndkprd.fortiadc-glb-servers -> `fad_glb_servers` and `fad_glb_servers_member`;
- ndkprd.fortiadc-glb-vsp -> `fad_glb_vs_pool` and `fad_glb_vs_pool_member`.

## License

MIT, use at your own risk.
