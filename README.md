stuttgart-things/install-configure-docker
=========================================

deployment and configuration of docker (compose).

<details><summary><b>VARIABLES</b></summary>

* `set_proxy` - Set on true to generate http-proxy.conf (default:false)
* `add_registry_mirrors` - Set on true to Configure daemon.json (default:false)
* `login_into_private_registry` - Set on true to install python packages and to log into a private registry (default:false)
* `install_kind` - Set on true to install Kind and Kubectl (default:false)
* `docker_install_compose` - Set on true/false to (not) install docker compose

</details>

<details><summary><b>ROLE INSTALLATION</b></summary>

installs role and all of it's dependencies w/:

```bash
cat <<EOF > /tmp/requirements.yaml
---
roles:
- src: https://github.com/stuttgart-things/install-configure-docker.git
  scm: git
- src: https://github.com/stuttgart-things/install-requirements.git
  scm: git

collections:
- name: community.general
  version: 7.0.1
- name: community.docker
  version: 3.4.8
EOF

ansible-galaxy install -r /tmp/requirements.yaml --force
ansible-galaxy collection install -r /tmp/requirements.yaml --force
rm -rf /tmp/requirements.yaml
```

</details>

<details><summary>EXAMPLE INVENTORY</summary>

```bash
cat <<EOF > inventory
[appserver]
1.2.3.4 ansible_user=sthings
EOF
```

</details>

<details><summary><b>EXAMPLE PLAYBOOK - BASIC DOCKER AND DOCKER COMPOSE INSTALLATION</b></summary>

```yaml
cat <<EOF > install-configure-docker.yaml
---
- hosts: "{{ target_host | default('all') }}"
  become: true
  
  vars:
    docker_install_compose: true
    docker_version: '' # leave empty for latest version or set e.g. '=5:23.0.6-1~ubuntu.23.04~lunar'

  roles:
    - install-configure-docker
EOF
```

</defaults>

<details><summary><b>EXAMPLE PLAYBOOK - DOCKER INSTALLATION + REGISTRY LOGIN AND MIRROR</b></summary>

```yaml
cat <<EOF > install-configure-docker.yaml
---
- hosts: "{{ target_host | default('all') }}"
  become: true

  vars:
    docker_install_compose: true 
    docker_version: '' # leave empty for latest version or set e.g. '=5:23.0.6-1~ubuntu.23.04~lunar'

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
EOF
```

</defaults>

<details><summary><b>EXAMPLE PLAYBOOK - DOCKER INSTALLATION + KIND AND KUBECTL</b></summary>

```yaml
cat <<EOF > install-configure-docker.yaml
---
- hosts: "{{ target_host | default('all') }}"
  become: true

  vars:
    docker_install_compose: true 
    docker_version: '' # leave empty for latest version or set e.g. '=5:23.0.6-1~ubuntu.23.04~lunar'

    install_kind: false
    kind_version: 0.20.0
    kind_url: "https://github.com/kubernetes-sigs/kind/releases/download/v{{ kind_version }}/kind-linux-amd64"
    kind_cluster_name: kind1
    kubectl_version: 1.27.3
    kind_api_port: 6443
    kind_server: 127.0.0.1

    bin:
      kubectl:
        bin_name: kubectl
        bin_version: "{{ kubectl_version }}"
        source_url: "https://dl.k8s.io/v{{ kubectl_version }}/kubernetes-client-linux-amd64.tar.gz"
        bin_to_copy: "kubernetes/client/bin/kubectl"
        bin_dir: /usr/bin/
        to_remove: kubernetes
        check_bin_version_before_installing: true
      kind:
        bin_name: "kind"
        bin_version: "{{ kind_version }}"
        source_url: "https://github.com/kubernetes-sigs/kind/releases/download/v{{ kind_version }}/kind-linux-amd64"
        bin_to_copy: kind-linux-amd64
        bin_dir: /usr/bin/
        to_remove: kind-linux-amd64
        check_bin_version_before_installing: true
EOF
```

</defaults>

<details><summary>EXAMPLE EXECUTION</summary>

```bash
ansible-playbook -i inventory install-configure-docker.yaml -vv
```

</details>

## License
<details><summary>LICENSE</summary>

Copyright 2020 patrick hermann.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
</details>

Role history
----------------
| date  | who | changelog |
|---|---|---|
|2021-24-02   | Andre Ebert | Ansible- and Yamllint added collection release workflow
|2021-24-02   | Patrick Hermann | Added support for registry mirrors and login to private registry
|2020-10-10   | Patrick Hermann | Updated for using of ansible collections, fixed role structure, added to be stable version
|2020-04-01  | Patrick Hermann | intial commit for this role in codehub

Author Information
------------------

```yaml
Andre Ebert (andre.ebert@sva.de); 04/2023

Patrick Hermann (patrick.hermann@sva.de); 04/2020

This role was created in 2017 by Jeff Geerling, author of Ansible for DevOps - adopted to be used in multiple stuttgart-things projects by Patrick Hermann in 2020.
```
