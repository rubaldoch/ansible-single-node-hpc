# Single Node HPC

The following playbook will setup openldap with TLS, ldapscripts and lmod.

## Definitions

- Control Node: machine where ansible runs
- Managed Node: machine managed with ansible 

## Requirements

Execute the following commands in the control node.

- Install ansible

```bash
 python3 -m pip install --user ansible
```

- Create a ssh-key to allow password-less authentication to the managed node.

```bash
ssh-keygen -t ed25519 -C "ansible" # ~/.ssh/ansible
ssh-copy-id -i ~/.ssh/ansible.pub <user>@<ip-managed-node> 
```
- Update `inventory.yaml`. 

    ```yml
    servers:
        hosts:
            server01:
                ansible_host: 192.168.122.5
                ansible_user: ubuntu
                node_hostname : darwin
        vars:
            ldap_organization: Darwin
            ldap_domain: darwin.pe
            ldap_dn: dc=darwin,dc=pe
            ldap_host: localhost
            ldap_password: alabama
            ldap_default_user_password: darwin123
    ```
    - **ansible_host:** managed node ip/host
    - **ansible_user:** managed node user
    - **node_hostname:** managed node hostname
    - **ldap_organization:** name of the ldap organization 
    - **ldap_domain:** domain of the ldap organization 
    - **ldap_dn:** dn of the ldap organization. Must match with ldap_domain.
    - **ldap_host:** host/ip where ldap will run. 
        - `localhost`: for working only on the managed node. 
        - `managed node hostname/ip`: for working in LAN.
    - **ldap_password:** password for ldap admin user `cn=admin,{{ ldap_dn }}`. 
    - **ldap_default_user_password:** default password for ldap users. 

- Test ansible connection

    ```bash
    ansible all -m ping -i inventory.yaml
    ```

## Execution

- Execute all ansible roles in playbook

    ```bash
    ansible-playbook -i inventory.yaml playbook.yaml --ask-become-pass
    ```
- List existing tags
    ```bash
    ansible-playbook -i inventory.yaml playbook.yaml --list-tags 
    ```
- Execute specific tag
    ```bash
    ansible-playbook -i inventory.yaml playbook.yaml --ask-become-pass --tags  <specific-tag>
    ```
## Ldap Organization

- After running the ansible playbook ldap will be created with the following entities
    ```text
    dn: dc=darwin,dc=pe
    objectClass: top
    objectClass: dcObject
    objectClass: organization
    o: Darwin
    dc: darwin

    dn: ou=People,dc=darwin,dc=pe
    objectClass: organizationalUnit
    ou: People

    dn: ou=Groups,dc=darwin,dc=pe
    objectClass: organizationalUnit
    ou: Groups

    ```

- `dn: ou=People`: to store ldap users
- `dn: ou=Groups`: to store ldap users

## Post Install

- Every user should have a group. To create a user, you will first need to create a group:

    ```bash
    # inside managed node
    sudo ldapaddgroup <group-name>
    sudo ldapadduser <user-name> <group-name>
    ```
- If you want a `<group-name>` with sudoers permissions. You must add this group to sudoers. Enter `/etc/sudoers` with `sudo visudo /etc/sudoers` and add the following line.

    ```shell
    # Alllow Ldap Admins members to execute any command
    %<group-name> ALL=(ALL:ALL) ALL
    ```

## References
- [Install and configure LDAP](https://ubuntu.com/server/docs/install-and-configure-ldap)
- [Installing Lua and Lmod](https://lmod.readthedocs.io/en/latest/030_installing.html)