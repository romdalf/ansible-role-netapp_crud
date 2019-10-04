## ansible_netapp
This repository is a collection of playbooks written for a NetApp customer.
Currently, these are not roles but they will eventually move to when I have time and with your contribution ;)

### Repository Structure
The structure of the repository looks like this when cloned:

```
playbook-netapp
|---dev
    |---sandbox
    |---tasks
        |---inputs
            |---templates
        |---vars
|---ntap_provision.yml

```
 
- `dev`: contains playbooks currently in development and should not be used against production systems.
- `sandbox`: old playbooks to be converted 
- `tasks`: playbook files containing one tasks only to be reused with a master playbook
- `inputs`: contains files about user inputs like name, size, ... 
- `template`: template to use to create a volume, lun, nfs export, smb share
- `vars`: contains inventory files

Note: if you include credentials in the inventory file, it is strongly adviced to encrypt the file for obvious security reasons.

### Playbook Structure
In order to make it as generic as possible and reusable at will, there is a "global" and "subset" playbooks including several others ones loaded dynamically depending on the needs. Example:
`ntap_nas.yml` is a global playbook and will: 
- include `ntap_qtree.yml` (subset playbook) if qtree is defines in the `nas-inputs.yml`
- include `ntap_[smb or nfs].yml` (subset playbook) if smb or nfs is defined in the `nas-inputs.yml`

### Usage
- verify the inventory file to insure the connection details are configured
- update the related input file (san or nas) with the settings to create or delete the object(s)

Note:
- using absent in playbook targetting as an example volume, lun, mapping will destroy the overall!!
