## MySQL Ansible (EL8-focused)

This repository contains an Ansible role and playbooks to install and configure MySQL 8 on Enterprise Linux, primarily tested on EL8 (RHEL/Rocky/Alma 8). The role can select RPM sets based on the host's EL major version.

### Structure

- `ansible.cfg`: Ansible configuration with inventory and defaults
- `inventories/dev/hosts`: Example inventory with a `mysql` group
- `site.yml`: Main playbook including the `mysql` role
- `playbooks/mysql.yml`: Example playbook applying the role
- `roles/mysql`: Role to install/configure MySQL 8

### Prerequisites

- Ansible 2.13+
- Collection: `community.mysql`

```bash
ansible-galaxy collection install community.mysql
```

### Usage

Run the role with your inventory (example uses all hosts):

```bash
ansible-playbook playbooks/mysql.yml -i <your_inventory>
```

Override variables as needed via `-e` or by editing `roles/mysql/defaults/main.yml`.

### Variables

Core (RPM and OS targeting):

- `mysql_el8_rpm_packages`, `mysql_el9_rpm_packages`, `mysql_el10_rpm_packages`: lists of RPM names per EL major
- The role dynamically picks the correct list based on `ansible_facts.distribution_major_version` (EL8 focus)

Disk/LVM layout (for data, archive, backup):

- `mysql_disk`: e.g. `/dev/sdb`
- `mysql_partition`: e.g. `/dev/sdb1`
- `mysql_vg_name`: e.g. `datavg`
- `mysql_data_lv`, `mysql_archive_lv`, `mysql_backup_lv`
- `mysql_data_size`, `mysql_archive_size`, `mysql_backup_size` (can be driven by Morpheus custom options)
- `mysql_fs_type`: default `xfs`

Directories and ownership:

- `mysql_data_dir`: e.g. `/u101/mysql/data`
- `mysql_archive_dir`: e.g. `/u102/mysql/archive`
- `mysql_backup_dir`: e.g. `/u103/mysql/backup`
- `mysql_user`: default `mysql`
- `mysql_group`: default `mysql`

MySQL basics:

- `mysql_version`: MySQL version selector (from Morpheus options if used)
- `mysql_password`: root/admin password
- Optional: `mysql_databases` (list), `mysql_users` (list)

### Notes

- This role is focused on EL8. It resolves RPM lists per EL major to avoid cross-version package mixing.
- Ensure the required RPMs are available (either pre-staged on the host, via your repository, or downloaded by the role's RPM tasks if configured).

### Example inventory

```ini
[mysql]
db1.example.com
```

### Example run

```bash
ansible-playbook playbooks/mysql.yml -i inventory.ini -e "mysql_password=StrongPassw0rd!"
```
