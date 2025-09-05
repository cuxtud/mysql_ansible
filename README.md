## MySQL Ansible

This repository contains an Ansible setup to install and configure MySQL 8.

### Structure

- `ansible.cfg`: Ansible configuration with inventory and defaults
- `inventories/dev/hosts`: Example inventory with a `mysql` group
- `site.yml`: Main playbook including the `mysql` role
- `playbooks/mysql.yml`: Example playbook applying the role
- `roles/mysql`: Role to install/configure MySQL 8

### Usage

1. Ensure you have Ansible installed and, for MySQL modules, install collections:

   ```bash
   ansible-galaxy collection install community.mysql
   ```

2. Run against the dev inventory:

   ```bash
   ansible-playbook site.yml
   ```

3. Override variables as needed via `-e` or by editing `roles/mysql/defaults/main.yml`.

### Variables

- `mysql_version` (default: `8.0`)
- `mysql_bind_address` (default: `127.0.0.1`)
- `mysql_port` (default: `3306`)
- `mysql_root_password` (default: `changeMe!`)
- `mysql_databases`: list of database names or objects `{ name: db }`
- `mysql_users`: list of objects `{ name, password, host, priv }`

### Notes

- The default root password task is intended for development. Harden for production.
- Debian/Ubuntu and RHEL-family repository setup tasks are included.

# mysql_ansible
Install mysql 8 and above using ansible on redhat and rocky linux 8 and 9
