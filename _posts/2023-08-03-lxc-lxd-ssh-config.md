---
layout: post
title:  "LXD/LXC SSH configuration"
date:   2023-08-03 14:00:00 +0200
categories: container lxc lxd
---

# Configuring ssh config to connect to LXD/LXC running containers:

## Enable your public keys on container ssh access point

### On a running container

Add your ssh public key to the running container you want to connect to:

```bash
lxc file push ~/.ssh/id_<xxx>.pub u1/home/ubuntu/.ssh/authorized_keys
```

### On a dedicated profile

You can add your SSH public key into an LXD profile. To do so on the `default` profile:

```bash
lxc profile edit default
```

And add your public SSH key (found in `~/.ssh/id_<xxx>.pub`):

```conf
config:
  user.user-data: |
    #cloud-config
    ssh_authorized_keys: <place your ssh public key here>
```

## Make SSH connection even smoother

Add a dedicated helper script for lxd ssh connections:

```bash
mkdir -p ~/Developments/canonical/scripts/
cd ~/Developments/canonical/scripts/
wget https://gist.githubusercontent.com/basak/72b87a5b619a100ace1476715bfc5b18/raw/d716f711a152a71d9a0eaae5fb932f83ff1c03d7/lxd-ssh.sh
chmod +x lxd-ssh.sh
sudo ln -s ~/Developments/canonical/scripts/lxd-ssh.sh /usr/local/bin/lxd-ssh
```

Update your `~/.ssh/config` :

```conf
Host *.lxd
  User ubuntu
  ProxyCommand lxd-ssh %h
  StrictHostKeyChecking no
``````

Then you can connect on running container u1 with:

```bash
ssh u1.lxd
```

## Going further

It might be interesting to integrate https://github.com/dustinkirkland/ssh-import-id for the public key importation.