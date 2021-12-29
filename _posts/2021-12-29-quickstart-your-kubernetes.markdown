---
layout: post
title:  "Quickstart your Kubernetes"
date:   2021-12-29 12:30:00 +0000
categories: kubernetes
author: Ben Ewen
---

Today I want to talk about Kubernetes. Yes! Kubernetes! That thing that everyone talks about but perhaps feels overwhelming to adopt. You might be wondering why bother adopting Kubernetes if you have an existing solution (e.g. Rancher, Docker Swarm, ECS etc).

Well - if you've already got your organisation's workloads running in containers, you're most of the way there. Kubernetes will complement your existing set up rather than fight it. In my experience, Kubernetes offers so much more than just running containers. It has standardised how we handle hundreds of operational tasks into a single ecosystem. Think, load balancers, routing/ingress, certificate management, security, garbage collection, rolling upgrades, planned node maintenance, storage management, and so much more!

And if you have it set up in one Kubernetes cluster, it's guaranteed to work anywhere else you deploy a cluster, which provides an amazing amount of freedom and decoupling between your applications and your infrastructure.

### How we deployed our first on-prem cluster

You've got multiple options these days for setting up a cluster -  any major public cloud such as AWS' EKS; or on-premise with kubeadm, k3s, or micro-k8s. We decided to go for k3s on-prem as this allowed us to use existing hardware without incurring any additional cost.

#### Requirements

- 3x VMs with at least:
  - 8 cores
  - 16GB RAM
  - 120GB Storage
  - Ubuntu 20.04 LTS
  - Internet connection
  - Static IP address assigned
- A reservable CIDR range on your internal network (excluding the static IPs of your VMs)

### Step 1 - Create the VMs

We use Nutanix to host our VMs, but you can use whatever your organisation uses, e.g. VMWare, Hyper-V. In our example, we used the following VMs:

- k8s-1 : 192.168.31.35
- k8s-2 : 192.168.31.36
- k8s-3 : 192.168.31.37

We set the user to `ubuntu` but you can use anything you like. We then have to allow sudo to run without repeating the password. To do this, run on each VM:

```sh
$ sudo visudo
```

```sh
# Then add to the bottom of the file
ubuntu ALL=(ALL) NOPASSWD: ALL
```

We set up static IPs with netplan by modifying the config file: `/etc/netplan/00-installer-config.yaml`. Your config may vary, check that the IP you assign in `addresses` is available on your network. Speak to your network administrator if you are unsure.

```sh
$ sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      addresses: [192.168.31.35/24]
      gateway4: 192.168.31.1
      nameservers:
        addresses: [192.168.10.10, 192.168.10.3, 192.168.30.35, 192.168.30.46]
```

We then applied the settings. Note you will lose connection to you current SSH session, so reconnect after with the new IP.

```sh
$ sudo netplan apply
```

Repeat for each VM, changing the IP each time.

### Step 2 - Install K3s

Now we have the VMs we can get to installing Kubernetes. To do this, we decided to use k3s, an extremely lightweight Kubernetes deployment developed by Rancher. Even better, there's a brilliant open source tool called [k3sup](https://github.com/alexellis/k3sup) that helps deploy it.

k3sup runs on your local machine, and bootstraps k3s to each of the VMs you specify. 

Before we use it, we need to share the local machine's SSH key with each VM so k3sup can authenticate (repeat for each VM):

```sh
# you don't have to run ssh-keygen if you already have an ssh key
$ ssh-keygen
# replace mykey wih your ssh key that exists in the ~/.ssh dir
# replace vm-ip with the VM ip
$ ssh-copy-id -i ~/.ssh/mykey ubuntu@vm-ip
```

Now to install k3sup on your local machine run:

```sh
$ curl -sLS https://get.k3sup.dev | sh
$ sudo install k3sup /usr/local/bin/
```

Now we can finally make the cluster! 

1. Pick one of the VMs to be the starting master by placing its ip in the `SERVER_IP` variable. Replace `ubuntu` as the `USER` if your user is named differently. Set the `--ssh-key` to be he key used earlier, e.g. `~/.ssh/mykey`.
    ```sh
    $ export SERVER_IP=192.168.31.35
    $ export USER=ubuntu

    $ k3sup install \
      --ip $SERVER_IP \
      --user $USER \
      --cluster \
      --k3s-version v1.19.1+k3s1 \
      --k3s-extra-args '--disable traefik --disable servicelb' \
      --ssh-key ~/.ssh/mykey
    ```

    Verify the cluster has been created by running:
    ```sh
    $ export KUBECONFIG=`pwd`/kubeconfig
    $ kubectl get node -o wide
    ```

    You should see your node:

    ```sh
    $ kubectl get nodes -o wide
    NAME    STATUS   ROLES         AGE   VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
    k8s-1   Ready    etcd,master   10s   v1.19.1+k3s1   192.168.31.35   <none>        Ubuntu 20.04.3 LTS   5.4.0-91-generic   containerd://1.4.0-k3s1
    ```

2. To join the other two VMs, run the `k3sup join` command below, being sure to replace the `--ip` argument with the 2nd and 3rd VM IPs.
    ```sh
    $ k3sup join \
      --ip 192.168.31.36 \
      --user $USER \
      --server-user $USER \
      --server-ip $SERVER_IP \
      --server \
      --k3s-version v1.19.1+k3s1 \
      --k3s-extra-args '--disable traefik --disable servicelb' \
      --ssh-key ~/.ssh/mykey
    ```

  Nice! We should now have a 3-node (nearly) high availability Kubernetes cluster!

  ```sh
  NAME    STATUS   ROLES         AGE   VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
  k8s-1   Ready    etcd,master   1m    v1.19.1+k3s1   192.168.31.35   <none>        Ubuntu 20.04.3 LTS   5.4.0-91-generic   containerd://1.4.0-k3s1
  k8s-2   Ready    etcd,master   30s   v1.19.1+k3s1   192.168.31.36   <none>        Ubuntu 20.04.3 LTS   5.4.0-91-generic   containerd://1.4.0-k3s1
  k8s-3   Ready    etcd,master   10s   v1.19.1+k3s1   192.168.31.37   <none>        Ubuntu 20.04.3 LTS   5.4.0-91-generic   containerd://1.4.0-k3s1
  ```

# Step 3 - Put your Kubernetes API behind a load balancer

If you look inside your `kubeconfig` you'll notice that it is using one of the IPs of the nodes. In an HA set up, if one of the nodes goes down, the cluster can continue to operate, but you won't be able to issue `kubectl` commands to it if the node that goes down is in your `kubeconfig`. To avoid having to change the `kubeconfig` in this scenario, you should place all three IPs behind a load balancer of your choice. Common examples are with HAProxy, NGINX, but any Layer 4 capable load balancer will suffice.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ...
    server: https://192.168.31.35:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: ...
```

The simplest set up that we used is a TCP network load balancer that balanced TCP port 6443 across 192.168.31.35-37. We then replaced `https://192.168.31.35:6443` in the `kubeconfig` to be the URL of the load balancer. If you use a Layer 7 load balancer, do not terminate TLS at the load balancer, be sure that it is passed-through as Kubernetes uses X509 Client Certs authentication via the TLS protocol. 

### Step 4 - Add a LoadBalancer provider to your cluster (MetalLB)

Not to be confused with the load balancer in step 3! An extremely common way of exposing services from within your cluster to the outside network is via load balancers. In the cloud, these load balancers are automatically deployed for you to the cloud (e.g. AWS ELB) when specifying `type: LoadBalancer` in your service definition. See more: https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer

To enable this in an on-prem installation, we want the cluster to be able to assign an IP on our internal network to a service within the cluster. To do this we can use [MetalLB](https://metallb.universe.tf/).

1. Install the MetalLB controller and namespace:
    ```sh
    $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
    $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
    ```
2. Create the config for MetalLB, this is where the reserved CIDR range is used. Be sure to specify an unused range of IPs on your internal network. Save the below config in a file called `metallb-config.yaml`. More advanced configuration options are available: https://metallb.universe.tf/configuration/
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 192.168.31.38-192.168.31.40 # your CIDR range goes here!
    ```
3. Apply the config to the cluster: `kubectl apply -f metallb-config.yaml`

### Step 5 - Trial it!

Everything should now be in place to be able to trial your cluster. In this example, we'll deploy a sample app to our cluster, and expose the service over a MetalLB load balancer. We'll then be able to access our sample app from an IP on our network!

1. Deploy a sample app: `kubectl create deployment hello-world --image=nginxdemos/hello`
2. Expose the deployment via a service with a `LoadBalancer`: `kubectl expose deployment hello-world --type=LoadBalancer --port=80`
3. Check the service now has an IP from our network: `kubectl get service hello-world`
    ```sh
    NAME          TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    hello-world   LoadBalancer   10.43.202.108   192.168.31.38   80:31869/TCP   3s
    ```
4. Go to http://192.168.31.38 (the external IP) and see your demo app!


### Outro

And that's it! If you've got a good basic understanding of networking and Linux, it's fairly easy to set up an on-prem high availability Kubernetes cluster.

Upcoming posts will show how you can take this set up further to include certificate management, using your own domain names, and Ingress controllers :D

I'll also be talking about cluster maintenance, and how easy it can be in an HA set up.