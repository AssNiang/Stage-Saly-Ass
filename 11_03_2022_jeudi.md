# Jeudi 3 novembre 2022
## I. Configuration d'un vpn site to site
files: 
- varsA.yml var file for site A
- varsB.yml var file for site B
- playbook vpn-setup.yml

### varsA.yml
```yaml
  vpn:
   site:
    host: 192.168.32.134
    username: admin
    password: 04062001
    vdom: root
    peerIP: 192.168.32.133
    pskey: sample
    dh: 5
    proposal: des-sha512
    phase2keylifetime: 3600
    subnets:
      site_local: Tokyo
      site_remote: Dublin
      sas: 
        - name: sa1
          local: 10.10.50.0 255.255.255.0
          remote: 10.10.40.0 255.255.255.0
```

### varsB.yml
```yaml
 vpn:
  site:
    host: 192.168.32.133
    username: admin
    password: 04062001
    vdom: root
    peerIP: 192.168.32.134
    pskey: sample
    dh: 5
    proposal: des-sha512
    phase2keylifetime: 3600
    subnets:
      site_local: Dublin
      site_remote: Tokyo
      sas: 
        - name: sa1
          local: 10.10.40.0 255.255.255.0
          remote: 10.10.50.0 255.255.255.0
```

### vpn-setup.yml
```yaml
    - hosts: fortigate-A
  gather_facts: no
  vars_files:
    - ./varsA.yml
  tasks:

  - name: VPN PHASE1
    tags: vpn1
    fortios_vpn_ipsec_phase1_interface:
      vdom:  "{{ item.value.vdom }}"
      state: "present"
      vpn_ipsec_phase1_interface:
        name: "P1vpn-to-{{item.value.subnets.site_remote}}"
        interface: "port1" 
        peertype: "any"
        proposal: "{{item.value.proposal}}"
        dpd_retryinterval: "5"
        dhgrp: "{{item.value.dh}}"
        psksecret: "{{item.value.pskey}}" 
        remote_gw: "{{item.value.peerIP}}"
    with_dict: "{{vpn}}"  
    register: result

  - set_fact: hostname="{{vpn.site.host}}"
              user="{{vpn.site.username}}"
              password="{{vpn.site.password}}"
              vdom="{{vpn.site.vdom}}"
              proposal="{{vpn.site.proposal}}"
              phase2keylifetime="{{vpn.site.phase2keylifetime}}"
              site_local_name="{{vpn.site.subnets.site_local}}"
              site_remote_name="{{vpn.site.subnets.site_remote}}"
              sas="{{vpn.site.subnets.sas}}"
    tags: route,vpn_status

  - name: VPN PHASE2
    tags: vpn1
    fortios_vpn_ipsec_phase2_interface:
      vdom:  "{{ vdom }}"
      state: "present"
      vpn_ipsec_phase2_interface:
        name: "P2vpn-to-{{site_remote_name}}-{{item.name}}"
        phase1name: "P1vpn-to-{{site_remote_name}}"
        proposal: "{{ proposal }}"
        replay: "enable"
        pfs: "enable"
        auto_negotiate: "enable"
        keylifeseconds: "{{phase2keylifetime }}"
        src_subnet: "{{item.local}}"
        dst_subnet: "{{item.remote}}" 
    with_items: "{{sas}}"
    

  - name: firewall address local
    tags: firewall_addr
    fortios_firewall_address:
      vdom:  "{{ vdom }}"
      state: "present"
      firewall_address:                                 
        name: "{{site_local_name}}-{{item.name}}"
        subnet: "{{item.local}}"
    with_items: "{{sas}}"

  - name: firewall address remote
    tags: firewall_addr
    fortios_firewall_address:
      vdom:  "{{ vdom }}"
      state: "present"
      firewall_address:
        name: "{{site_remote_name}}-{{item.name}}"
        subnet: "{{item.remote}}"
    with_items: "{{sas}}"


  - name: firewall policy outbound
    tags: firewall_out
    fortios_firewall_policy:
      vdom: "{{ vdom }}"
      state: "present"
      firewall_policy:
        policyid: "0"
        srcintf:
         - name: "port2"
        dstintf:
         - name: "P1vpn-to-{{site_remote_name}}"
        dstaddr:
         - name: "{{site_remote_name}}-{{item.name}}"
        srcaddr:
         - name: "{{site_local_name}}-{{item.name}}"
        schedule: "always"
        action: "accept"
        service:
         - name: "PING"
         - name: "SSH" 
    with_items: "{{sas}}"

  - name: firewall policy inbound
    tags: firewall_in                                                                                                                                                                    
    fortios_firewall_policy:                                                                                                                                                                                                                                                                                                                                       
      vdom: "{{ vdom }}"                                                                                                                                                                                                                                                                                                                                                         
      state: "present"                                                                                                                                                                         
      firewall_policy:                                                                                                                                                                      
        policyid: "0"                                                                                                                                                                                 
        srcintf:                                                                                                                                                                                               
         - name: "P1vpn-to-{{site_remote_name}}"                                                                                                                                
        dstintf:                                                                                                                                                                                        
         - name: "port2"                                                                                                      
        dstaddr:                                                                                                                                                                                        
         - name: "{{site_local_name}}-{{item.name}}"
        srcaddr:                                                                                                                                                                                              
         - name: "{{site_remote_name}}-{{item.name}}"
        schedule: "always"                                                                                                                                                                                 
        action: "accept"                                                                                                                                                                                 
        service:                                                                                                                                                                                                
         - name: "PING"                                                                                                                                                                                           
         - name: "SSH"                                                                                                                                                                                           
    with_items: "{{sas}}"

  - name: route through vpn tunnel
    tags: route                                                                                                                                                                                
    ignore_errors: yes
    fortios_router_static:                                                                                                                                                                     
      vdom: "{{ vdom }}"                                                                                                                                                                                                                                                                                                                                                      
      state: "present"                                                                                                                                                                                  
      router_static:
        dst: "{{item.remote}}"
        device: "P1vpn-to-{{site_remote_name}}"
        seq_num: "0" 
    with_items: "{{sas}}"
```

**Run the playbook** 
```shell
    root@Salimata-Diagana:~# ansible-playbook -i fortigate_inventory.yml vpn-setup.yml
```
## III. TO DO NEXT
- Optimiser les playbooks

## IV. Some useful links
1. [Configuration vpn site to site](https://latebits.com/2020/04/16/ansible-s2svpn-fortinet/)
2. [code github](https://github.com/czirakim/Ansible.s2sVPN.Fortinet)
