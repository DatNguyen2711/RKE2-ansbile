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


# Update cluster

- For example, you run this playbooks to deploy cluster v1.22.

```yaml
- name: Install rancher rke2
  hosts: k8s_cluster
  become: true
  vars:
    rke2_ha_mode: true
    rke2_api_ip : 192.168.10.100
    rke2_server_taint: true
    rke2_download_kubeconf: true
  roles:
  - role: rke2
```

- If you want to upgrade to version 1.28 you have to add this line then run playbooks.

```yaml
- name: Install rancher rke2
  hosts: k8s_cluster
  become: true
  vars:
    rke2_version: v1.28.5+rke2r1   #add the version you want to upgrade
    rke2_ha_mode: true
    rke2_api_ip : 192.168.10.100
    rke2_server_taint: true
    rke2_download_kubeconf: true
  roles:
  - role: rke2
```
