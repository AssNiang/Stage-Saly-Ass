# Mercredi 2 novembre 2022
## I. Configuration d'une interface
fichier: firewall-conf-playbook.yml
```yaml
   -   name: Configuration port3
                fortios_system_interface:
                    vdom: root
                    state: present
                    enable_log: yes
                    system_interface:
                     algorithm: L3
                     allowaccess:
                        - ping
                        - https
                        - http
                        - ssh
                     ip: 10.10.50.10 255.255.255.0
                     name: port3
                     status: up
                     type: physical
                     role: lan
                     mode: static
                     alias: LAN2
```

## II. Configuration d'une policy
fichier: firewall-conf-playbook.yml
```yaml
   -   name: Accès Internet
                fortios_firewall_policy:
                    vdom: root
                    state: present
                    firewall_policy:
                     name: access to internet
                     action: accept
                     dstaddr:
                     - name: all
                     dstintf:
                     - name: port1
                     logtraffic: utm
                     nat: enable
                     policyid: 1
                     service:
                     - name: HTTPS
                     - name: HTTP
                     - name: DNS
                     srcaddr:
                     - name: all
                     srcintf:
                     - name: port2
                     schedule: always
                     schedule_timeout: disable
```

## III. Configuration vpn site to site
  1. Configuration sur le forti 1 :
    - phase 1 interface
    - phase 2 interface
    - static route
```yaml
    - name: VPN site to site configuration for forti1
  hosts: fortinet

  gather_facts: false
  connection: httpapi

  tasks:
    - name: configure phase 1 interface
      fortios_vpn_ipsec_phase1_interface:
        state: present
        vpn_ipsec_phase1_interface:
          name: To_forti2
          interface: port1
          peertype: any
          proposal:
            - aes128-sha256 
            - aes256-sha256 
            - aes128-sha1
            - aes256-sha1
          remote_gw: 192.168.229.117
          psksecret: sample

    - name: Configure phase 2 interface
      fortios_vpn_ipsec_phase2_interface:
        state: present
        vpn_ipsec_phase2_interface:
          name: To_forti2
          phase1name: To_forti2
          proposal:
            - aes128-sha256 
            - aes256-sha256 
            - aes128-sha1
            - aes256-sha1
          auto_negotiate: enable

    - name: configure static routes
      fortios_router_static:
      state: present
      router_static:
        dst: 192.168.229.0
        device: To_forti2
```

## III. TO DO NEXT
- Continuer la configuration du vpn site to site.

## IV. Some useful links
1. [documentation ansible vpn phase 1 interface](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_vpn_ipsec_phase1_interface_module.html#ansible-collections-fortinet-fortios-fortios-vpn-ipsec-phase1-interface-module)
2. [documentation ansible vpn phase 2 interface](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_vpn_ipsec_phase2_interface_module.html#ansible-collections-fortinet-fortios-fortios-vpn-ipsec-phase2-interface-module)
3. [documentation ansible static route](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_router_static_module.html#ansible-collections-fortinet-fortios-fortios-router-static-module)
4. [Documention fortinet vpn site to site](https://docs.fortinet.com/document/fortigate/7.2.1/administration-guide/913287/basic-site-to-site-vpn-with-pre-shared-key)

