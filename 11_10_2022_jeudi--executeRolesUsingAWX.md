# Thursday, November 10th, 2022
## I. Create the github credential
1. Generate a ssh key in the host (with the ansible server)
```shell
    assniang@ASS-NIANG: ssh-keygen
```
2. Copy the ssh key after printing it on the terminal
```shell
    assniang@ASS-NIANG: cat ~/.ssh/id_rsa
```
3. Create new credential in awx ui
- Credential Type = Source Control
- Paste the ssh-key in the SCM Private Key field
    
4. Create a new ssh-key in github
```shell
    assniang@ASS-NIANG: cat ~/.ssh/id_rsa.pub
```
- Copy the ssh-key
- Go to settings/SSH and GPG keys in github
- Create a new ssh-key.
- Key type : Authentication key
- Paste the ssh-key

## II. Create a project in awx ui
- Source Control Type : Git
- Paste the ssh-url of the project
- Add the credential you've just created
- Check Update Revision on Lauch
- Wait for synchronization

## III. Create an inventory
- Go to add inventory, give a name and save it.
- In the inventory you've just created, go to souces and add a new source.
- Source = Sourced from a Project.
- Choose the project .
- Inventory file = /(project root)
- You can modify the verbosity.
- Check update on launch and save.
- Synchronized the project and the sources

### IV. Create a template
- Go to add template and create a job.

 # WARNING
 Add a requirements.yml file to include the collections.
 
 ```yaml 
    collections:
        -   name: fortinet.fortios
```




## III. TO DO NEXT
- Update the `firewall-policy-playbook.yml` file to set some firewall policies using ansible

## IV. Some useful links
1. [SSH your fortigate](https://www.youtube.com/watch?v=CB2lv4ebBJg)
2. [Forigate: Introduction à l'automatisation avec ansible](https://www.youtube.com/watch?v=U5Y7_VIe6fs&t=151s)
3. [fortinet.fortios.fortios_system_global module – Configure global attributes in Fortinet’s FortiOS and FortiGate.](https://docs.ansible.com/ansible/latest/collections/fortinet/fortios/fortios_system_global_module.html#ansible-collections-fortinet-fortios-fortios-system-global-module)
