# Monday, October 31st, 2022
## I. Connect to the fortigate using ssh
1. Generate a ssh key in the host (with the ansible server)
```shell
    assniang@ASS-NIANG: ssh-keygen -t rsa -b 2048
```
2. Copy the ssh key after printing it on the terminal
```shell
    assniang@ASS-NIANG: cat ~/.ssh/id_rsa.pub
```
3. Create a new admin in the fortigate (with super-admin priveleges)
    > System > Administrators > Create New
4. Open the fortigate CLI and set the ssh key (copied from the host terminal with the ansible server)
```shell
    forti # System config admin
    forti (admin) # edit <name-of-the-superadmin-created>
    forti (<name-of-the-superadmin-created>) # set ssh-public-key1 " <ssh-key-copied> "
    forti (<name-of-the-superadmin-created>) # end
```
5. Back to the terminal, connect to the fortigate using ssh
```shell
    assniang@ASS-NIANG: ssh <name-of-the-superadmin-created>@<ip-address-fortigate-wan>
```

## II. Some basic global configs using ansible
1. Inventory file: (ex: `fortigate_inventory.yml`)
```yaml
    fortinet:
        hosts:
            fortigate-A:
                ansible_host: 192.168.176.137
                ansible_user: "ass"
                ansible_password: "ass"
                vdom: "root"
                ansible_connection: httpapi
                ansible_httpapi_port: 443
                ansible_httpapi_use_ssl: true
                ansible_httpapi_validate_certs: false
                ansible_network_os: fortios
                ssl_verify: "False"
```
2. Playbook file: (ex: `firewall-policy-playbook.yml`)
```yaml
    ---
    -   name: CONF DE BASE FORTIGATE
        hosts: fortinet

        gather_facts: false
        connection: httpapi

        tasks:
            # task 01
            -   name: Configuration globale des fortigates 
                fortios_system_global:
                    vdom: root
                    system_global:
                        admin_concurrent: enable
                        admin_console_timeout: 300
                        admin_https_redirect: enable
                        admin_login_max: 100
                        admin_maintainer: enable
                        admin_port: 80
                        admin_restrict_local: disable
                        admin_scp: disable
                        admin_server_cert: Fortinet_GUI_Server
                        admin_sport: 443
                        admin_ssh_grace_time: 120
                        admin_ssh_password: enable
                        admin_ssh_port: 22
                        admin_telnet: enable
                        admin_telnet_port: 23
                        admintimeout: 120
                        gui_date_format: dd/MM/yyyy
                        gui_date_time_source: system
                        #post_login_banner: enable
                        hostname: "{{ inventory_hostname }}"
                        timezone: 80
                        language: english
                        gui_firmware_upgrade_warning: enable
```
3. Run the playbook in the terminal:
```shell
    assniang@ASS-NIANG:~/ansible-playbooks$ ansible-playbook -i fortigate_inventory.yml firewall-policy-playbook.yml
```

## III. TO DO NEXT
- Update the `firewall-policy-playbook.yml` file to set some firewall policies using ansible

## IV. Some useful links
1. [SSH your fortigate](https://www.youtube.com/watch?v=CB2lv4ebBJg)
2. [Forigate: Introduction ?? l'automatisation avec ansible](https://www.youtube.com/watch?v=U5Y7_VIe6fs&t=151s)
3. [fortinet.fortios.fortios_system_global module ??? Configure global attributes in Fortinet???s FortiOS and FortiGate.](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_system_global_module.html#ansible-collections-fortinet-fortios-fortios-system-global-module)
