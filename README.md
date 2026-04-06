# Automate the installation of Ubuntu24.04 LTS Server and Ubuntu24.04 LTS Desktop using Cloud-Init and Subiquity technologies. 

## Abstract
 This document describes the steps to automate the installation of the Ubuntu 24.04.x LTS Server and Desktop using Cloud-Init and Subiquity technologies.   


## Introduction
  The Ubuntu24.04 LTS Server and Ubuntu24.04 LTS Desktop ISO uses Subiquity, which means that the
Subiquity Autoinstall format is used for automation.
* [Desktop Latest images](https://cdimage.ubuntu.com/daily-live/current/).
* [Server Latest images](https://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/).

For more information on Autoinstall, please see:
* [Automated Server Installs](https://ubuntu.com/server/docs/install/autoinstall)
* [Automated Server Install Quickstart](https://ubuntu.com/server/docs/install/autoinstall-quickstart)
* [Automated Server Installs Config File Reference](https://ubuntu.com/server/docs/install/autoinstall-reference)

## About Ubuntu24.04 Desktop automated installation  <br>
I saw a lot of similar technical documents, there is no way to realize the real automation process, they are all through the url way to specify the autoinstall.yaml file to achieve, after I tested and found that Ubuntu24.04 Desktop completely unattended fully automated installation, namely I found that Ubuntu24.04 Desktop can be fully automated unattended installation, that is, userdata and meta-data to answer and start the installer, only that during the automatic installation of Ubuntu24.04 Desktop, it still pops up the GUI installation interface, but it will be executed automatically, that is, after booting from PXE or IPXE, the installer will pop up automatically to finish automatically, and then it will finish automatically according to the file “userdata”. userdata” file, it will reboot automatically, and then you can see the ubuntu desktop.
You can also view the installation log during the whole automated installation process as shown below: <br>
   **1.start the installation program:** <br>
  ![image](https://github.com/user-attachments/assets/fc11a0f2-5d6f-4c92-b446-e19fcdc5847b)
   **2.Automatic reboot after installation is complete (takes about 20-30 minutes):** <br>
  ![image](https://github.com/user-attachments/assets/c00f85db-f245-4f7a-8183-5764e93fedb8)
   **3.after power on：** <br>
  ![image](https://github.com/user-attachments/assets/5ff1eda1-a7a2-4998-9824-bc9ee0a4e687)
  ![image](https://github.com/user-attachments/assets/36c4c3eb-e8df-4653-a5f2-c7f155d22891)
  ![image](https://github.com/user-attachments/assets/5e9129fb-77bd-4ed5-9e3f-43e5f7f97f25)






## Disclaimer:
 All operations and installation tests here require the host to be set to UEFI mode, the traditional BIOS mode is outdated and I don't want to look into it.
So, please do it in UEFI mode.

## 2026-04 Stability updates
- Added required NoCloud `meta-data` content (`instance-id` and `local-hostname`) for both server and desktop profiles.
- Escaped `;` in PXE `ds=nocloud-net\;s=...` kernel args to avoid parser truncation in GRUB-style chains.
- Removed duplicate `refresh-installer` key in `24.04_live_server/user-data` and set installer refresh to deterministic mode (`update: false`).
- Added UEFI-friendly storage setting `grub.reorder_uefi: false` in default profiles.
- Added an explicit UEFI+GPT+LVM sample profile for Issue #3:
  - `24.04_live_server/user-data-uefi-manual`
  - `24.04_Noble_Desktop/user-data-uefi-manual`


## Ubuntu24.04 LTS Desktop Autoinstall (user-data):
```yaml
#cloud-config
# See the autoinstall documentation at:
# https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html
autoinstall:
  refresh-installer:
    update: true
    channel: latest/edge
  apt:
    disable_components: []
    fallback: abort
    geoip: false
    mirror-selection:
      primary:
      - country-mirror
      - arches: &id001
        - amd64
        - i386
        uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
      - arches: &id002
        - s390x
        - arm64
        - armhf
        - powerpc
        - ppc64el
        - riscv64
        uri: http://ports.ubuntu.com/ubuntu-ports
    preserve_sources_list: false
    security:
    - arches: *id001
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
    - arches: *id002
      uri: http://ports.ubuntu.com/ubuntu-ports
  codecs:
    install: true
  drivers:
    install: true
  kernel:
    package: linux-generic-hwe-24.04
  keyboard:
    layout: us
    variant: ''
  locale: en_US.UTF-8
  network:
    version: 2
    ethernets:
        ens33:
            match:
                name: "en*"
            dhcp4: true
        eth1:
            match:
                name: "eth*"
            dhcp4: true
  oem:
    install: auto
  source:
    id: ubuntu-desktop-minimal
    search_drivers: true
#  ssh:
#    allow-pw: true
#    authorized-keys: []
#    install-server: true
  storage:
    layout:
      name: lvm
      sizing-policy: all
  identity:
    realname: ''
    hostname: ubuntu-desktop
    username: ubuntu
    password: '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0'
  timezone: Asia/Shanghai
  updates: security
  late-commands:
#    - curtin in-target --target /target -- apt update -y
#    - curtin in-target --target /target -- apt upgrade -y
#    - curtin in-target --target /target -- apt-get dist-upgrade -y
#    - curtin in-target --target /target -- apt autoremove -y
#    - chmod -R 600 /target/etc/netplan/
    - curtin in-target --target=/target -- sudo usermod -p '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0' root
    - reboot
  version: 1
```
## Ubuntu24.04 LTS Server Autoinstall(user-data):
```yaml
#cloud-config
autoinstall:
  refresh-installer:
    update: true
    channel: latest/edge
  apt:
    disable_components: []
    fallback: abort
    geoip: false
    mirror-selection:
      primary:
      - country-mirror
      - arches: &id001
        - amd64
        - i386
        uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
      - arches: &id002
        - arm64
        uri: http://ports.ubuntu.com/ubuntu-ports
    preserve_sources_list: false
    security:
    - arches: *id001
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
    - arches: *id002
      uri: http://ports.ubuntu.com/ubuntu-ports
  packages:
    - git
    - curl
    - wget
    - vim 
    - build-essential
  drivers:
    install: true
  refresh-installer:
    channel: edge
    update: yes
  storage:
    layout:
      name: lvm
      sizing-policy: all
  user-data:
    disable_root: false
  identity:
    realname: ''
    hostname: ubuntu
    username: ubuntu
    password: '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0'
  kernel:
    package: linux-generic
  keyboard:
    layout: us
    toggle: null
    variant: ''
  locale: en_US.UTF-8
  network:
    version: 2
    ethernets:
        ens33:
            match:
                name: "en*"
            dhcp4: true
        eth1:
            match:
                name: "eth*"
            dhcp4: true
  oem:
    install: auto
  snaps:
  - channel: stable
    classic: true
    name: powershell
  source:
    id: ubuntu-server
    search_drivers: true
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
  updates: security
  timezone: Asia/Shanghai
  late-commands:
    - curtin in-target --target /target -- apt update -y
    - curtin in-target --target /target -- apt upgrade -y
    - curtin in-target --target /target -- apt-get dist-upgrade -y
    - curtin in-target --target /target -- apt autoremove -y
    - curtin in-target --target=/target -- /usr/bin/wget -P /etc/netplan/ http://172.17.80.238/ubuntu/netcfg/server-netcfg.yaml
    - chmod -R 600 /target/etc/netplan/
    - curtin in-target --target=/target -- sudo usermod -p '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0' root
    - curtin in-target --target=/target -- update-grub2
    - reboot
  version: 1
```
  ### PXE&iPXE BOOT Config
  1. Ubuntu24.04 LTS Desktop PXE&iPXE BOOT Config
     ```yaml
     set base-url http://172.17.100.221/iso/ubuntu/24.04_desktop/24.04_Noble_Desktop
     kernel ${base-url}/casper/vmlinuz
     initrd ${base-url}/casper/initrd
     set ubuntu_iso_url ${base-url}/noble-desktop-amd64.iso
     imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=8388608 cloud-config-url=/dev/null url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     #imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=8388608 url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     boot || goto failed
     ```
  2. Ubuntu24.04 LTS Server PXE&iPXE BOOT Config
     ```yaml
     set base-url http://172.17.100.221/iso/ubuntu/24.04_live_server
     kernel ${base-url}/casper/vmlinuz
     initrd ${base-url}/casper/initrd
     set ubuntu_iso_url ${base-url}/noble-live-server-amd64.iso
     #imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=8388608 url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     imgargs vmlinuz initrd=initrd ip=dhcp cloud-config-url=/dev/null url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     boot || goto failed
     ```
 ### Other
   1. VMware virtual machines and configurations used for testing:
      ![image](https://github.com/user-attachments/assets/056d860c-12a9-4e99-8ce8-14c1c8f6fbc8)
      ![image](https://github.com/user-attachments/assets/12df8197-2219-45c0-a100-e341995960f4)


  2. PXE boot, boot preview:
    ![image](https://github.com/user-attachments/assets/fec18617-a856-4da8-b7aa-87eb1edef5db)

  3. 24.04_live_server Preview of system files and directories at deployment：
      ![image](https://github.com/user-attachments/assets/53763127-7a58-4f45-9f17-fe9ef858e5b6)
     
   4. 24.04_Noble_Desktop Preview of system files and directories at deployment：
     ![image](https://github.com/user-attachments/assets/07e5a623-8418-42ea-a26b-a53be429da25)

  5. ........
     
 ### Issues Summary
  1. When I changed the apt installation source to my own internal address: “http://172.17.80.238/ubuntu/”, I found that the installation would sometimes fail, which could be due to a problem with the installation source server that I deployed internally using “apt mirror”, for example, the installation source failed to download some files. This may be due to a problem with the installation source server I deployed using “apt mirror” internally, for example, some files in the installation source failed to download, causing the installation source to be incomplete.
     Configuring Ubuntu Local Repository (offline APT mirror) with “apt mirror” should be problematic, I see that the “apt-mirror” project ( I see that the “apt-mirror” project ( https://github.com/apt-mirror/apt-mirror ) hasn't been updated for a long time, and it may not support ubuntu 22.04-ubuntu 24.04 well, hopefully some awesome developer will fix these problems and help more users.
  
  3. Older computers are not supported。
     I in the VMware virtual machine to test automated ubuntu 24.04 desktop installation, the configuration of the virtual machine (16G + 16CPU + 100G HD), found that in the VMware virtualization environment can be installed successfully; but I changed to a laptop (my laptop is older, the model is Lenovo ThinkPad X260, it is) The configuration of my laptop is (cpu: i3-6100u, Memory: 8G; Samsung 870 SSD 500G), let the laptop boot from PXE, and finally the installer is stuck at “PreparingUbuntu.....”. I've tested this several times, and this is what happens. I'm analyzing that my laptop is too low-configured, and that ubuntu 24.04 doesn't support older computers when automating deployments with Cloud-Init and Subiquity?
In this case, I tested it on my newer DELL3640 Desktop computer (my DELL3640 desktop computer configuration: cpu:i5, Memory: 16G; Samsung870 SSD 500G), and found that my DELL desktop computer can also be installed normally, so what's the problem
![image](https://github.com/user-attachments/assets/0b73af60-9676-416e-95a4-7c71752f8faa)



