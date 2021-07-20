# Kontainerd 

![Kubeadm](https://img.shields.io/badge/-Kubeadm-326CE5?style=for-the-badge&logo=Kubernetes&logoColor=white)
![Vagrant](https://img.shields.io/badge/-Vagrant-1563FF?style=for-the-badge&logo=Vagrant&logoColor=white)
![Ansible](https://img.shields.io/badge/-ansible-C9284D?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/-ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

Creating a kubernetes cluster using Vagrant machines as nodes and `Containerd` as a container runtime
- This role is just an automation for the steps of aCloudGuru CKS Lesson **Building a Kubernetes Cluster**
- Parts of the roles are also borrowed from my previous projects for creating K8s cluster
    - [Kluster](https://github.com/theJaxon/Kluster)
    - [K8s-Vagrant](https://github.com/theJaxon/K8s-Vagrant)
- Vagrant [ansible local provisioner](https://www.vagrantup.com/docs/provisioning/ansible_local) is used to execute the roles on the target hosts

- Kubernetes version to be used can be modified by [changing the fact](https://github.com/theJaxon/Kontainerd/blob/main/kontainerd/tasks/prerequisites.yml#L40) inside kontainerd role 
```yaml
- set_fact:
    k8s_version: 1.20.1-00 # Change to whatever desired version
```
---

### Vagrant Machines details:

|  Machine |    Address    |         FQDN         |
|:--------:|:-------------:|:--------------------:|
|  master | 192.168.100.11 |  master master.com |
| worker | 192.168.100.10 | worker worker.com |

---

### How to use:
```bash
# Clone the repo
git clone https://github.com/theJaxon/Kontainerd.git

cd Kontainerd

# Start the machines 
vagrant up 

# SSH into any of the machines 
vagrant ssh < master | worker >
```

- One of the quirks i've faced was with installing kubernetes packages (kubeadm, kubectl and kubelet), the problem has to do with the order so i started first by installing `kubelet` which in turn installed kubectl (and this was breaking the installation since it was installing latest kubectl version not the version i'm specifying) so upon continuing the task another attempt to install kubectl is made with a downgraded version thus ansible errors. 
- The workaround was to change the sequence and start by installing the desired kubectl version 