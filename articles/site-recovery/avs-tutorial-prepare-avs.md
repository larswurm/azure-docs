---
title: Prepare Azure VMware Solution for disaster recovery to Azure Site Recovery
description: Learn how to prepare Azure VMware Solution servers for disaster recovery to Azure using the Azure Site Recovery service.
author: ankitaduttaMSFT
manager: rochakm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/29/2020
ms.author: ankitadutta
ms.custom: MVC

---
# Prepare Azure VMware Solution for disaster recovery to Azure Site Recovery

This article describes how to prepare Azure VMware Solution servers for disaster recovery to Azure using the [Azure Site Recovery](site-recovery-overview.md) services. 

This is the second tutorial in a series that shows you how to set up disaster recovery to Azure for Azure VMware Solution VMs. In the first tutorial, we [set up the Azure components](avs-tutorial-prepare-azure.md) needed for Azure VMware Solution disaster recovery.


In this article, you learn how to:

> [!div class="checklist"]
> * Prepare an account on the vCenter Server to automate VM discovery.
> * Prepare an account for automatic installation of the Mobility service on VMware vSphere VMs.
> * Review VMware vCenter Server and VM requirements and support.
> * Prepare to connect to Azure VMs after failover.

> [!NOTE]
> Tutorials show you the simplest deployment path for a scenario. They use default options where possible, and don't show all possible settings and paths. For detailed instructions, review the article in the How To section of the Site Recovery Table of Contents.

## Before you start

Make sure you've prepared Azure as described in the [first tutorial in this series](avs-tutorial-prepare-azure.md).

## Prepare an account for automatic discovery

Site Recovery needs access to Azure VMware Solution servers to:

- Automatically discover VMs. At least a read-only account is required.
- Orchestrate replication, failover, and failback. You need an account that can run operations such
  as creating and removing disks, and powering on VMs.

Create the account as follows:

1. To use a dedicated account, create a role at the vCenter Server level. Give the role a name such as
   **Azure_Site_Recovery**.
2. Assign the role the permissions summarized in the table below.
3. Create a user on the vCenter Server. Assign the role to the user.

### VMware account permissions

**Task** | **Role/Permissions** | **Details**
--- | --- | ---
**VM discovery** | At least a read-only user<br/><br/> Data Center object –> Propagate to Child Object, role=Read-only | User assigned at datacenter level, and has access to all the objects in the datacenter.<br/><br/> To restrict access, assign the **No access** role with the **Propagate to child** object, to the child objects (vSphere hosts, datastores, VMs and networks).
**Full replication, failover, failback** |  Create a role (Azure_Site_Recovery) with the required permissions, and then assign the role to a VMware user or group<br/><br/> Data Center object –> Propagate to Child Object, role=Azure_Site_Recovery<br/><br/> Datastore -> Allocate space, browse datastore, low-level file operations, remove file, update virtual machine files<br/><br/> Network -> Network assign<br/><br/> Resource -> Assign VM to resource pool, migrate powered off VM, migrate powered on VM<br/><br/> Tasks -> Create task, update task<br/><br/> Virtual machine -> Configuration<br/><br/> Virtual machine -> Interact -> answer question, device connection, configure CD media, configure floppy media, power off, power on, VMware tools install<br/><br/> Virtual machine -> Inventory -> Create, register, unregister<br/><br/> Virtual machine -> Provisioning -> Allow virtual machine download, allow virtual machine files upload<br/><br/> Virtual machine -> Snapshots -> Remove snapshots | User assigned at datacenter level, and has access to all the objects in the datacenter.<br/><br/> To restrict access, assign the **No access** role with the **Propagate to child** object, to the child objects (vSphere hosts, datastores, VMs and networks).

## Prepare an account for Mobility service installation

The Mobility service must be installed on machines you want to replicate. Azure Site Recovery can do a push installation of this service when you enable replication for a machine, or you can install it manually, or using installation tools.

- In this tutorial, we're going to install the Mobility service with the push installation.
- For this push installation, you need to prepare an account that Azure Site Recovery can use to access the VM. You specify this account
when you set up disaster recovery in the Azure console.

Prepare the account as follows:

- Prepare a domain or local account with permissions to install on the VM.
- **Windows VMs**: To install on Windows VMs if you're not using a domain account, disable Remote User Access
   control on the local machine. To do this, in the registry > **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, add the
     DWORD entry **LocalAccountTokenFilterPolicy**, with a value of 1.
- **Linux VMs**: To install on Linux VMs, prepare a root account on the source Linux server.


## Check Azure VMware Solution requirements

Make sure VMware vCenter Server and VMs comply with requirements.

1. Verify Azure VMware Solution [software versions](../azure-vmware/concepts-private-clouds-clusters.md#vmware-software-versions).
2. Verify [VMware vCenter Server requirements](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers).
3. For Linux VMs, [check](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage) file system and storage requirements. 
4. Check [network](vmware-physical-azure-support-matrix.md#network) and [storage](vmware-physical-azure-support-matrix.md#storage) support. 
5. Check what's supported for [Azure networking](vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover), [storage](vmware-physical-azure-support-matrix.md#azure-storage), and [compute](vmware-physical-azure-support-matrix.md#azure-compute), after failover.
6. Your Azure VMware Solution VMs you replicate to Azure must comply with [Azure VM requirements](vmware-physical-azure-support-matrix.md#azure-vm-requirements).
7. In Linux virtual machines, device name or mount point name should be unique. Ensure that no two devices/mount points have the same names. Note that name aren't case-sensitive. For example, naming two devices for the same VM as _device1_ and _Device1_ isn't allowed.


## Prepare to connect to Azure VMs after failover

After failover, you might want to connect to the Azure VMs from your Azure VMware Solution network.

To connect to Windows VMs using RDP after failover, do the following:

- **Internet access**. Before failover, enable RDP on the Azure VMware Solution VM before failover. Make sure that TCP, and UDP rules are added for the **Public** profile, and that RDP is allowed in **Windows Firewall** > **Allowed Apps**, for all profiles.
- **Site-to-site VPN access**:
    - Before failover, enable RDP on the Azure VMware Solution VM.
    - RDP should be allowed in the **Windows Firewall** -> **Allowed apps and features** for **Domain and Private** networks.
    - Check that the operating system's SAN policy is set to **OnlineAll**. [Learn
   more](https://support.microsoft.com/kb/3031135).
- There should be no Windows updates pending on the VM when you trigger a failover. If there are, you won't be able to sign in to the virtual
   machine until the update completes.
- On the Windows Azure VM after failover, check **Boot diagnostics** to view a screenshot of the VM. If you can't connect, check that the VM is running and review these
   [troubleshooting tips](https://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).

To connect to Linux VMs using SSH after failover, do the following:

- On the Azure VMware Solution VM before failover, check that the Secure Shell service is set to start
   automatically on system boot.
- Check that firewall rules allow an SSH connection.
- On the Azure VM after failover, allow incoming connections to the SSH port for the network security group rules on the failed over VM, and for the Azure subnet to which it's connected.
- [Add a public IP address](./site-recovery-monitor-and-troubleshoot.md) for the VM.
- You can check **Boot diagnostics** to view a screenshot of the VM.


## Failback requirements
If you plan to fail back to your Azure VMware Solution cloud, there are a number of [prerequisites for failback](avs-tutorial-reprotect.md#before-you-begin). You can prepare these now, but you don't need to. You can prepare after you fail over to Azure.




## Next steps
> [!div class="nextstepaction"]
> [Setup disaster recovery](avs-tutorial-replication.md)
- If you're replicating multiple VMs, [perform capacity planning](site-recovery-deployment-planner.md).
