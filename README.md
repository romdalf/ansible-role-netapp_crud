# ansible_netapp
This repository is a collection of playbooks written for a NetApp customer.

## Structure
The structure is as this:
- inputs: content a couple of yaml file to define the object(s) to be created (volume, qtree, lun, smb share, nfs, exports...)
- vars: content the inventory files

Note: if you include credentials in the inventory file, it is strongly adviced to encrypt the file for obvious security reasons.

## Usage
- verify the inventory file to insure the connection details are configured
- update the related input file (san or nas) with the settings to create or delete the object(s)

Note:
- using absent in playbook targetting as an example volume, lun, mapping will destroy the overall!!
