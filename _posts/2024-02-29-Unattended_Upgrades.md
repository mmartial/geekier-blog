---
title: "Unattended Upgrades"
date: 2024-02-29
tags:
- Linux
- Guide
- Ubuntu22
show_downloads: [false]
---

"[Unattended Upgrades](https://manpages.ubuntu.com/manpages/jammy/man8/unattended-upgrade.8.html)" is a package available on [Ubuntu](https://ubuntu.com/) systems that automatically installs updates for security and, optionally, other software packages. 
This tool is crucial for maintaining system security and stability by ensuring that vulnerabilities and bugs are promptly addressed without requiring manual intervention.
This post details the setup instructions for using it for security updates on an [Ubuntu Linux 22.04 server](https://ubuntu.com/server) and sending emails on completion.

<h1>Unattended Upgrades</h1>

Revision: 20240302-0

- [1. Preamble](#1-preamble)
  - [1.1. Ubuntu Pro](#11-ubuntu-pro)
- [2. Unattended upgrades](#2-unattended-upgrades)
  - [2.1. Initial setup](#21-initial-setup)
  - [2.2. Optional setup](#22-optional-setup)
    - [2.2.1. email](#221-email)
    - [2.2.2. automatic reboot](#222-automatic-reboot)
  - [2.3. Testing](#23-testing)
    - [2.3.1. Manual updates](#231-manual-updates)
- [3. Revision History](#3-revision-history)
  - [3.1. Contribute](#31-contribute)

Instructions for a Linux host running Ubuntu 22.04 server to set up unattended upgrades, so that your server will automatically upgrade security updates (only).
Optionally, we will request the tool to send us emails each time it runs, and in particular to let us know if a reboot of the server is needed. Another blog post was added on the same original date as this post on the topic of using FastMail with Postfix to send emails.

# 1. Preamble

"Unattended Upgrades" on Ubuntu is configured to automatically install security updates only. This ensures that the system receives important security patches without manual intervention, helping to keep the system secure against vulnerabilities. 
While it can be configured to update a broader range of packages, doing so may increase the risk of introducing stability issues with automatic updates of non-security critical packages.

These instructions will enable the end user to have security updates (only) done automatically. 

## 1.1. Ubuntu Pro

If your system runs Ubuntu Pro, some additional security packages might get installed.

[Ubuntu Pro](https://ubuntu.com/pro) requires a Ubuntu account and is free for up to five systems. It is a subscription-based service offered by Canonical, providing enhanced security and compliance features for Ubuntu users, including extended security maintenance (ESM) for applications and infrastructure, patching for high and critical Common Vulnerabilities and Exposures (CVEs) for supported packages, and additional compliance certifications for regulated industries or sensitive environments. If you have it enabled, go to [https://ubuntu.com/pro/dashboard](https://ubuntu.com/pro/dashboard) and look at the "Command to attach a machine" (in the form of `sudo pro attach TOKEN`) to enable it. When run, you will be prompted with additional details on the different services enabled.

# 2. Unattended upgrades

## 2.1. Initial setup

Ubuntu's original writeup is at [https://help.ubuntu.com/community/AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

```bash
# Install needed packages
sudo apt install -y unattended-upgrades apt-listchanges
# accept the choices given to you in the interactive dialogue

# enable the automatic updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
# select "yes" to "Automatically download and install stable updates?"

# (optional) make it possible to reboot automatically
sudo apt install -y update-notifier-common
```

You can confirm by checking `cat /etc/apt/apt.conf.d/20auto-upgrades` which should contain
```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

You can `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and see the list of options to customize the system to work best for your needs.

## 2.2. Optional setup

### 2.2.1. email

`sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` find the `Unattended-Upgrade::Mail` line and uncomment it (remove the `//` at the beginning of the line) and set the destination email, such as `test@gmail.com`. Note that mail sending needs to be functional on your host for this to work. The final line will look something like:
```bash
Unattended-Upgrade::Mail "test@gmail.com";
```

### 2.2.2. automatic reboot

In the `/etc/apt/apt.conf.d/50unattended-upgrades`, find, uncomment and adapt the following lines according to your needs:

```bash
// Automatically reboot *WITHOUT CONFIRMATION* if
//  the file /var/run/reboot-required is found after the upgrade
//Unattended-Upgrade::Automatic-Reboot "false";

// Automatically reboot even if there are users currently logged in
// when Unattended-Upgrade::Automatic-Reboot is set to true
//Unattended-Upgrade::Automatic-Reboot-WithUsers "true";

// If automatic reboot is enabled and needed, reboot at the specific
// time instead of immediately
//  Default: "now"
//Unattended-Upgrade::Automatic-Reboot-Time "02:00";
```

## 2.3. Testing

You can see that your configuration file is functional by running `sudo unattended-upgrades --dry-run`

After confirming it is, use `sudo unattended-upgrades -v` to run the tool for the first time and confirm all is functional.
If any update was performed, and you have email notifications setup, you will get an email with the details of the operation.
It is up to you to act on `Warning: A reboot is required to complete this upgrade, or a previous one` notification in the email's content.

### 2.3.1. Manual updates

As noted earlier, our setup is configured to automatically install security updates only.

When your system informs you that it requires a reboot, this might be an opportune time to run `sudo apt-get update`, `sudo snap refresh --list`, `brew update`, ... as needed.


# 3. Revision History

- 20240302-0: Added links to the introduction section.
- 20240229-0: Intitial release.
 
## 3.1. Contribute

If you see a problem or simply want to contribute, this is a GitHub-based repo, feel free to send a PR.
