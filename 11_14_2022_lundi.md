# Monday, November 14th, 2022
## I. Optimizations
1. Vpn_site_to_site
  - Deleted : 
      - the include_vars in the task folder and replaced with
        ```yaml
          - set_fact: vpn="{{vpn_fortigate1}}"
            when:
              - inventory_hostname == fortigate1
              - vpn_fortigate1 is defined

          - set_fact: vpn="{{vpn_fortigate2}}"
            when:
              - inventory_hostname == fortigate2
              - vpn_fortigate2 is defined
        ```
      - varsA.yml file
      - varsB.yml file
  - Updated the main.yml file in vars folder
    ```yaml
        vpn_fortigate1:
        site:
            host: 192.168.32.134
            vdom: root
            peerIP: 192.168.32.133
            pskey: sample
            dh: 5
            proposal: des-sha512
            phase2keylifetime: 3600
            subnets:
              site_local: Local
              site_remote: Remote
              sas:
                - name: sa1
                  local: 10.10.50.0 255.255.255.0
                  remote: 10.10.40.0 255.255.255.0
            outbound_policy_id: 5
            inbound_policy_id: 7
            seq_num_route_static: 1

        vpn_fortigate2:
          site:
            host: 192.168.32.133
            vdom: root
            peerIP: 192.168.32.134
            pskey: sample
            dh: 5
            proposal: des-sha512
            phase2keylifetime: 3600
            subnets:
              site_local: Local
              site_remote: Remote
              sas:
                - name: sa1
                  local: 10.10.40.0 255.255.255.0
                  remote: 10.10.50.0 255.255.255.0
            outbound_policy_id: 5
            inbound_policy_id: 7
            seq_num_route_static: 1
    ```

  - Add the content in the extra-vars field in AWX
  
2. Config-policy

- Add the creation of: 
    - a schedule
    - addresses
 
 `vars/main.yml`

 ```yaml
  # Create a schedule
  # if create_new_schedule is false all the following lines are ignored
  create_new_schedule: false
  schedule_name: test
  start_date: 10:25 2022/11/14
  end_date: 10:40 2022/11/14
  expiration_days: 0

  # Create addresses
  # if create_new_address is false all the following lines are ignored
  create_new_address: false
  addresses:
    - address_name: test_address
      address_subnet: 10.10.20.2/32
      address_interface: any
    # You can add/create others addresses in the list
 ```
 `tasks/main.yaml`

 ```yaml
  # Create a schedule
  - name: Onetime schedule configuration.
    fortios_firewall_schedule_onetime:
      vdom: "root"
      state: "present"
      firewall_schedule_onetime:
        name: "{{schedule_name}}"
        start: "{{start_date}}"
        end: "{{end_date}}"
        expiration_days: "{{ expiration_days }}"
    when: create_new_schedule is true

  - set_fact: schedule="{{schedule_name}}"
    when: create_new_schedule is true

  # Create an address
  - name: Configure IPv4 addresses.
    fortios_firewall_address:
      vdom: "root"
      state: "present"
      firewall_address:
        name: "{{ item.address_name }}"
        subnet: "{{ item.address_subnet}}"
        interface: "{{ item.address_interface }}"
    with_items: "{{ addresses }}"
    when: create_new_address is true

 ```

## II. TO DO NEXT

## III. Some useful links
1. [code github](https://github.com/SalyDgn/fortigate-ansible)
2. [Forigate ansible schedule module documentation](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_firewall_schedule_onetime_module.html#ansible-collections-fortinet-fortios-fortios-firewall-schedule-onetime-modules)
3. [fortigate ansible address module documentation](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_firewall_address_module.html#ansible-collections-fortinet-fortios-fortios-firewall-address-module)
