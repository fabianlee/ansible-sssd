# refual.sssd

Install SSSD and configure the authentication backend

## Requirements

- Ansible >= 2.5
- Python >= 3.6 (on the target system if using the custom fact)
- a compatible OS:
  - Debian/Ubuntu
  - RedHat-like

## Role Variables

The main configuration variables and their defaults:

```yaml
# If True, will deploy custom Ansible facts (currently only reports installed SSSD version).
sssd_deploy_facts: True
# List of domains for sssd.conf.
# See the playbook below for an example.
sssd_domains: {}
# When set to True, will install oddjobd and oddjobd-mkhomedir
# to automatically generate home directories as the user first log in.
sssd_oddjob_mkhomedir: False
# List of service sections for sssd.conf, such as nss, pam, etc.
# See the playbook below for an example.
sssd_services: {}
```

Additional variables you may override if necessary:

```yaml
# Arguments for the authconfig utility, without the dashes.
# Automatically adds enablemkhomedir if sssd_oddjob_mkhomedir is True.
sssd_authconfig: "{{ ['enablemkhomedir', 'enablesssdauth'] if sssd_oddjob_mkhomedir else ['enablesssdauth'] }}"
# Packages to install for oddjobd and oddjobd-mkhomedir, if sssd_oddjob_mkhomedir is True.
sssd_oddjob_packages:
  - oddjob
  - oddjob-mkhomedir
# Packages to install for SSSD. Defaults to sssd on Debian and authconfig + sssd on RedHat.
sssd_packages: "{{ __sssd_debian_sssd_packages if __sssd_debian else __sssd_redhat_sssd_packages }}"
# The configuration directives for the sssd section of sssd.conf.
# In addition to this, the values for the domains and services directives will be autogenerated
# from the entries in sssd_domains and sssd_services, although you are free to override them.
sssd_sssd:
  config_file_version: 2
```

## Example Playbook

```yaml
---
- name: SSSD
  hosts: localhost
  roles:
    - refual.sssd
  vars:
    sssd_services:
      nss: {}
      pam:
        offline_credentials_expiration: 3
    sssd_domains:
      example.com:
        id_provider: ldap
        auth_provider: ldap
        ldap_uri: ldap://ad1.example.com, ldap://ad2.example.com
        ldap_schema: rfc2307bis
        ldap_id_use_start_tls: false
        ldap_tls_reqcert: never

        ldap_default_bind_dn: cn=manager,dc=example,dc=com
        ldap_default_authok: bindpassword
        ldap_default_authok_type: password

        ldap_search_base: dc=example,dc=com
        ldap_user_search_base: dc=users,dc=example,dc=com
        ldap_user_object_class: user
        ldap_user_gecos: name
        ldap_group_search_base: dc=groups,dc=example,dc=com
        ldap_group_object_class: group

        override_shell: /bin/bash

        cache_credentials: true
    sssd_oddjob_mkhomedir: True
```

License
-------

[MIT](https://github.com/refual/ansible-sssd/blob/master/LICENSE)

Author Information
------------------

- [Paul Laufer](https://github.com/refual)

