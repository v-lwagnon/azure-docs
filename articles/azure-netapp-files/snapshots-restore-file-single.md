---
title: Restore individual files in Azure NetApp Files using single-file snapshot restore | Microsoft Docs
description: Describes how to recover individual files directly within a volume from a snapshot. 
services: azure-netapp-files
documentationcenter: ''
author: b-hchen
manager: ''
editor: ''

ms.assetid:
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 12/14/2021
ms.author: b-hchen
---

# Restore individual files using single-file snapshot restore 

If you do not want to [restore the entire snapshot to a new volume](snapshots-restore-new-volume.md) or [copy large files across the network](snapshots-restore-file-client.md), you have the option to use the single-file snapshot restore feature to recover individual files directly within a volume from a snapshot, without requiring an external client data copy.  

The single-file snapshot restore feature enables you to restore a single file or a list of files (up to 10 files at a time) from a snapshot.  You can specify a specific destination location or folder where the files will be restored to.    

## Considerations  

* If you use this feature to restore files to be new files, ensure that the volume has enough logical free space to accommodate the files.
* You can restore up to 10 files at a time, specified in a total length of 1024 characters.    
* All the directories in the destination path that you specify must be present in the active file system. 
The restore operation does not create directories in the process. If the specified destination path is invalid (does not exist in Active file system), the restore operation will fail.
* If you don’t specify a destination path, the files will be restored to the original file location. If the files already exist in the original location, they will be overwritten by the files restored from the snapshot. 
* A volume can have only one active file-restore operation. If you want to restore additional files, you must wait until the current restore operation is complete before triggering another restore operation.   
* *During the file restore operation*, the following restrictions apply: 
    * You cannot create new snapshots on the volume.  
    * You cannot delete the snapshot from which the files are being restored. 
    * If a snapshot policy is scheduled to take place at the same time, the snapshot schedule will be skipped, and a snapshot will not be created.

## Register the feature  

The single-file snapshot restore feature to restore individual files from a snapshot is currently in preview. If you are using this feature for the first time, register the feature before using it.

> [!IMPORTANT]
> Until further communication, [Azure NetApp Files for Azure Government](azure-government.md) users should contact their account team to access this feature instead of following the steps below.

1. Register the feature: 

    ```azurepowershell-interactive
    Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSingleFileSnapshotRestore
    ```

2. Check the status of the feature registration: 

    > [!NOTE]
    > The **RegistrationState** may be in the `Registering` state for up to 60 minutes before changing to `Registered`. Wait until the status is **Registered** before continuing.

    ```azurepowershell-interactive
    Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSingleFileSnapshotRestore
    ```
You can also use [Azure CLI commands](/cli/azure/feature) `az feature register` and `az feature show` to register the feature and display the registration status. 
 
## Steps

1. Navigate to the volume that has the snapshot to use for restoring files.    

2. Click **Snapshots** to display the list of volume snapshots.

3. Right-click the snapshot that you want to use for restoring files, and then select **Restore Files** from the menu.

    [ ![Snapshot that shows how to access the Restore Files menu item.](../media/azure-netapp-files/snapshot-restore-files-menu.png) ](../media/azure-netapp-files/snapshot-restore-files-menu.png#lightbox)

5. In the Restore Files window that appears, provide the following information:  
    1. In the **File Paths** field, specify the file or files to restore by using their full paths.   
        * You can specify up to 10 files each time. Multiple files must be separated with commas or newlines.
        * The maximum length of the File Paths field must not exceed 1024 characters and 10 files.
        * Regardless of the volume’s protocol type (NFS, SMB, or dual protocol), directories in the path must be specified using forward slashes (`/`) and not backslashes (`\`).  

    2. In the **Destination Path** field, provide the location in the volume where the specified files are to be restored to.
        * If you don’t specify a destination path, the files will be restored to their original location.  If files with the same names already exist in the original location, they will be overwritten by the files restored from the snapshot.  
        * If you specify a destination path: 
            * Ensure that all directories in the path are present in the active file system.  Otherwise, the restore operation will fail.   
                For example, if you specify `/CurrentCopy/contoso` as the destination path, the `/CurrentCopy/contoso` path must already exist.  
            * By specifying a destination path, all files specified in the File Paths field will be restored to the destination path (folder).
            * Regardless of the volume’s protocol type (NFS, SMB, or dual protocol), directories in the path must be specified using forward slashes (`/`) and not backslashes (`\`).   

    3. Click **Restore** to begin the restore operation.

    ![Snapshot the Restore Files window.](../media/azure-netapp-files/snapshot-restore-files-window.png)

## Examples 
The following examples show you how to specify files from a volume snapshot for restore. 

### NFS volumes (NFSv3/NFSv4.1)   

```bash
bash# sudo mkdir volume-azure-nfs
bash# sudo mount –t nfs –o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 10.1.1.8:/volume-azure-nfs volume-azure-nfs
bash# cd volume-azure-nfs/.snapshot
bash# ls 
daily-10-min-past-12am.2021-09-08_0010
daily-10-min-past-12am.2021-09-09_0010
bash# cd daily-10-min-past-12am.2021-09-08_0010
bash# ls
contoso department1 department2
bash# cd contoso
bash# ls
vm-7891.vmdk vm-8976.vmdk
```

File to be restored to the active file system:   
`volume-azure-nfs/.snapshot/daily-10-min-past-12am.2021-09-08_0010/contoso/vm-8976.vmdk`

Destination path in the active file system:  
`volume-azure-nfs/currentCopy/contoso`  

The path `/volume-azure-nfs/currentCopy/contoso` must be valid in the active file system.

From the Azure portal:   

1. Click **Snapshots**. Right-click the snapshot `daily-10-min-past-12am.2021-09-08_0010`.
2. Click **Restore Files**.
3. Specify **`/contoso/vm-8976.vmdk`** in File Paths.
4. Specify **`/currentCopy/contoso`** in Destination Path.

### SMB volumes 

```
C:\> net use N: \\scppr2-8336.contoso.com\volume-azure-smb
N:\> cd ~snapshot
N:\ dir
Directory of N:\~snapshot
09/11/2021  12:10 AM    <DIR>          .
09/22/2021  07:56 PM    <DIR>          ..
09/08/2021  01:47 PM                 102,400,000 daily-10-min-past-12am.2021-09-08_0010
09/09/2021  11:00 PM                 106,400,000 daily-10-min-past-12am.2021-09-09_0010
N:\> cd daily-10-min-past-12am.2021-09-08_0010
N:\> dir
Directory of N:\~snapshot\daily-10-min-past-12am.2021-09-08_0010
09/11/2021  12:10 AM    <DIR>          .
09/22/2021  07:56 PM    <DIR>          ..
02/27/2021  01:47 PM                 102,400 contoso
04/21/2021  11:00 PM                 106,400 department1
N:\> cd contoso
N:\> dir
Directory of N:\~snapshot\ daily-10-min-past-12am.2021-09-08_0010\contoso
09/11/2021  12:10 AM    <DIR>          .
09/22/2021  07:56 PM    <DIR>          ..
02/27/2021  01:47 PM                 102,400 vm-9981.vmdk
04/21/2021  11:00 PM                 106,400 vm-7654.vmdk
```

File to be restored to active file system:   
`N: \~snapshot\daily-10-min-past-12am.2021-09-08_0010\contoso\vm-9981.vmdk`

Destination path in the active file system:   
`N: \currentCopy\contoso`

The path `N:\currentCopy\contoso` must be valid in the active file system.

From the Azure portal: 
1. Click **Snapshots**. Select the snapshot `daily-10-min-past-12am.2021-09-08_0010`.
2. Click **Restore Files**.
3. Specify **`/contoso/vm-9981.vmdk`** in File Paths.
4. Specify **`/currentCopy/contoso`** in Destination Path.

## Next steps

* [Learn more about snapshots](snapshots-introduction.md) 
* [Resource limits for Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Azure NetApp Files Snapshot Overview](https://anfcommunity.com/2021/01/31/azure-netapp-files-snapshot-overview/)