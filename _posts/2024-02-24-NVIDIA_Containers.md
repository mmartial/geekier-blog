---
title: "Set up NVIDIA compatible containers"
date: 2024-02-24
tags:
- Linux
- Guide
- Docker
- Podman
- NVIDIA
- Ubuntu22
show_downloads: [false]
---

The NVIDIA GPU Container Runtime is a plugin that enables container platforms to securely access and manage NVIDIA GPUs as part of a containerized application environment.
Docker is an open-source platform that automates the deployment, scaling, and management of applications within lightweight, portable containers.
Podman is an open-source, daemonless container engine designed for developing, managing, and running OCI Containers, functioning as a drop-in replacement for Docker.
This post contains the setup instructions for NVIDIA GPU container toolkits on Linux hosts running Ubuntu 22.04 for docker and podman usage.

<h1>Setting up NVIDIA docker & podman</h1>

Revision: 20240302-0

- [1. Preamble](#1-preamble)
  - [1.1. Confirming the nvidia driver is available](#11-confirming-the-nvidia-driver-is-available)
  - [1.2. Docker setup (from docker.io)](#12-docker-setup-from-dockerio)
  - [1.3. Install podman (\> 4.1.0) on Ubuntu 22.04](#13-install-podman--410-on-ubuntu-2204)
- [2. NVIDIA Container Toolkit](#2-nvidia-container-toolkit)
  - [2.1. Docker](#21-docker)
  - [2.2. Podman](#22-podman)
- [3. Revision History](#3-revision-history)
  - [3.1. Contribute](#31-contribute)

Instructions for a Linux host running Ubuntu 22.04 to install the nvidia runtime for `docker` and `podman`.
We note that NVIDIA’s Container Toolkit officially only supports Ubuntu LTS release, but see [this](https://github.com/NVIDIA/nvidia-container-toolkit/issues/72) if your system is a 23.04 for example.

# 1. Preamble

The following are only required if you do not have some of the tools installed already.

## 1.1. Confirming the nvidia driver is available

The rest of this guide expects an already functional `nvidia-driver`. 

To install it :
- On Ubuntu Desktop, install from `Software & Updates`'s `Additional Drivers` and reboot. 
- On Ubuntu Server, confirm the device is available using `ubuntu-drivers devices` and install the `recommended` "server" driver: `sudo apt install nvidia-driver-535-server` then reboot.

To confirm it is functional, after a reboot, from a terminal,  run `nvidia-smi`; if a valid prompt shows up, you will have the information on the `Driver Version` and the supported `CUDA Version` for future running GPU-enabled containers.

## 1.2. Docker setup (from docker.io)

We will follow the instructions to set it up using the `apt` registry option.
Details can be found at https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

On a Ubuntu 22.04 system from a terminal:
- Clean up potential conflicting packages.
```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
- Add support for GPG keys, load Docker’s key, and set up the repository.

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
- Perform the package installation
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Confirm `docker` is functional, by checking if you get the `Hello from Docker!` message running its `hello-world`.
```bash
sudo docker run hello-world
```

- (Optional)  make `docker` is available without `sudo`, which has some security implications, as detailed in [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/). You will need to completely logout for the changes to take effect. Once this is done, you should be able to run `docker run hello-world` without the need of a `sudo`.

```bash
sudo usermod -aG docker $USER
```
## 1.3. Install podman (> 4.1.0) on Ubuntu 22.04

On Ubuntu 22.04, `apt search podman` returns version 3 of `podman`, we need a version of `podman` above 4.1.0 to be able to use the Container Device Interface (CDI) for `nvidia-container-toolkit`.

We recommend using [Homebrew](https://brew.sh/) to get a more recent version of `podman`.

To install `brew` (per [Homebrew](https://brew.sh/)'s instructions)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
, then follow the instructions in the `Next Steps` section of the installation process ("to add Homebrew to your PATH", ...).

The `brew` command and all the Hombebrew insallation are relative to its location of `/home/linuxbrew`, for example `/home/linuxbrew/.linuxbrew/bin/brew`.

Confirm `brew` is available in your PATH using `which brew` and is functional with `brew update`.

Install podman:
```bash
brew install podman
```
, will install many depdencies before installalling `podman`: 
```bash
==> Fetching dependencies for podman: bzip2, zlib, pcre2, expat, dbus, ncurses, readline, libxcrypt, util-linux, libffi, mpdecimal, ca-certificates, openssl@3, sqlite, xz, berkeley-db@5, libedit, krb5, libtirpc, libnsl, unzip, python@3.12, glib, libseccomp, libcap, lz4, zstd, systemd, conmon, yajl, crun, libfuse, fuse-overlayfs, gmp, libunistring, libidn2, libtasn1, nettle, p11-kit, libevent, libnghttp2, unbound, gnutls, libgpg-error, libassuan, libgcrypt, libksba, libusb, npth, openldap, libsecret, pinentry, gnupg, gpgme, libslirp and slirp4netns
[...]
==> Installing podman
==> Pouring podman--4.9.3.x86_64_linux.bottle.tar.gz
```

As you can see, at the time of this writeup we get version 4.9.3, which is above the minimum 4.1.0 required to be able to use the CDI.

In the `Caveats` section of the installation, it is recommended to `brew services start podman` to have `podman` ran as a service now and after reboot, which we want, so we recommend you run this command.

After doing so, we can check that `which podman` points us to `/home/linuxbrew/.linuxbrew/bin/podman`.

For `podman` to be functional ,we need to install `newuidmap` for "rootless mode" (Linux-native "fake root" for rootless containers) by 
```bash
sudo apt install rootlesskit
```

Now we can test podman:
```bash
podman run hello-world
```

`podman` runs similarly to `docker`; for example:
```bash
podman run --rm -it ubuntu:22.04 /bin/bash
```
will download `ubuntu:22.04` and give you a `bash` shell prompt in an interactive session, and will delete the created container when you exit the shell.

We note that `podman` has a `/home/linuxbrew/.linuxbrew/etc/containers/registries.conf` file already configured with `unqualified-search-registries=["docker.io"]`. More details on that topic at https://podman.io/docs/installation#registriesconf


# 2. NVIDIA Container Toolkit

For further details on what this supports, NVIDIA has a good primer document at https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/overview.html and 
complete instructions available at https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
On this page you will find details on:
- supported Linux distributions
NVIDIA driver requirements and minimal hardware supported
- Docker versions

Note that the NVIDIA Container Toolkit includes support for generating Container Device Interface (CDI)  for `podman` as well.

Installing the toolkit:
- Setup the package repository and the GPG keys:
```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
- Install the toolkit
```bash
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

## 2.1. Docker

- Configure `docker` to recognize the toolkit
```bash
sudo nvidia-ctk runtime configure --runtime=docker
```
- Restart `docker`
```bash
sudo systemctl restart docker
```
- Confirm `docker` (no `sudo` needed if you made the optional step in the last section) sees any GPU that you have running on your system by having it run `nvidia-smi`. Note that `docker` will need both `--runtime=nvidia` and `--gpus all` to use the proper runtime and have access to `all` the GPUs
```bash
docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

You can inspect your `/etc/docker/daemon.json` file to see that the `nvidia-container-runtime` is added:
```json 
[...]
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
```

To make this runtime the default, add the following content to the top of the file `"default-runtime": "nvidia",` (after the first `{`) and `sudo systemctl restart docker`. You should not have to add `--runtime=nvidia` to the CLI anymore.

## 2.2. Podman

For `podman` we will follow the instructions for Container Device Interface (CDI) setup from https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html

- Verify that the required tool is installed:
```bash
nvidia-ctk --version
```
, in this run we have `NVIDIA Container Toolkit CLI version 1.14.5`
- If successful, have it generate the CDI specifications
```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```
, at the end of this run, we have `INFO[0001] Generated CDI spec with version 0.5.0`
- Confirm devices show up (might be `0` and `all`)
```bash
grep "  name:" /etc/cdi/nvidia.yaml
```

Test `podman` (no `sudo` needed) using the proper devices `--device nvidia.com/gpu=all` 
```bash
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

# 3. Revision History

- 20240302-0: Introduction extension, grammar fixes.
- 20240225-0: Intitial release.
 
## 3.1. Contribute

If you see a problem, or simply want to contribute, this is a GitHub-based repo, feel free to send a PR.
