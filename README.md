# Deploy Kubernetes cluster

This repository aims to deploy a Kubernetes cluster using [Kubespray](https://github.com/kubernetes-sigs/kubespray) in the most straightforward manner.

In addition, it provides access to GPU nodes by using [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator). In this way, there is no need to worry about installing the required NVIDIA drivers beforehand.

## Set Up

In order to use a custom configuration the `config.example` folder has to be copied to a new one.

```bash
cp -rfp config.example config
```

> **_NOTE:_**  ansible.cfg uses config as the default folder for the inventory file. Another different inventory´s location can be provided using `-i` with the `ansible-playbook` command.

The configuration template contains the following files/folders:
* `inventory.ini` where hosts must be added of removed depending on the desired cluster status.
* `group_vars` folder which contains different variables files to apply to each group. By default it only contains variables for `all` and `k8s_cluster` groups, but additional ones can be added according to the existing groups in `inventory.ini`.

## Deploy cluster

A playbook called `deploy-cluster.yml` that is responsible for deploying a Kubernetes cluster is provided. This playbook uses Kubespray `cluster.yml` under the hood.

```bash
ansible-playbook --user <user> --become --become-user=root deploy-cluster.yml
```

The rest of the cluster management in terms of adding and deleting nodes or resetting the cluster can be done directly using the playbooks provided by Kubespray in `submodules/kubespray`.

### Add node

```bash
ansible-playbook --user <user> --become --become-user=root submodules/kubespray/scale.yml
```

### Remove node/s

```bash
ansible-playbook --user <user> --become --become-user=root submodules/kubespray/remove-node.yml --extra-vars "node=nodename1,nodename2"
```

Remove node configuration from `inventory/<cluster-name>/hosts.yaml`.

### Reset cluster

```bash
ansible-playbook --user <user> --become --become-user=root submodules/kubespray/reset.yml
```

## GPU nodes

To be able to have GPU nodes on the cluster, the easiest and cleanest way is to use [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator) on top of Kubernetes.

By default, the use of GPU nodes is disabled. To enabled it, `nvidia_gpu_operator` variable can be set to true in `group_vars/k8s_cluster/k8s-cluster.yml`. It will automatically install NVIDIA GPU Operator with the deployment playbook [`deploy-cluster.yml`](https://github.com/jaime-cespedes-sisniega/k8s-cluster-deploy/blob/feature-optional-nvidia-gpu-operator/deploy-cluster.yml).

For a more manual management, a couple of simple playbooks are also provided:

* [`add-nvidia-gpu-operator.yml`](https://github.com/jaime-cespedes-sisniega/k8s-cluster-deploy/blob/main/add-nvidia-gpu-operator.yml) which uses [`nvidia-gpu-operator`](https://github.com/jaime-cespedes-sisniega/k8s-cluster-deploy/tree/main/roles/nvidia-gpu-operator) role.
* [`remove-nvidia-gpu-operator.yml`](https://github.com/jaime-cespedes-sisniega/k8s-cluster-deploy/blob/main/remove-nvidia-gpu-operator.yml) which removes operator´s pods.

GPU nodes are treated as normal CPU nodes, which allows to use the [Add Node](#add-node), [Remove Node](#remove-nodes) and [Reset Cluster](#reset-cluster) commands without any additional modification.

### Labeling GPU node (optional)

Node can be labeled to schedule desired pods. More information: [Assign Pods to Nodes](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/).
```bash
kubectl label nodes <node-with-gpu> accelerator=<nvidia-gpu-type>
```