stuttgart-things/install-configure-docker
=========================================

deployment and configuration of docker (compose).

install this role + dependecies
-------------------------------------

<details><summary><b>requirements</b></summary>

copy the and paste on your ansible host to install the required roles/collections:

```
cat <<EOF > /tmp/requirements.yaml
---
roles:
- src: https://github.com/stuttgart-things/install-configure-docker.git
  scm: git
- src: https://github.com/stuttgart-things/install-requirements.git
  scm: git

collections:
- name: community.general
  version: 2.0.1
- name: community.docker
  version: 1.2.2
EOF

ansible-galaxy install -r /tmp/requirements.yaml --force
ansible-galaxy collection install -r /tmp/requirements.yaml --force
rm -rf /tmp/requirements.yaml
```
</details>

Example Playbook 1: Basic Docker installation + compose
-------------------
```
- hosts: "{{ target_host }}"
  become: true
  
  vars:
    docker_install_compose: true

  roles:
    - install-configure-docker
```

Example Playbook 2: Docker Installation + registry login and mirror
-------------------
```
- hosts: "{{ target_host | default('all') }}"
  become: true

  vars:
    docker_install_compose: true 

    add_registry_mirrors: true
    registry_mirrors:
      - https://harbor.tiab.labda.sva.de
      - https://harbor.labul.sva.de

    vault_url: "{{ lookup('env','VAULT_URL') }}"
    vault_token: "{{ lookup('env','VAULT_TOKEN') }}"

    login_into_private_registry: true
    private_registry_url: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=harbor/data/harbor validate_certs=false token={{vault_token}} url={{vault_url}}')['url'] }}"
    private_registry_user: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=harbor/data/harbor validate_certs=false token={{vault_token}} url={{vault_url}}')['user'] }}"
    private_registry_password: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=harbor/data/harbor validate_certs=false token={{vault_token}} url={{vault_url}}')['password'] }}"

  roles:
    - install-configure-docker
```

Role history
----------------
| date  | who | changelog |
|---|---|---|
|2021-24-02   | Patrick Hermann | Added support for registry mirrors and login to private registry
|2020-10-10   | Patrick Hermann | Updated for using of ansible collections, fixed role structure, added to be stable version
|2020-04-01  | Patrick Hermann | intial commit for this role in codehub

License
-------

BSD

Author Information
------------------

Patrick Hermann (patrick.hermann@sva.de); 04/2020

License:
-------

BSD

Author Information
------------------
This role was created in 2017 by Jeff Geerling, author of Ansible for DevOps - adopted to be used in multiple stuttgart-things projects by Patrick Hermann in 2020.
