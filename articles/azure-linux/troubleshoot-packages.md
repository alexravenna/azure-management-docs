---
title: Troubleshooting Azure Linux Container Host for AKS package upgrade issues
description: How to troubleshoot Azure Linux Container Host for AKS package upgrade issues.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: troubleshooting
ms.date: 08/18/2024
# Customer intent: "As a system administrator managing Azure Linux Container Hosts for AKS, I want to troubleshoot package upgrade issues, so that I can ensure that all necessary updates are applied and the system remains stable and secure."
---

# Troubleshoot issues with package upgrades on the Azure Linux Container Host

The Azure Linux Container Host for AKS has `dnf-automatic` enabled by default, a systemd service that runs daily and automatically installs any recently published updated packages. This ensures that packages in the Azure Linux Container Host should automatically update when a fix is published. Note, that for some settings of [Node OS Upgrade Channel](/azure/aks/auto-upgrade-node-image), `dnf-automatic` will be disabled by default.

## Symptoms

However, sometimes the packages in the Azure Linux Container Host fail to receive automatic upgrades, which can lead to the following symptoms:
- Error messages while referencing or using an updated package.
- Packages not functioning as expected.
- Outdated versions of packages are displayed when checking the Azure Linux Container Host package list. You can verify if the packages on your image are synchronized with the recently published packaged by visiting the repository on [packages.microsoft.com](https://packages.microsoft.com/cbl-mariner/) or checking the release notes in the [Azure Linux GitHub](https://github.com/microsoft/CBL-Mariner/releases) repository.

## Cause

Some packages, such as the Linux Kernel, require a reboot for the updates to take effect. To facilitate automatic reboots, the Azure Linux VM runs the check-restart service, which creates the `/var/run/reboot-required` file when a package update requires a reboot.

## Solution

To ensure that Kubernetes acts on the request for a reboot, we recommend setting up the [kured daemonset](/azure/aks/node-updates-kured). [Kured](https://github.com/kubereboot/kured) monitors your nodes for the `/var/run/reboot-required` file and, when it's found, drains the work off the node and reboots it.

## Next steps

If the preceding steps do not resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
