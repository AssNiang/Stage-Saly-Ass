# Wednesday, November 17th, 2022
## I. Create the role config-vlan
1. Task/main.yml
```yaml
    ---
# tasks file for config-vlan
- name: "Configuration vlan"
  fortios_system_interface:
    vdom: root
    state: present
    #enable_log: true
    system_interface:
      name: "{{ nom }}"
      vdom: "root"
      algorithm: L3
      allowaccess: "{{ allowaccess }}"
      ip: "{{ adress_ip }}"
      interface: "{{ port_name }}"
      status: up
      type: vlan
      role: "{{ role }}"
      mode: "{{ mode }}"
      alias: "{{ alias }}"

- name: Configuration du DHCP
  fortios_system_dhcp_server:
    vdom: root
    state: present
    system_dhcp_server:
      auto_configuration: enable
      auto_managed_status: enable
      conflicted_ip_timeout: 1800
      default_gateway: "{{ default_gateway }}"
      netmask: "{{netmask}}"
      dns_server1: 8.8.8.8
      dns_server2: 8.8.4.4
      dns_server3: "{{ end_ip }}"
      dns_service: default
      id: "{{id}}"
      interface: "{{ port_name }}"
      ip_mode: range
      ip_range:
        - end_ip: "{{ end_ip }}"
          id: "{{ip_range_id}}"
          start_ip: "{{ start_ip }}"
      lease_time: 6048000
  when: configure_dhcp is true

```
2. Vars/main.yml
```yaml
    ---
# vars file for config-vlan
nom: VLAN-0
allowaccess:
  - http
adress_ip: 10.10.60.1 255.255.255.0
port_name: port2
role: lan
mode: static
alias: VLAN-0
# Config dhcp
# if configure_dhcp is false all the following lines are ignored
configure_dhcp: false
id: 2
default_gateway: 10.10.40.1
netmask: 255.255.255.0
end_ip: 10.10.40.254
ip_range_id: 1
start_ip: 10.10.40.2
```

3- Add the vars in awx

## II. TO DO NEXT

## IV. Some useful links
1. [fortinet.fortios.fortios_system_interface module – Configure vlan in Fortinet’s FortiOS and FortiGate.]([https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_system_global_module.html#ansible-collections-fortinet-fortios-fortios-system-global-module](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_system_interface_module.html#ansible-collections-fortinet-fortios-fortios-system-interface-module))
