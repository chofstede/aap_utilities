# infra.aap_utilities.aap\_setup\_prepare

A role to prepare the installation of AAP 2.x, installing pre-requisites,
unpacking the installation tarball and (optionally) writing the necessary inventory file.

## Requirements

* The installer tarball must be available, by default downloaded with the `aap_setup_download` role.
* The (RPM) pre-requisites must have been installed, or root access must be given.

## Role Variables

The following input variables are available:

|Variable Name|Default Value|Required|Description|Example|
|---:|:---:|:---:|:---|:---:|
|`aap_setup_prep_installer_file`|"`{{ aap_setup_down_installer_file }}`"|no|absolute path where to find the tarball on the remote host, or URL http(s), note that `aap_setup_down_installer_file` is a fact set by the role `aap_setup_download`|`'https://myhost/myinstaller.tar.gz'` or `'/var/tmp/myinstaller.tar.gz'`|
|`aap_setup_prep_working_dir`|"`{{ aap_setup_working_dir \| default('/var/tmp') }}`"|no|absolute path to a working directory, note that `aap_setup_working_dir` is used by other roles in the collection|'/srv/workdir'|
|`aap_setup_prep_process_template`|true|no|shall the inventory be generated by the role?|false|
|`aap_setup_prep_inv_nodes`|none|yes|a dictionary of dictionaries, the first level key is the inventory group name, the 2nd level key is the hostname with the value being its inventory host variables in INI-format|see [defaults/main.yml](defaults/main.yml)|
|`aap_setup_prep_inv_vars`|{}|see below|a dictionary of dictionaries, the first level key is the inventory group name, the 2nd level key is the variable name with the value being the variable's value|see [defaults/main.yml](defaults/main.yml)|
|`aap_setup_prep_inv_secrets`|{}|see below|a dictionary of dictionaries, the first level key is the inventory group name, the 2nd level key is the variable name with the value being the variable's value|see [defaults/main.yml](defaults/main.yml)|

Some notes about the inventory variables and secrets:

* both values will be combined (the secrets overwriting the variables) and used to generate the installation inventory, so that secret variables can be defined separately for example in a vault.
* even if formally both variables don't need to be defined, you'll get a viable inventory only if you define some keys/variables at least in the group `all`.
By convention the [defaults/main.yml](defaults/main.yml) contains all possible variables as comments, the variables commented out _twice_ are truly optional.

## Dependencies

* `aap_setup_download`, in the same collection, can be used to download the tarball automatically.

## Example Playbook

```yaml
- name: download and install AAP from the bastion
  hosts: bastion
  gather_facts: false
  become: false
  tags: aap_installation

  vars_files:
    - inventory_vars/variables.yml
  roles:
    - infra.aap_utilities.aap_setup_download
    - infra.aap_utilities.aap_setup_prepare
    - infra.aap_utilities.aap_setup_install
```

Note that this only works without root access if the bastion host isn't part of the future cluster,
and if the RPM pre-requisites have been pre-installed.
Else change to `become: true`.

## Example Inventory Variables

```yaml
---
aap_setup_down_type: "setup-bundle"
aap_setup_rhel_version: 8

aap_setup_prep_inv_nodes:
  automationcontroller:
    - ansible-ctrl.example.com
  automationhub:
    - ansible-hub.example.com
  automationedacontroller:
    - ansible-eda.example.com
  database:
    - database.example # if using an already existing DB, remove this and ensure that the following variables are filled with the valid details for your Controller and PAH
                       # databases: pg_host, pg_port, pg_database, ph_username, pg_password, automationhub_pg_host, automationhub_pg_port, automationhub_pg_database,
                       # automationhub_pg_username, automationhub_pg_password, automationhub_pg_sslmode
  execution_nodes:
    - execution-1.example
    - execution-2.example
  #servicescatalog_workers:
  #sso:

aap_setup_prep_inv_vars:
  automationcontroller: # denotes the automation controller nodes as hybrid nodes (controller and execution)
    peers: execution_nodes
    node_type: hybrid

  execution_nodes:
    node_type: execution

  all:
    ansible_user: ansible
    ansible_become: true
    admin_password: changeme # admin password for Automation Controller UI
    pg_host: 'database.example'
    pg_port: '5432'

    pg_database: 'awx'
    pg_username: 'awx'
    pg_password: changeme
    pg_sslmode: 'prefer'  # set to 'verify-full' for client-side enforced SSL

    registry_url: 'registry.redhat.io'
    receptor_listener_port: 27199

    automationhub_admin_password: changeme # admin password for PAH UI
    automationhub_pg_host: 'database.example'
    automationhub_pg_port: '5432'

    automationhub_pg_database: 'automationhub'
    automationhub_pg_username: 'automationhub'
    automationhub_pg_password: changeme
    automationhub_pg_sslmode: 'prefer'
    automationhub_main_url: https://hub.example #url, not hostname
    automationhub_require_content_approval: False
    automationhub_enable_unauthenticated_collection_access: True

    automationhub_ssl_validate_certs: False

    automationedacontroller_admin_password: 'password' # Admin password for EDA UI
    automationedacontroller_pg_host: 'controller.aap24.local'
    automationedacontroller_pg_port: '5432'
    automationedacontroller_pg_database: 'automationedacontroller'
    automationedacontroller_pg_username: 'automationedacontroller'
    automationedacontroller_pg_password: 'password'

    sso_console_admin_password: ''

aap_setup_prep_inv_secrets:
  all:
    registry_username: changeme
    registry_password: changeme
```

## License

[GPLv3+0](https://github.com/redhat-cop/aap_utilities#licensing)

## Author Information

Eric Lavarde <elavarde@redhat.com>
