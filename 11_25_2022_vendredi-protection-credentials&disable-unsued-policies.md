# I- Protection of the credentials

1. We removed informations about ansible_user and ansible_password written in the inventory file. 
2. To run the jobs on AWX, we created a Machine type credential with the username and password of the fortigate. 
3. Then we add this credential to the job we want to run.

# II- Disable unused policies

1. We added comments in the task of creation of policies to get the date of creation of the policy
  ``` comments: "{{ansible_date_time.epoch}}" ```
  
2. With all the statitics policies we get a list of unused policies id.
  ```yaml 
  - name: "List of never used policies"
  set_fact: never_used_policies="{{never_used_policies + [item.policyid]}}"
  when: item.last_used is undefined
  loop: "{{policy.meta.results}}"
  ```
  
3. We get the list of all unused policies
  ```yaml
  - name: Get the never used policies
  fortinet.fortios.fortios_configuration_fact:
    formatters:
      - comments
      - policyid
    filters:
      - policyid=="{{item}}"
    selector: "firewall_policy"
  register: policies
  loop: "{{never_used_policies}}"
  ```
  4. We disable them if the duration since their creation date is greater than the threshold
  ```yaml
  - name: Disable never used policies since more than "{{years}}" years "{{months}}" months "{{days}}" days "{{hours}}" hours "{{minutes}}" minutes "{{seconds}}" seconds
  fortios_firewall_policy:
    vdom: root
    state: present
    firewall_policy:
      policyid: "{{item.meta.results[0].policyid}}"
      status: disable
  when:
    - years | int + months | int + days | int + hours | int + minutes | int + seconds | int > 0
    - (ansible_date_time.epoch | int - item.meta.results[0].comments | int)  > threshold | int
  with_items: "{{policies.results}}"
  ```
