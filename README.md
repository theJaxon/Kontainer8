# Kontainer8

![Kubeadm](https://img.shields.io/badge/-Kubeadm%201.24.0-326CE5?style=for-the-badge&logo=Kubernetes&logoColor=white)
![Vagrant](https://img.shields.io/badge/-Vagrant-1563FF?style=for-the-badge&logo=Vagrant&logoColor=white)
![Ansible](https://img.shields.io/badge/-ansible-C9284D?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/-ubuntu%2022.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)


![Ingress](https://img.shields.io/badge/nginx-ingress%20controller-269539?style=for-the-badge&logo=Nginx)
![Locl-path-provisioner](https://img.shields.io/badge/Dynamic%20provisioning-local%20path%20provisioner-0075A8?style=for-the-badge&logo=Rancher)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Vagrant Machines details](#vagrant-machines-details)
- [How to use](#how-to-use)
- [Locally building images](#locally-building-images)
- [Extras](#extras)
- [Useful Resources](#useful-resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Creating a kubernetes cluster using Vagrant machines as nodes and `Containerd` as a container runtime
- This role is just an automation for the steps of aCloudGuru CKS Lesson **Building a Kubernetes Cluster**
- Parts of the roles are also borrowed from my previous projects for creating K8s cluster
    - [Kluster](https://github.com/theJaxon/Kluster)
    - [K8s-Vagrant](https://github.com/theJaxon/K8s-Vagrant)
- Vagrant [ansible local provisioner](https://www.vagrantup.com/docs/provisioning/ansible_local) is used to execute the roles on the target hosts

- Kubernetes version to be used can be modified by [changing the fact](https://github.com/theJaxon/Kontainerd/blob/main/kontainerd/tasks/prerequisites.yml#L40) inside kontainerd role 
```yaml
- set_fact:
    k8s_version: 1.24.0-00 # Change to whatever desired version
```
---

### Vagrant Machines details

|  Machine |    Address    |         FQDN         |
|:--------:|:-------------:|:--------------------:|
|  master | 192.168.100.11 |  master master.com |
| worker | 192.168.100.10 | worker worker.com |

---

### How to use
```bash
# Clone the repo
git clone https://github.com/theJaxon/Kontainer8.git

cd Kontainerd

# Start the machines 
vagrant up 

# SSH into any of the machines 
vagrant ssh < master | worker >
```

---

### Locally building images

- Start by installing podman
```bash
# Assuming there's a Dockerfile in the current working directory
podman build --tag jenkins-local .

# Save the image into tar file
podman save jenkins-local -o jenkins-local.tar

# Use ctr to import the image 
sudo ctr -n=k8s.io images import jenkins-local.tar

# Verify that the image is now available for k8s to use 
sudo crictl image ls

> localhost/jenkins-local
``` 

---

### Accessing the Ingress from the Host OS

- Nginx Ingress controller is configured to use **nodeport 30000** so that we end up with a fixed port number for the controller
- Using the controller as a proxy can be done by calling any of the 2 machines 
```bash
curl --proxy http://192.168.100.10:30000 http://jellyfin.media/web/index.html
curl --proxy http://192.168.100.11:30000 http://jellyfin.media/web/index.html
```
- On the host OS one can take advantage of Firefox by setting the proxy to point to the ingress controller 

![Firefox_proxy](https://github.com/theJaxon/Kontainer8/blob/main/etc/Firefox_Proxy.jpg)

- You can take this one step further and install a plugin like [Proxy Toggle](https://addons.mozilla.org/en-US/firefox/addon/proxy-toggle-button/) to easily use the controller and revert back to the regular browser settings
- Once the Ingress controller is set up as the proxy, you can reach the needed services using the configured ingress
![Jellyfin](https://github.com/theJaxon/Kontainer8/blob/main/etc/Jellyfin.jpg)



---

### Extras
- Extras role defines additional resources that will be deployed to the kubernetes cluster, it's responsible for creating 2 new namespaces
  1. ingress-nginx - where nginx ingress controller will be deployed
  2. local-path-storage - where local path provisioner will be deployed (local path is the default storage class)

---

- One of the quirks i've faced was with installing kubernetes packages (kubeadm, kubectl and kubelet), the problem has to do with the order so i started first by installing `kubelet` which in turn installed kubectl (and this was breaking the installation since it was installing latest kubectl version not the version i'm specifying) so upon continuing the task another attempt to install kubectl is made with a downgraded version thus ansible errors. 
- The workaround was to change the sequence and start by installing the desired kubectl version 

---

### Useful Resources
- [ justmeandopensource/kubernetes - vagrant-provisioning ](https://github.com/justmeandopensource/kubernetes/tree/master/vagrant-provisioning)
- [Upgrading an existing cluster with kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/coredns/#migrating-to-coredns)
- [Manually Loading Container Images with containerD](https://blog.scottlowe.org/2020/01/25/manually-loading-container-images-with-containerd/)
- [Local Path Provisioner](https://github.com/rancher/local-path-provisioner)
- [Kubernetes nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)