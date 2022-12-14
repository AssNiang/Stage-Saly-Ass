# Vendredi, 4 novembre 2022
## I. Installing AWX on an Ubuntu distribution
1. Open the terminal
2. Update and Upgrade the system.
```shell
    assniang@ASS-NIANG:~$ sudo apt update && sudo apt upgrade -y
```
3. Execute the following commands
```shell
    assniang@ASS-NIANG:~$ apt install docker.io
    assniang@ASS-NIANG:~$ systemctl enable docker
    assniang@ASS-NIANG:~$ systemctl status docker
    assniang@ASS-NIANG:~$ apt install curl
    assniang@ASS-NIANG:~$ curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    assniang@ASS-NIANG:~$ chmod +x /usr/local/bin/docker-compose
    assniang@ASS-NIANG:~$ docker-compose --version
    assniang@ASS-NIANG:~$ apt update
    assniang@ASS-NIANG:~$ sudo apt install ansible
    assniang@ASS-NIANG:~$ sudo apt install -y nodejs npm
    assniang@ASS-NIANG:~$ sudo npm install npm --global
    assniang@ASS-NIANG:~$ sudo apt install python3-pip git pwgen vim
    assniang@ASS-NIANG:~$ sudo pip3 install requests
    assniang@ASS-NIANG:~$ sudo pip3 install docker-compose==1.23.1
    assniang@ASS-NIANG:~$ wget https://github.com/ansible/awx/archive/refs/tags/17.1.0.zip
    assniang@ASS-NIANG:~$ apt install unzip
    assniang@ASS-NIANG:~$ unzip 17.1.0.zip
    assniang@ASS-NIANG:~$ cd awx-17.1.0/installer/
    assniang@ASS-NIANG:~$ pwgen -N 1 -s 30 # copy the output
    assniang@ASS-NIANG:~$ vi inventory # paste it in the 'secret_key', then set the username, and password.
    assniang@ASS-NIANG:~$ ansible-playbook -i inventory install.yml
    assniang@ASS-NIANG:~$ netstat -tnlp # then, use the host ip address to connect to the AWX GUI.
```
AWX-21.8.0
```shell
    assniang@ASS-NIANG:~$ apt install docker.io
    assniang@ASS-NIANG:~$ systemctl enable docker
    assniang@ASS-NIANG:~$ systemctl status docker
    assniang@ASS-NIANG:~$ apt install curl
    assniang@ASS-NIANG:~$ curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    assniang@ASS-NIANG:~$ chmod +x /usr/local/bin/docker-compose
    assniang@ASS-NIANG:~$ docker-compose --version
    assniang@ASS-NIANG:~$ apt update
    assniang@ASS-NIANG:~$ sudo apt install ansible
    assniang@ASS-NIANG:~$ sudo apt install -y nodejs npm
    assniang@ASS-NIANG:~$ sudo npm install npm --global
    assniang@ASS-NIANG:~$ sudo apt install python3-pip git pwgen vim
    assniang@ASS-NIANG:~$ sudo pip3 install requests
    assniang@ASS-NIANG:~$ sudo pip3 install docker-compose==1.23.1
    assniang@ASS-NIANG:~$ git clone -b 21.8.0 https://github.com/ansible/awx.git
    assniang@ASS-NIANG:~$ pip install --upgrade PyYaml
    assniang@ASS-NIANG:~$ make docker-compose-build
    assniang@ASS-NIANG:~$ make docker-compose 
    assniang@ASS-NIANG:~$ # laisser docker-compose en background et ouvrir un autre terminal
    assniang@ASS-NIANG:~$ docker exec tools_awx_1 make clean-ui ui-devel
    assniang@ASS-NIANG:~$ docker exec -ti tools_awx_1 awx-manage createsuperuser
    assniang@ASS-NIANG:~$ # The UI can be reached in your browser at https://localhost:8043/#/home

```


4. Some tips to change the ownership of a document, to make it editable by a given user:
```shell
    assniang@ASS-NIANG:~$ ls -lsa <path_of_the_document> # to verify the ownership
    assniang@ASS-NIANG:~$ whoami # to get the username of the current user
    assniang@ASS-NIANG:~$ groups # to get all groups where the current user is a member
    assniang@ASS-NIANG:~$ sudo chown <username>:<group> <path_of_the_document> # to change the ownership of the document
```


## II. TO DO NEXT
- Optimiser les playbooks

## III. Some useful links

1. [Install AWX on ubuntu 20](https://www.youtube.com/watch?v=NolU7yKfLGU)
2. [AttributeError: module 'collections' has no attribute 'Hashable' (conds ainstalaltion with py 10)](https://github.com/ablab/spades/issues/873)
> modify this file : /usr/local/lib/python3.10/dist-packages/yaml/constructor.py
> pip install --upgrade PyYaml
3. [ansible/awx](https://github.com/ansible/awx/tree/devel/tools/docker-compose)
