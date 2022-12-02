# I. Création du rôle Enable Policies
Le rôle Enable policies permettra de réactiver une liste de policies donnée en variable. 
> tasks/main.yml </br>
On récupère d'abord les policies de la liste.
```yaml
- name: filter the policy name
  fortinet.fortios.fortios_configuration_fact:
    vdom: "root"
    filters:
      - status=="disable"
      - name=="{{item}}"
    formatters:
      - name
      - policyid
    selector: "firewall_policy"
  register: policies
  loop: "{{policies_name}}"
```
Si le nom d'une policy dans la liste ne correspond pas à une policy existante on fait échouer le playbook
```yaml 
- name: fail when the given policy name is not found
  ansible.builtin.fail:
    msg: There is no disabled policy named "{{item.item}}".
  when: item.meta.matched_count | int == 0
  loop: "{{policies.results}}"
  ```
Sinon on réactive les policies une à une
```yaml
- name: Enable the policies
  fortios_firewall_policy:
    vdom: root
    state: present
    firewall_policy:
      policyid: "{{item.meta.results[0].policyid}}"
      status: enable
  with_items: "{{policies.results}}"
  ```
  
  # II. Modification du rôle Disable policy
  On a modifié le rôle Disable Policy en s'inspirant du rôle Enable policy pour pouvoir supprimer une liste de policies donnée en variable.
