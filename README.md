# EVPN-VXLAN Quickstart based on AVD + Containerlab

> **WARNING**  
> Please read the guide before you start using AVD Quickstart Containerlab.
> Make sure that you understand the consequences of running Containerlab, cEOS-lab and various scripts provided with this repository on your machine. The components of the lab may change your system settings as they will have super user privileges.



- EVPN-VxLAN Quickstart
  - [Overview](#overview)
  - [Lab Requirements](#lab-requirements)
  - [MacOS Limitations and Required Settings](#macos-limitations-and-required-settings)
  - [Release Notes](#release-notes)
  - [How To Use The Lab](#how-to-use-the-lab)
  - [How To Destroy The Lab](#how-to-destroy-the-lab)
  - [Containerlab Scalability with cEOS](#containerlab-scalability-with-ceos)

## Overview

The AVD Quickstart repository is a collection of Arista EOS labs based on [Containerlab](https://containerlab.srlinux.dev/) that you can build on any machine with Docker in a few minutes.
The ultimate target of this repository is to provide a **portable Arista lab collection** for everyone. This collection can be used to learn and test certain Arista EOS features and in certain cases even build configs for production environment with the excemption of hardware features.

Some labs provided in this repository can be used with CloudVision Portal VM that must be deployed separately. But all labs that are not focused on CVP features can be used without such VM as it is quite resource intensive and can not be deployed on an avarage laptop for example.

> **WARNING**: if CVP VM is part of the lab, make sure that it's reachable and credentials configured on CVP are matching the lab.

The initial lab list provided in this repository is focused on learning and testing [AVD](https://avd.sh/en/latest/).
Some labs can be easily adjusted to your needs using simplified CSV and YAML inputs.

Currently following labs are available:

- AVD repository to build EVPN MLAG network
- AVD repository to build EVPN Active-Active network

## Lab Requirements

A machine with Docker CE or Docker Desktop is required.
Following operating systems were tested:

 - The lab is expected to run on any major Linux distribution.

Hardware requirements depend on the number of containers deployed. Please read [Containerlab Scalability with cEOS](#containerlab-scalability-with-ceos) section before deploying a large topology.
For a small topology of 10+ cEOS containers 8 vCPUs and 16 GB RAM are recommended.

> **WARNING**: Please make sure that your host has enough resorces. Otherwise Containerlab can enter "frozen" state and require Docker / host restart.

To install Docker on a Linux machine, check [this guide](https://docs.docker.com/engine/install/ubuntu/).
To get Docker Desktop, check [docker.com](https://www.docker.com/products/docker-desktop/).

If you are planning to deploy Containerlab on a dedicated Linux host, you can also install and configure KVM and deploy CloudVision Portal as a virtual machine. To install KVM, check [this guide](https://github.com/arista-netdevops-community/kvm-lab-for-network-engineers) or any other resource available on internet. Once KVM is installed, you can use one of the following repositories to install CVP:

- ISO-based KVM installer - currently not available on Github and distributed under NDA only. That will be fixed later.
- [CVP KVM deployer](https://github.com/arista-netdevops-community/cvp-kvm-deployer)
- [CVP Ansible provisioning](https://github.com/arista-netdevops-community/cvp-ansible-provisioning)

It is also possible to run CVP on a dedicated host and a different hypervisor as long as it can be reached by cLab devices.

> NOTE: to use CVP VM with container lab it's not required to recompile Linux core. That's only required if you plan to use vEOS on KVM for you lab setup.

The lab setup diagram:

![lab diagram](media/lab_setup.png)



## How To Use The Lab

This section is explaining basic AVD quickstart lab workflow.

1. Clone AVD quickstart repository to your lab host: `git clone https://github.com/marekplaza/evpnvxlan_quickstart.git` Or use your favorite IDE (like VSCode) for that.
2. It is recommended to remove git remote as changes are not supposed to be pushed to the origin: `git remote remove origin`
3. Change to the lab directory: `cd evpnvxlan_quickstart`. IDE should do that for you automatically.
4. Before running the lab it is recommended to create a dedicated git branch for you lab experiments to keep original branch clean.
5. Check makefile help for the list of commands available: `make help`

```bash
[ ApiusLAB 🧪 ] # make
avd_build_eapi                 Build configs and configure switches via eAPI: ansible-playbook playbooks/fabric-deploy-eapi.yml 
avd_snapshot                   Snapshot: ansible-playbook playbooks/snapshot.yml
avd_validate                   Validate states: ansible-playbook playbooks/validate-states.yml
build                          Build docker image
clab_deploy                    Deploy Containerlab
clab_destroy                   Destroy Containerlab
clab_graph                     Build Containerlab graph
clab_inspect                   Inspect Containerlab
clean                          Remove all Containerlab files and directories
help                           Display help message
inventory_evpn_aa              Generate inventory for EVPN AA
inventory_evpn_mlag            Generate inventory for EVPN MLAG
run                            Run docker image
[ ApiusLAB 🧪 ] # 
```

6. If cEOS image does not exist on your host yet, download it from [arista.com](https://www.arista.com/) and import. For example: `docker import <path-to-you-download-location>/ cEOS-lab-4.30.4M.tar ceos-lab:4.30.4M`. The image tag must match the parameters defined in the lab files, for example `CSVs_EVPN_AA/clab.yml` or `CSVs_EVPN_MLAG/clab.yml`. You can use `eos-downloader` which is included to this container. To do such use `make run` and execute command as below:


7. Use `make build` to build `avd-quickstart:latest` container image. If that was done earlier and the image already exists, you can skip this step. If you are using VSCode devcontainer, VSCode will do that for you automatically.
8. (Optional) If VSCode devcontainer is used, the container will start automatically. Otherwise you can start it manually in the interactive mode by entering `make run`. You can also skip this steps and execute all commands listed below directly on your host. They will run inside a non-interactive container in that case.
9. Build lab inventory with `make inventory_evpn_aa` (EVPN Active-Active scenario) or `make inventory_evpn_mlag` (EVPN MLAG scenario). This will create a directory with all files required to build the lab.
10. Review the inventory generated by AVD quickstart. Optional: you can git commit the changes.
11. Run `make clab_deploy` to build the containerlab. Wait until the deployment will finish.
12. If you are working with `avd-quickstart:latest` container in the interactive mode, you can add aliases to connect to the lab devices quickly: `add_aliases`. This will allow you to connect to any lab device by typing it's short hostname. For example, `leaf1` will SSH to the switch with the corresponding hostname. You can list generated aliases with `alias` command. This can be especially useful on MacOS, as `/etc/hosts` file can not be changed by Containerlab due to system integrity check restrictions.

![connect to a device from the container](media/connect-to-device.png)

13. If CVP VM is used in the lab, onboard cLab switches with `make onboard`. Once the script behind this shortcut wil finish, devices will appear in the CVP inventory.
14. To build EVPN configuration and deploy it on the lab switches, you can execute corresponding Ansible AVD playbook with `make avd_build_eapi` (deploy directly) or `make avd_build_cvp` (deploy via CVP if it's present in the lab) shortcut. That will execute `playbook/fabric-deploy-eapi.yml` or `playbook/fabric-deploy-cvp.yml` playbooks. If the configuration is delivered via CVP, don't forget to execute the tasks generated by the playbook.
15. Once the configs are deployed, you can SSH to any switch using corresponding alias and type any show commands to check the lab state. Verify that hosts can ping each other.
16. To test Ansible AVD post-validation role, use `make avd_validate`. The playbook itself is located at `playbooks/validate-states.yml`. This will generate corresponding reports in the lab directory.
17. You can also collect as snapshot (series of pre-defined show commands) with `make avd_snapshot`.
18. You can optionally git commit the changes and start playing with the lab. Use CSVs to add some VLANs, etc. for example. Re-generate the inventory and check how the AVD repository data changes.

## How To Destroy The Lab

1. Execute `make clab_destroy` to destroy the containerlab.
2. Execute `make rm` or `make clean` to delete the generated AVD inventory.

## Containerlab Scalability with cEOS

> under construction, coming soon
