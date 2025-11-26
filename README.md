## MySQL Ansible Role (EL8/EL9/EL10)

This repository ships a single Ansible role (`roles/mysql`) plus a helper play (`playbooks/mysql.yml`) that provisions a fully managed MySQL 8 environment on RHEL/Rocky/Alma Linux. The role currently understands two upstream versions (8.0.43 and 8.4.3) and can size disks/LVs from Morpheus custom options before configuring the database, users, and backup access.

### Repository Layout

- `playbooks/mysql.yml` &mdash; minimal example play that applies the role to `all`.
- `roles/mysql/{defaults,vars}` &mdash; opinionated defaults plus version-specific RPM definitions (`vars/8.0.43.yml`, `vars/8.4.3.yml`).
- `roles/mysql/tasks` &mdash; task files split by concern (disk/LVM, mounting, RPM download, install, configuration, ownership, DB/user creation).
- `roles/mysql/files/my.cnf` &mdash; base MySQL configuration copied to `/etc/my.cnf`.

> The previous `ansible.cfg`/`inventories` scaffolding was removed; bring your own inventory and config when consuming this role.

### Requirements

- Ansible 2.13+ with access to your Morpheus inventory (if using the provided defaults).
- Collections: `community.mysql`.
- Managed hosts need `python3-PyMySQL` (installed by `mysql_config.yml`) and must allow `lookup('cypher', ...)` if you keep the default secret retrievals.
- Outbound HTTPS access (optionally via the provided `proxy_env`) to download RPM bundles and GPG keys.

Install the MySQL collection locally:

```bash
ansible-galaxy collection install community.mysql
```

### Core Variables (override in `group_vars`, inventory, or `-e`)

- **Storage sizing**: `mysql_disk`, `mysql_partition`, `mysql_vg_name`, `mysql_data_lv`, `mysql_archive_lv`, `mysql_backup_lv`. Sizes default to Morpheus custom options (`mysql_data_size`, archive 50%, backup 100%).
- **Mount points**: `mysql_data_dir`, `mysql_archive_dir`, `mysql_backup_dir`, plus ownership controlled in `mysql_set_ownership.yml`.
- **Version handling**: `mysql_version` drives which vars file is included. Each vars file provides `mysql_el*_rpm_bundle` + `mysql_el*_rpm_packages`. EL10 support currently exists for 8.0.43.
- **Credentials**: `mysql_new_password`, `mysql_backup_user`, `mysql_backup_user_password`, `mysql_user_db`, `mysql_user_name`, `mysql_user_password`.
- **Proxy**: `proxy_env` is used by RPM downloads and GPG key imports; customize or disable per environment.

See `roles/mysql/defaults/main.yml` for the authoritative list.

### Role Workflow

1. **Version vars include** (`tasks/main.yml`) selects `vars/8.0.43.yml` or `vars/8.4.3.yml`.
2. **Disk prep** via `disk_setup.yml` (partition creation) and `lvm_setup.yml` (PV/VG/LV provisioning, XFS formatting, optional `/usr` resize).
3. **Mounting and directory prep** in `mount_setup.yml`, `directory_setup.yml`, followed by `mysql_set_ownership.yml` to enforce ownership of `/u101`, `/u102`, `/u103`.
4. **RPM handling** (`rpm_download.yml`) downloads the correct bundle per EL major, using the proxy environment, and extracts it under `/tmp`.
5. **Install phase** (`mysql_install.yml`) validates bundle contents, removes conflicting `mariadb-connector-c`, imports the MySQL GPG key, installs prerequisites (`mysql_req_packages`), then installs every RPM detected for the running OS.
6. **Configuration** (`mysql_config.yml`) deploys `my.cnf`, starts/enables `mysqld`, installs `python3-PyMySQL`, rotates the root password (using the temporary password from logs when needed), writes `/root/.my.cnf`, and ensures the `mysql` user has `/bin/bash`.
7. **Database/User provisioning** (`mysql_database_user_creation.yml`) optionally creates an application database and user plus the `azbackup` account with privileges, adapting auth plugins per version.

### RPM & Version Matrix

- **8.0.43**: downloads from `cdn.mysql.com` and supports EL8/EL9/EL10.
- **8.4.3**: downloads from the internal AZ Artifact repository and supports EL8/EL9.

The install logic only checks/installs packages for the host's `ansible_distribution_major_version`, preventing cross-version contamination.

### Usage

```
ansible-playbook playbooks/mysql.yml -i inventory.ini \
  -e mysql_version=8.4.3 \
  -e mysql_disk=/dev/sdc \
  -e mysql_user_db=mydb \
  -e mysql_user_name=myapp \
  -e mysql_user_password='Sup3rSecret!'
```

Override storage sizes, Morpheus-derived values, or proxy settings as required. If Morpheus is not in play, ensure `mysql_*_size`, `mysql_new_password`, and `mysql_user_*` variables are set explicitly.

### Additional Notes

- `roles/mysql/files/my.cnf` assumes data under `/u101/mysql/data` and binlogs under `/u102/mysql/archive/binlog`; adjust the file (or supply a template) if your layout differs.
- `mysql_database_user_creation.yml` writes `/root/.my.cnf` twice (once before DB creation, once conditionally in config). Ensure your security policies allow this or customize accordingly.
- Consider pre-staging RPM bundles if outbound access is restricted; place them in `/tmp` with the expected filenames to skip the download block.
- The role currently enables but does not start backups—`azbackup` merely grants access. Wire this user into your backup tooling separately.
