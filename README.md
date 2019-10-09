## ansible_netapp
This repository is a collection of playbooks written for a NetApp customer.
Currently, these are not roles but they will eventually move to when I have time and with your contribution ;)

### Repository Structure
The structure of the repository looks like this when cloned:

```
playbook-netapp
|D---dev
    |D------inputs
    |D------sandbox
    |D------tasks
    |D------templates
    |D------vars
    |F---ntap_decomm.yml
    |F---ntap_provision.yml
|D---prod

```

- `dev`: directory containing all the dev environment
- `dev/inputs`: contains files about user inputs like name, size, ... 
- `dev/sandbox`: old playbooks to be converted 
- `dev/tasks`: playbook files containing one tasks only to be reused with a master playbook
- `dev/templates`: template to use to create a volume, lun, nfs export, smb share
- `dev/vars`: contains inventory files
- `prod`: directory containing production ready playbooks

Note: if you include credentials in the inventory file, it is strongly adviced to encrypt the file for obvious security reasons.

### Playbook structure & usage
In order to make it as generic as possible and reusable at will, there is generic playbooks including several standalone tasks loaded dynamically depending on the content of the input file. 

Example:
- `ntap_provision.yml` is a generic playbook 
To run the playbook, the following needs to be done:
- Copy and/or modify `vars/cluster.yml` to match your environment
- Copy one of input template files from the `templates` folder to the `inputs` folder like: `cp template/nas_nfs.yml inputs/req123456.yml` where req123456 could be the ticket reference number
- Modify `inputs/req123456.yml` to the provisioning needs
- Run `ansible-playbook -e "input=req123456" -C` to perform a playbook check and if ok, remove `-C`
