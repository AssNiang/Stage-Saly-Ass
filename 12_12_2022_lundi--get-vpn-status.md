# I. Get VPN status
  ## 1. Playbook
We created a playbook "vpn-status-playbook.yml" to get a list of the status of every vpn in the Fortigate. This list will be printed in a txt file located 
in the desktop of the machine with awx. In this playbook, there is two plays. The first one act on the fortigate and the second one act on the machine with awx.
```yaml
- name: Get the status of VPN
  hosts: "{{node}}"
  vars:
    - down_vpn: []
    - up_vpn: []

  tasks:
    - name: Get the vpn ipsec
      fortinet.fortios.fortios_monitor_fact:
        vdom: "root"
        selector: "vpn_ipsec"
      register: vpn

    - name: Get vpn with down status
      set_fact: down_vpn="{{down_vpn + [item.name]}}"
      when: item.proxyid[0].status=="down"
      loop: "{{vpn.meta.results}}"

    - name: Get vpn with up status
      set_fact: up_vpn="{{up_vpn + [item.name]}}"
      when: item.proxyid[0].status=="up"
      loop: "{{vpn.meta.results}}"
```

```yaml
- name: Create the file
  hosts: awx_localhost
  vars:
    - up_vpn: '{{hostvars[node]["up_vpn"]}}'
    - down_vpn: '{{hostvars[node]["down_vpn"]}}'

  tasks:
    - name: Remove file (delete file)
      file:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: absent

    - name: Insert the date of access of the vpn status
      lineinfile:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: present
        create: true
        line: "File generated at {{ansible_date_time.time}} {{ansible_date_time.date}}\n\n[{{node}}]"

    - name: Hedear of down vpn
      lineinfile:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: present
        create: true
        line: "\n>> DOWN VPN:"

    - name: Write the down vpn in the txt file
      lineinfile:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: present
        create: true
        line: "\t- {{item}}"
      when: hostvars[node]["down_vpn"] is defined
      loop: "{{down_vpn}}"

    - name: Hedear of up vpn
      lineinfile:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: present
        create: true
        line: "\n>> UP VPN:"

    - name: Write the up vpn in the txt file
      lineinfile:
        path: "/home/{{awx_ui_user}}/Bureau/vpn_status.txt"
        state: present
        create: true
        line: "\t- {{item}}"
      when: hostvars[node]["up_vpn"] is defined
      loop: "{{up_vpn}}"
```
## 2. Inventory
In the inventory file, we added an ssh connection with the machine with AWX.
```yaml
awx_localhost:
            ansible_host: 192.168.32.139
            ansible_user: "root"
            ansible_connection: ssh
            ansible_ssh_pass: "04062001"
 ```
