# Azure Virtual Machines and Disk Management: AZ-104 Study Guide

This comprehensive guide covers core concepts, practical lab examples, and AZ-104 exam preparation material for Azure Virtual Machine disk and management topics.

---

## 1. Azure Virtual Machine - Disks & Adding Data Disks

Azure Virtual Machines (VMs) utilize virtual hard disks (VHDs) to store the operating system and data. Azure manages these VHDs as "Managed Disks," which abstract the underlying storage account management away from the user, providing better reliability and scalability.

### Types of Disks on an Azure VM
*   **OS Disk:** Every VM has one attached operating system disk. It contains the pre-installed OS selected when the VM was created. 
    *   *Windows:* Typically drive `C:`
    *   *Linux:* Typically `/dev/sda`
*   **Temporary Disk:** Every VM contains a temporary (ephemeral) disk. It provides short-term storage for applications and processes (like page files or swap files). 
    *   *Warning:* Data on this disk is **lost** during maintenance events or when you stop (deallocate) the VM. Never store persistent data here.
    *   *Windows:* Typically drive `D:`
    *   *Linux:* Typically `/dev/sdb`
*   **Data Disks:** These are manually attached to VMs to store application data. They are registered as SCSI drives. The maximum number of data disks you can attach is determined by the VM's size (e.g., a Standard_D2s_v3 might support 4 data disks, while a larger size supports 32).

### Lab Example: Adding a Data Disk
To successfully add and use a new data disk, you must complete tasks in both Azure and the VM's OS:
1.  **Azure Portal:** Navigate to your VM > **Disks** > **Create and attach a new disk**. Specify the name, storage type (e.g., Premium SSD), and size (GiB), then save.
2.  **OS Configuration (Crucial Step):** Attaching the disk in Azure only connects the virtual hardware. You must log into the VM:
    *   *Windows:* Open **Disk Management**, initialize the new disk (MBR or GPT), create a new simple volume, and format it (NTFS).
    *   *Linux:* Use `fdisk` to partition the drive (`/dev/sdc`), format it with `mkfs.ext4`, and mount it to a directory.

---

## 2. VM States: What Happens When We Stop the Machine

Understanding VM states is a critical AZ-104 concept because it directly impacts your Azure billing.

*   **Stopped (Allocated):** If you log into the VM (via RDP/SSH) and shut it down from within the OS, the VM goes into a "Stopped" state. 
    *   *Billing:* Azure still reserves the CPU and RAM on the physical host server. **You continue to be billed for compute resources.**
*   **Stopped (Deallocated):** If you click the **Stop** button in the Azure Portal (or use Azure CLI/PowerShell, e.g., `Stop-AzVM`), the VM is deallocated. 
    *   *Billing:* The hardware resources are released back to the Azure pool. **You stop paying for compute costs.** (Note: You are still billed for the attached disks and any static public IPs).
*   **IP Address Implication:** If your VM has a *Dynamic* Public IP address, it is released when the VM is deallocated. When you start the VM again, it will receive a completely new Public IP.

---

## 3. Data Disks Snapshot

A snapshot is a full, read-only copy of a virtual hard disk (VHD) taken at a specific point in time. 

### Key Concepts
*   **Purpose:** Snapshots are highly useful for manual backups, disaster recovery, or creating a baseline before a risky system change (like a major patching operation).
*   **Snapshot Types:**
    *   *Full Snapshot:* A complete copy of the disk.
    *   *Incremental Snapshot:* Only backs up the data changes since the last snapshot, which significantly saves on storage costs.
*   **Deployment limitation:** You **cannot** attach a snapshot directly to a VM. 

### Lab Example: Restoring from a Snapshot
If a bad OS patch crashes your VM:
1.  Locate the snapshot you took prior to the patch.
2.  Click **Create Disk** from the snapshot menu to generate a new Managed Disk.
3.  Go to the VM's **Disks** blade and select **Swap OS disk**.
4.  Choose the newly created disk to replace the corrupted one and restart the VM.

---

## 4. Azure Disks - Server Side Encryption & Key Vault

Azure ensures that data on managed disks is secure by encrypting it at rest. You need to know the different methods of key management for the exam.

### Server-Side Encryption (SSE)
By default, all Azure Managed Disks, Snapshots, and Images are automatically encrypted at rest using Storage Service Encryption (SSE) with **Platform Managed Keys (PMK)**. Microsoft handles key rotation and management transparently.

### Customer-Managed Keys (CMK), Key Vault, and Disk Encryption Sets
For strict compliance, organizations often must control their own encryption keys. This requires three distinct Azure resources:

1.  **Azure Key Vault:** A secure cloud service used to store and manage cryptographic keys, secrets, and certificates. You generate or import your key here.
2.  **Disk Encryption Set (DES):** You cannot link a managed disk directly to a Key Vault. The DES acts as a bridge. It connects to your Key Vault, retrieves the encryption key, and applies it to your Managed Disks.
3.  **Managed Identity:** The DES must be granted a System Assigned Managed Identity with permission to "Get", "Wrap Key", and "Unwrap Key" inside the Key Vault's access policies.

### Lab Example: Configuring CMK
1. Create a Key Vault and generate an RSA key.
2. Create a Disk Encryption Set, point it to the Key Vault and the specific key.
3. Grant the DES access to the Key Vault.
4. When creating a new Managed Disk (or modifying an unattached one), change the encryption type to "Encryption at-rest with a customer-managed key" and select your DES.

---

## AZ-104 Exam-Type Questions

**Question 1: VM Stopping States**
You have an Azure Virtual Machine named `VM1` that is currently running. You connect to `VM1` via RDP and shut down the operating system using the Windows Start menu. Which of the following statements is true regarding your Azure bill?
A) Both compute and storage billing stop immediately.
B) Compute billing continues, and storage billing continues.
C) Compute billing stops, but storage billing continues.
D) The VM is automatically deleted after 30 days.

*Answer:* **B**. Because the VM was shut down from within the OS, it enters a "Stopped (Allocated)" state. The Azure fabric still reserves the hardware on the host, so you are still billed for compute resources. To stop compute billing, you must stop the VM from the Azure Portal, CLI, or PowerShell.

**Question 2: Temporary Disks**
You are deploying a new application to an Azure VM. The application requires a scratch disk to write temporary processing logs. You decide to point the application to the `D:` drive (Temporary Storage) to save on data disk costs. What is the primary risk of this design?
A) The `D:` drive cannot be formatted with NTFS.
B) The `D:` drive is encrypted by default and will slow down processing.
C) Data on the `D:` drive will be permanently lost if the VM is stopped (deallocated) or moved to a new host during maintenance.
D) The `D:` drive has a strict 10 GB limit regardless of VM size.

*Answer:* **C**. The temporary disk provides ephemeral storage. Data is lost upon deallocation or standard hardware maintenance events. It is perfectly fine for scratch logs, but you must be aware that the data is volatile.

**Question 3: Disk Encryption**
Your company's security policy dictates that all Azure Virtual Machine data disks must be encrypted at rest using keys generated and managed by your own internal security team. Which three Azure resources must you deploy and configure to meet this requirement? (Select three)
A) Azure Key Vault
B) Disk Encryption Set
C) Azure Storage Account
D) Azure Bastion
E) Customer Managed Key (CMK)

*Answer:* **A, B, E**. To manage your own keys for disk encryption, you must generate a Customer Managed Key (CMK), store it securely in an Azure Key Vault, and bridge it to the VM disks using a Disk Encryption Set.

**Question 4: Snapshots and Disk Recovery**
You have an Azure VM with a 128 GB OS disk. You take a snapshot of the OS disk. The next day, a junior administrator accidentally deletes several critical system files, rendering the VM unbootable. You need to restore the VM using the snapshot. What is the correct sequence of steps?
A) Attach the snapshot directly to the VM as a data disk and boot from it.
B) Create a new managed disk from the snapshot, then use the "Swap OS disk" feature on the VM to attach the new disk.
C) Run a PowerShell script to merge the snapshot directly back into the running OS disk.
D) Upload the snapshot to an Azure Storage account and use Azure Backup to restore the VM.

*Answer:* **B**. Snapshots are read-only point-in-time backups and cannot be attached directly to a VM. You must first create a new managed disk from the snapshot, and then swap the current OS disk with the newly created disk in the VM's settings.