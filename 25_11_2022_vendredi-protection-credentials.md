# Protection of the credentials

1. We removed informations about ansible_user and ansible_password written in the inventory file. 
2. To run the jobs on AWX, we created a Machine type credential with the username and password of the fortigate. 
3. Then we add this credential to the job we want to run.
