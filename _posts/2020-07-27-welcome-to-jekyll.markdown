---
layout: post
title:  "Deploying K3s on Raspbian"
date:   2020-04-19 16:47:30 -0700
categories: k8s kubernetes pi raspbian
---

## Preparing the Raspberry Pis

I'm currently using two [Raspberry Pi 3 B+](https://www.amazon.com/gp/product/B07BDR5PDW/) 

### Hostnames

K3s requires that all machines have different hostnames, so the first step I took was to update the hostname on each Raspberry Pi to something unique.

There are a couple files that include the hostname of the Pi. The first one can be found at `/etc/hostname`. This file only contains one line. Edit the file using `sudo`

```bash
$ sudo vi /etc/hostname
```

The second file is located at `/etc/hosts`. Find the starting line with the private address (10.X.X.X, 192.168.X.X) and update the hostname next to that address.

```bash
$ sudo vi /etc/hosts
```

For the changes to take place you'll need to reboot the Pi's.

```bash
$ sudo reboot
```

### Enabling cgroup memory

It's possible that the OS doesn't have cgroup memory enabled out of the box. The server node and agent rely on this to boot properly. To verify this open `/boot/cmdline.txt`.

```bash
$ sudo /vim/cmdline.txt
```

If you don't see the following key-values in `/boot/cmdline.txt`, add them:

```
cgroup_memory=1 cgroup_enable=memory
```

At this point you should be ready to spin up the Kubernetes cluster

## Initializing K3s

### Server Node

The first step to setting up your cluster is to set up the server node. To install, run the following:

```bash
$ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

The reason for setting the file permission to `644` is because most `kubectl` commands have to be ran with root permissions. The default configuration from the script prevents all features from working within `kubectl`. To get around this, the flag `--write-kubeconfig-mode` needs to be utilized passing the appropriate permissions.

The script will start a service that can be verified using `systemctl`

```bash
$ systemctl status k3s.service
```

At this point the server node should be set up. The URL of the machine and the K3s token need to be recorded to set up the worker nodes. The token can be retrieved by running the following:

```bash
$ sudo cat /var/lib/rancher/k3s/server/node-token
```

### Worker Nodes

Setting up the worker nodes is very similar to setting up the server node. The same script needs to be ran with the `K3S_URL` and `K3S_TOKEN` environment variables.

`K3S_URL` requires that the URL starts with "https". For example, if you're setting up the cluster using private addresses, the environment variable will be set to something like `https://192.168.X.X:6443`. 

```bash
$ curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

You've deployed a Kubernetes cluster using K3s. You can validate that everything okay by running:

```bash
$ kubectl get nodes
```

You should see your server node and workers with this output. My output looks like the following:

```bash
$ kubectl get nodes
NAME         STATUS   ROLES    AGE    VERSION
worker       Ready    <none>   12m   v1.17.4+k3s1
servernode   Ready    master   14m   v1.17.4+k3s1
```

## What's Next?

Looking for something to deploy to your cluster? Look into a Kubernetes Dashboard so you can monitor your cluster.

[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)