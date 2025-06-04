## Requirements
* Ansible 2.10+
* Change default token:
```
## Pre-shared secret token that other server or agent nodes will register with when connecting to the cluster
rke2_token: defaultSecret12345
```
## RKE2 version: Pick a proper version (avoid some ui bugs..)
```
rke2_version: v1.22.6+rke2r1
```
## 1.22 not require "swap off"

## Inventory file example

This role relies on nodes distribution to `masters` and `workers` inventory groups.
The RKE2 Kubernetes master/server nodes must belong to `masters` group and worker/agent nodes must be the members of `workers` group. Both groups has to be the children of `k8s_cluster` group.

```ini
[masters]
master-01 ansible_host=192.168.123.1 rke2_type=server
master-02 ansible_host=192.168.123.2 rke2_type=server
master-03 ansible_host=192.168.123.3 rke2_type=server

[workers]
worker-01 ansible_host=192.168.123.11 rke2_type=agent
worker-02 ansible_host=192.168.123.12 rke2_type=agent
worker-03 ansible_host=192.168.123.13 rke2_type=agent

[k8s_cluster:children]
masters
workers
```

## Playbook example

This playbook will deploy RKE2 to a single node acting as both server and agent.

```yaml
- name: Deploy RKE2
  hosts: node
  become: yes
  roles:
     - role: rke2

```

This playbook will deploy RKE2 to a cluster with one server(master) and several agent(worker) nodes.

```yaml
- name: Deploy RKE2
  hosts: all
  become: yes
  roles:
     - role: rke2

```

This playbook will deploy RKE2 to a cluster with one server(master) and several agent(worker) nodes in air-gapped mode. This works from downloading artifacts. When the RKE2 script installs, it will use the artifacts instead of using online resources.

```yaml
- name: Deploy RKE2
  hosts: all
  become: yes
  vars:
    rke2_airgap_mode: true
  roles:
     - role: rke2

```

This playbook will deploy RKE2 to a cluster with HA server(master) control-plane and several  agent(worker) nodes. The server(master) nodes will be tainted so the workload will be distributed only on worker/agent nodes. The role will install also keepalived on the control-plane nodes and setup VIP address where the Kubernetes API will be reachable. it will also download the Kubernetes config file to the local machine.

```yaml
- name: Deploy RKE2
  hosts: all
  become: yes
  vars:
    rke2_ha_mode: true
    rke2_server_taint: true
    rke2_api_ip : 192.168.123.100
    rke2_download_kubeconf: true
  roles:
     - role: rke2
```


# Update Ansible > 2.10 to resolve this issue

```
ERROR! this task 'ansible.builtin.include_tasks' has extra params, which is only allowed in the following modules: shell, group_by, include_vars, script, command, include, include_role, import_tasks, win_command, win_shell, add_host, import_role, raw, set_fact, meta, include_tasks

The error appears to be in '/usr/local/src/rke2/roles/rke2/tasks/main.yml': line 3, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:


- name: Install Keepalived when HA mode is enabled
  ^ here

```


# This requires internet outbound

```
fatal: [master-02]: FAILED! => {"changed": false, "msg": "Failed to update apt cache: unknown reason"}
```
