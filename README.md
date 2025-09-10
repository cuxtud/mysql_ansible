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

### Task breakdown

- `disk_setup.yml`: Partition and prepare `mysql_disk` → `mysql_partition`.
- `lvm_setup.yml`: Create VG `mysql_vg_name` and LVs (`mysql_data_lv`, `mysql_archive_lv`, `mysql_backup_lv`) with sizes from `mysql_*_size`.
- `mount_setup.yml`: Format (`mysql_fs_type`) and mount to `mysql_*_dir` paths.
- `directory_setup.yml`: Ensure directory structure and permissions for MySQL.
- `rpm_download.yml`: Optionally fetch/place required RPMs per EL version.
- `mysql_install.yml`: Verify RPM presence for current OS major and install packages.
- `mysql_config.yml`: Drop `my.cnf`, initialize data dirs, and configure service.
- `mysql_database_user_creation.yml`: Optionally create databases/users when provided.

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

Morpheus-driven options (if applicable):

- `morpheus.customOptions.mysqlData`: used to derive `mysql_data_size`, `mysql_archive_size` (50%), `mysql_backup_size` (100%).
- `morpheus.customOptions.mysqlVersion`: used to set `mysql_version`.

### Notes

- This role is focused on EL8. It resolves RPM lists per EL major to avoid cross-version package mixing.
- Ensure the required RPMs are available (either pre-staged on the host, via your repository, or downloaded by the role's RPM tasks if configured).
  - The role checks RPM presence only for the host's EL major (e.g., EL8 checks `mysql_el8_rpm_packages`).

### Example inventory

```ini
[mysql]
db1.example.com
```

### Example run

```bash
ansible-playbook playbooks/mysql.yml -i inventory.ini -e "mysql_password=StrongPassw0rd!"
```
