# I. Demo awx fortigate
[Demo awx fortigate](https://drive.google.com/file/d/1FUU8FkDPaGYU5HBPvgW9ncCbqHYROpdj/view?usp=share_link)

# II. IDs gestion
1. Delete a policy </br>
We modified the "delete-policy" role to be able to delete a policy based on the name of the policy instead of the id.
```yaml
- name: filter the policy id
  fortinet.fortios.fortios_configuration_fact:
    vdom: "root"
    filters:
      - name=="{{policyname}}"
    selector: "firewall_policy"
  register: policy

- name: Fail
  ansible.builtin.fail:
    msg: Policy not found. Please verify the name.
  when: policy.meta.results[0] is undefined

- set_fact: policyid="{{policy.meta.results[0].policyid}}"

- name: "delete policy"
  fortios_firewall_policy:
    vdom: root
    state: absent
    firewall_policy:
      policyid: "{{policyid}}"
```
2. Configure a policy </br>

We modified the "config-policy" and "vpn_site_to_site" roles to automatically give an id to a policy.
```yaml
- name: fetch all policies
  fortinet.fortios.fortios_configuration_fact:
    sorters: policyid
    formatters: policyid
    selector: firewall_policy
  register: policies

- set_fact: policyid="{{policies.meta.results[-1].policyid | int + 1}}"
  when: policies.meta.results[-1].policyid is defined

- set_fact: policyid=1
  when: policies.meta.results[-1].policyid is undefined
```

The policies for the vpn
```yaml
- name: fetch all policies
  fortinet.fortios.fortios_configuration_fact:
    sorters: policyid
    formatters: policyid
    selector: firewall_policy
  register: policies

- set_fact:
    outbound_policy_id="{{policies.meta.results[-1].policyid | int + 1}}"
    inbound_policy_id="{{policies.meta.results[-1].policyid | int + 2}}"
  when: policies.meta.results[-1].policyid is defined

- set_fact: outbound_policy_id=1
    inbound_policy_id=2
  when: policies.meta.results[-1].policyid is undefined
```
