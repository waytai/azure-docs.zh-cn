---
title: "通过 Linux 使用 Azure 文件 | Microsoft Docs"
description: "了解如何在 Linux 上通过 SMB 装载 Azure 文件共享。"
services: storage
documentationcenter: na
author: RenaShahMSFT
manager: aungoo
editor: tysonn
ms.assetid: 6edc37ce-698f-4d50-8fc1-591ad456175d
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/20/2017
ms.author: renash
ms.openlocfilehash: cca0d315a815faca5db07099b8e8e451ef55fad5
ms.sourcegitcommit: 2a70752d0987585d480f374c3e2dba0cd5097880
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/19/2018
---
# <a name="use-azure-files-with-linux"></a>通过 Linux 使用 Azure 文件
[Azure 文件](storage-files-introduction.md)是 Microsoft 推出的易用云文件系统。 可以使用 [CIFS 内核客户端](https://wiki.samba.org/index.php/LinuxCIFS)在 Linux 分发版中装载 Azure 文件共享。 本文介绍装载 Azure 文件共享的两种方法：使用 `mount` 命令按需装载，以及通过在 `/etc/fstab` 中创建一个条目在启动时装载。

> [!NOTE]  
> 若要将 Azure 文件共享装载到其被托管时所在的 Azure 区域之外（例如本地或其他 Azure 区域），OS 必须支持 SMB 3.0 的加密功能。

## <a name="prerequisites-for-mounting-an-azure-file-share-with-linux-and-the-cifs-utils-package"></a>使用 Linux 和 cifs-utils 包装载 Azure 文件共享的先决条件
* **选取可能安装了 cifs-utils 包的 Linux 分发版。**  
    以下 Linux 分发版可在 Azure 库中使用：

    * Ubuntu Server 14.04+
    * RHEL 7+
    * CentOS 7+
    * Debian 8+
    * openSUSE 13.2+
    * SUSE Linux Enterprise Server 12

* <a id="install-cifs-utils"></a>**cifs-utils 包已安装。**  
    可在所选的 Linux 分发版上使用包管理器安装 cifs-utils 包。 

    在 **Ubuntu** 和**基于 Debian** 的分发版上，请使用 `apt-get` 包管理器：

    ```
    sudo apt-get update
    sudo apt-get install cifs-utils
    ```

    在 **RHEL** 和 **CentOS** 上，请使用 `yum` 包管理器：

    ```
    sudo yum install samba-client samba-common cifs-utils
    ```

    在 **openSUSE** 上，请使用 `zypper` 包管理器：

    ```
    sudo zypper install samba*
    ```

    在其他分发版上，请使用相应的包管理器，或[从源编译](https://wiki.samba.org/index.php/LinuxCIFS_utils#Download)。

* <a id="smb-client-reqs"></a>**了解 SMB 客户端要求。**  
    可以通过 SMB 2.1 和 SMB 3.0 装载 Azure 文件。 对于来自本地或其他 Azure 区域中的客户端的连接，Azure 文件会拒绝 SMB 2.1（或没有加密的 SMB 3.0）。 如果为存储帐户启用“需要安全转移”，则 Azure 文件仅允许使用带加密的 SMB 3.0 进行连接。
    
    SMB 3.0 加密支持在 Linux 内核版本 4.11 中引入，已向后移植到常见 Linux 分发版的早期内核版本中。 在本文档发布时，Azure 库的以下分发版支持此功能：

    - Ubuntu Server 16.04+
    - openSUSE 42.3+
    - SUSE Linux Enterprise Server 12 SP3+
    
    如果此处未列出你的 Linux 分发版，则你可以使用以下命令查看 Linux 内核版本：

    ```
    uname -r
    ```

* **确定已装载共享的目录/文件权限**：在以下示例中，权限 `0777` 用于向所有用户授予读取、写入和执行权限。 可以根据需要将这些权限替换为其他 [chmod 权限](https://en.wikipedia.org/wiki/Chmod)。 

* **存储帐户名称**：需提供存储帐户的名称才能装载 Azure 文件共享。

* **存储帐户密钥**：需提供主要（或辅助）存储密钥才能装载 Azure 文件共享。 目前不支持使用 SAS 密钥进行装载。

* **确保已打开端口 445**：SMB 通过 TCP 端口 445 通信 - 请查看防火墙是否未阻止 TCP 端口 445 与客户端计算机通信。

## <a name="mount-the-azure-file-share-on-demand-with-mount"></a>使用 `mount` 按需装载 Azure 文件共享
1. [安装适用于 Linux 分发版的 cifs-utils 包](#install-cifs-utils)。

2. **为装入点创建文件夹**：可以在文件系统上的任何位置创建装入点的文件夹，但是通用约定是在 `/mnt` 文件夹下创建此文件夹。 例如：

    ```
    mkdir /mnt/MyAzureFileShare
    ```

3. **使用 mount 命令装载 Azure 文件共享**：记得将 `<storage-account-name>`、`<share-name>`、`<smb-version>`、`<storage-account-key>` 和 `<mount-point>` 替换为适用于你环境的相应信息。 如果你的 Linux 分发版支持带加密的 SMB 3.0（有关详细信息，请参阅[了解 SMB 客户端要求](#smb-client-reqs)），则将 `3.0` 用于 `<smb-version>`。 对于不支持带加密的 SMB 3.0 的 Linux 分发版，将 `2.1` 用于 `<smb-version>`。 请注意，只能使用 SMB 3.0 在 Azure 区域外部（包括本地或不同 Azure 区域中）装载 Azure 文件共享。 

    ```
    sudo mount -t cifs //<storage-account-name>.file.core.windows.net/<share-name> <mount-point> -o vers=<smb-version>,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
    ```

> [!Note]  
> 使用完 Azure 文件共享后，可以使用 `sudo umount <mount-point>` 卸载共享。

## <a name="create-a-persistent-mount-point-for-the-azure-file-share-with-etcfstab"></a>使用 `/etc/fstab` 为 Azure 文件共享创建持久装入点
1. [安装适用于 Linux 分发版的 cifs-utils 包](#install-cifs-utils)。

2. **为装入点创建文件夹**：可以在文件系统上的任何位置创建装入点的文件夹，但是通用约定是在 `/mnt` 文件夹下创建此文件夹。 无论在何处创建此文件夹，请记下此文件夹的绝对路径。 例如，以下命令在 `/mnt` 下创建新文件夹（路径为绝对路径）。

    ```
    sudo mkdir /mnt/MyAzureFileShare
    ```

3. **使用以下命令将以下代码行追加到 `/etc/fstab`**：记得将 `<storage-account-name>`、`<share-name>`、`<smb-version>`、`<storage-account-key>` 和 `<mount-point>` 替换为适用于你环境的相应信息。 如果你的 Linux 分发版支持带加密的 SMB 3.0（有关详细信息，请参阅[了解 SMB 客户端要求](#smb-client-reqs)），则将 `3.0` 用于 `<smb-version>`。 对于不支持带加密的 SMB 3.0 的 Linux 分发版，将 `2.1` 用于 `<smb-version>`。 请注意，只能使用 SMB 3.0 在 Azure 区域外部（包括本地或不同 Azure 区域中）装载 Azure 文件共享。 

    ```
    sudo bash -c 'echo "//<storage-account-name>.file.core.windows.net/<share-name> <mount-point> cifs nofail,vers=<smb-version>,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino" >> /etc/fstab'
    ```

> [!Note]  
> 编辑 `/etc/fstab` 后，可以使用 `sudo mount -a` 装载 Azure 文件共享，而无需重启。

## <a name="feedback"></a>反馈
Linux 用户，我们希望倾听意见！

针对 Linux 用户组的 Azure 文件提供了一个论坛，在 Linux 上评估和采用文件存储时，可以在该论坛上共享反馈。 向 [Azure 文件 Linux 用户](mailto:azurefileslinuxusers@microsoft.com)发送电子邮件可加入该用户组。

## <a name="next-steps"></a>后续步骤
请参阅以下链接，获取有关 Azure 文件的更多信息。
* [Azure 文件简介](storage-files-introduction.md)
* [规划 Azure 文件部署](storage-files-planning.md)
* [常见问题](../storage-files-faq.md)
* [故障排除](storage-troubleshoot-linux-file-connection-problems.md)