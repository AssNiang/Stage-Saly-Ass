# I. Disable a policy
We created a role to permit to disable automatically unused policies. 
The user specifies the threshold duration for a policy to be diabled in variables. <br/>
`vars/main.yml`
```yaml
years: 0
months: 0
days: 0
hours: 0
minutes: 0
secondes: 0
```

We disable the policies who had'nt been used or less time than the threshold duration. <br/>
`tasks/main.yml`
```yaml
- name: fact gathering
  fortinet.fortios.fortios_monitor_fact:
    selector: "firewall_policy"
  register: policy

- name: Disable a policy
  fortios_firewall_policy:
    vdom: root
    state: present
    firewall_policy:
      policyid: "{{item.policyid}}"
      status: disable
  when:
    - item.policyid | int > 0
    - years | int + months | int + days | int + hours | int + minutes | int + secondes | int > 0
    - item.last_used is undefined or (ansible_date_time.epoch | int - item.last_used | int)  > (years | int * 3600*24*365 + months | int * 3600*24*30 + days | int * 3600*24 + hours | int * 3600 + minutes | int * 60 + secondes | int)
  with_items: "{{policy.meta.results}}"
```
# I. Duplication of policies

To avoid the duplication of policies we prevent the creation of a policy if the name of the policy already exists.
```yaml
- name: filter the policy name
  fortinet.fortios.fortios_configuration_fact:
    vdom: "root"
    filters:
      - name=="{{policy_name}}"
    selector: "firewall_policy"
  register: policy

- name: Fail
  ansible.builtin.fail:
    msg: Policy name already exists.
  when: policy.meta.results[0] is defined
```
