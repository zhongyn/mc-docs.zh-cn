---
title: 使用 Azure 虚拟 WAN 与 Azure 建立点到站点连接 | Azure
description: 本教程介绍如何使用 Azure 虚拟 WAN 与 Azure 建立点到站点 VPN 连接。
services: virtual-wan
author: rockboyfor
ms.service: virtual-wan
ms.topic: tutorial
origin.date: 02/27/2019
ms.date: 06/28/2019
ms.author: v-yeche
Customer intent: As someone with a networking background, I want to connect remote users to my VNets using Virtual WAN and I don't want to go through a Virtual WAN partner.
ms.openlocfilehash: 56217daab3d0d3d0f60d9d959e10335bc4b17a60
ms.sourcegitcommit: 7ad1a199e9f81de27b9098b50062ef059099f765
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67461887"
---
# <a name="tutorial-create-a-point-to-site-connection-using-azure-virtual-wan-preview"></a>教程：使用 Azure 虚拟 WAN（预览版）创建点到站点连接

本教程介绍如何使用虚拟 WAN 通过 IPsec/IKE (IKEv2) 或 OpenVPN VPN 连接与 Azure 中的资源建立连接。 此类连接要求在客户端计算机上配置一个客户端。 有关虚拟 WAN 的详细信息，请参阅[虚拟 WAN 概述](virtual-wan-about.md)。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建 WAN
> * 创建 P2S 配置
> * 创建中心
> * 将 P2S 配置应用到中心
> * 将 VNet 连接到中心
> * 下载并应用 VPN 客户端配置
> * 查看虚拟 WAN
> * 查看资源运行状况
> * 监视连接

> [!IMPORTANT]
> 此公共预览版在提供时没有附带服务级别协议，不应用于生产工作负荷。 某些功能可能不受支持或受到约束，或者不一定在所有 Azure 位置都可用。 有关详细信息，请参阅 [Azure 预览版补充使用条款](https://www.azure.cn/support/legal/subscription-agreement/)。
>

## <a name="before-you-begin"></a>准备阶段

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

<a name="register"></a>
## <a name="register-this-feature"></a>注册此功能

如果在本地运行 PowerShell，请确保使用最新版本并使用 **Connect-AzAccount -Environment AzureChinaCloud** 和 **Select-AzSubscription** 命令登录。

<!--Not Available onClick the **TryIt** to register this feature easily using Azure local Shell.-->

>[!NOTE]
>如果未注册此功能，则将无法使用它，也无法在门户中看到它。
>
>

<!--Not Available on After clicking **TryIt** to open the Azure local Shell-->

使用管理特权打开 PowerShell 控制台，然后复制并粘贴以下命令：

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowP2SCortexAccess
```

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowP2SCortexAccess
```

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Network -FeatureName AllowVnetGatewayOpenVpnProtocol
```

该功能显示为已注册后，请将订阅重新注册到 Microsoft.Network 命名空间。

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Network
```

<a name="vnet"></a>
## <a name="1-create-a-virtual-network"></a>1.创建虚拟网络

[!INCLUDE [Create a virtual network](../../includes/virtual-wan-tutorial-vnet-include.md)]

<a name="openvwan"></a>
## <a name="2-create-a-virtual-wan"></a>2.创建虚拟 WAN

从浏览器导航到 [Azure 门户（预览版）](https://portal.azure.cn/?feature.canmodifystamps=true&Microsoft_Azure_Network=flight16&feature.vwaner=true&feature.vwanp2s=true)并使用 Azure 帐户登录。

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-vwan-include.md)]

<a name="hub"></a>
## <a name="3-create-a-hub"></a>3.创建中心

> [!NOTE]
> 在此步骤中，请勿选择“包括点到站点网关”设置。
>

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-hub-include.md)]

<a name="site"></a>
## <a name="4-create-a-p2s-configuration"></a>4.创建 P2S 配置

P2S 配置定义连接远程客户端的参数。

1. 导航到“所有资源”  。
2. 单击创建的虚拟 WAN。
3. 在“虚拟 WAN 体系结构”下，单击“点到站点配置”   。
4. 单击页面顶部的“+添加点到站点配置”  ，打开“创建新的点到站点配置”页。 
5. 在“创建新的点到站点配置”页上填写以下字段： 

    *  **配置名称** - 这是需要按其来引用配置的名称。
    *  **隧道类型** - 用于隧道的协议。
    *  **地址池** - 这是 IP 地址池，为客户端分配的 IP 来自该池。
    *  **根证书名称** - 证书的说明性名称。
    *  **根证书数据** - Base-64 编码的 X.509 证书数据。

6. 单击“创建”  以创建配置。

<a name="hub"></a>
## <a name="5-edit-hub-assignment"></a>5.编辑中心分配

1. 在虚拟 WAN 的页面上，单击“中心”。 
2. 选择要向其分配点到站点配置的中心。
3. 单击“...”，选取“编辑虚拟中心”  
4. 勾选“包括点到站点网关”。 
5. 从下拉列表中，选择“网关缩放单元”  。
6. 从下拉列表中，选择所创建的“点到站点配置”  。
7. 为客户端配置“地址池”  。
8. 单击“确认”  。 此操作可能最多需要 30 分钟才能完成。

<a name="vnet"></a>
## <a name="6-connect-your-vnet-to-a-hub"></a>6.将 VNet 连接到中心

此步骤在中心与 VNet 之间创建对等互连。 针对要连接的每个 VNet 重复这些步骤。

1. 在虚拟 WAN 的页面上，单击“虚拟网络连接”。 
2. 在虚拟网络连接页上，单击“+添加连接”。 
3. 在“添加连接”页上填写以下字段  ：

    * **连接名称** - 为连接命名。
    * **中心** - 选择要与此连接关联的中心。
    * **订阅** - 验证订阅。
    * **虚拟网络** - 选择要连接到此中心的虚拟网络。 此虚拟网络不能包含现有的虚拟网络网关。
4. 单击“确定”添加连接  。

<a name="device"></a>
## <a name="7-download-vpn-profile"></a>7.下载 VPN 配置文件

使用 VPN 配置文件来配置客户端。

1. 在虚拟 WAN 的页面上，单击“中心”。 
2. 选择要下载其配置文件的中心。
3. 单击“...”，选取“下载配置文件”   。 
4. 完成创建文件后，可以单击相应的链接下载该文件。
4. 使用此配置文件配置点到站点客户端。

<a name="device"></a>
## <a name="8-configure-point-to-site-clients"></a>8.配置点到站点客户端
使用下载的配置文件配置远程访问客户端。 每个操作系统的过程并不相同，请按照下面的正确说明操作：

### <a name="microsoft-windows"></a>Microsoft Windows
#### <a name="openvpn"></a>OpenVPN

1. 从官方网站下载并安装 OpenVPN 客户端。
2. 下载网关的 VPN 配置文件。 可通过 Azure 门户中的“点到站点配置”选项卡或 PowerShell 中的 New-AzVpnClientConfiguration 来完成。
3. 解压缩该配置文件。 从记事本中的 OpenVPN 文件夹中打开 vpnconfig.ovpn 配置文件。
4. 使用 base64 中的 P2S 客户端证书公钥填写 P2S 客户端证书部分。 在 PEM 格式的证书中，可以直接打开 .cer 文件并在证书标头之间复制 base64 密钥。 请参阅此处，了解如何导出证书以获取已编码的公钥。
5. 使用 base64 中的 P2S 客户端证书私钥填写私钥部分。 请参阅此处，了解如何提取私钥。
6. 不要更改任何其他字段。 使用客户端输入中的已填充的配置连接到 VPN。
7. 将 vpnconfig.ovpn 文件复制到 C:\Program Files\OpenVPN\config 文件夹。
8. 右键单击系统托盘中的 OpenVPN 图标，然后单击“连接”。

#### <a name="ikev2"></a>IKEv2

1. 根据 Windows 计算机的体系结构选择 VPN 客户端配置文件。 对于 64 位处理器体系结构，请选择“VpnClientSetupAmd64”安装程序包。 对于 32 位处理器体系结构，请选择“VpnClientSetupX86”安装程序包。
2. 双击所需的包进行安装。 如果显示 SmartScreen 弹出窗口，请单击“更多信息”，并单击“仍要运行”。
3. 在客户端计算机上，导航到“网络设置”，并单击“VPN”。 VPN 连接显示所连接到的虚拟网络的名称。
4. 尝试连接前，请验证客户端计算机上是否已安装客户端证书。 使用本机 Azure 证书身份验证类型时，客户端证书是身份验证必需的。 有关如何生成证书的详细信息，请参阅“生成证书”。 有关如何安装客户端证书的信息，请参阅“安装客户端证书”。

### <a name="mac-os-x"></a>Mac (OS X)
#### <a name="openvpn"></a>OpenVPN

1. 下载并安装 OpenVPN 客户端，例如从 https://tunnelblick.net/downloads.html 下载并安装 TunnelBlik 
2. 下载网关的 VPN 配置文件。 可通过 Azure 门户中的“点到站点配置”选项卡或 PowerShell 中的 New-AzVpnClientConfiguration 来完成。
3. 解压缩该配置文件。 从记事本中的 OpenVPN 文件夹中打开 vpnconfig.ovpn 配置文件。
4. 使用 base64 中的 P2S 客户端证书公钥填写 P2S 客户端证书部分。 在 PEM 格式的证书中，可以直接打开 .cer 文件并在证书标头之间复制 base64 密钥。 请参阅此处，了解如何导出证书以获取已编码的公钥。
5. 使用 base64 中的 P2S 客户端证书私钥填写私钥部分。 请参阅此处，了解如何提取私钥。
6. 不要更改任何其他字段。 使用客户端输入中的已填充的配置连接到 VPN。
7. 双击配置文件以在 tunnelblick 中创建配置文件
8. 启动应用程序文件夹中的 Tunnelblik
9. 单击系统托盘中的 Tunneblik 图标，然后选取“连接”

#### <a name="ikev2"></a>IKEv2

Azure 不提供用于本机 Azure 证书身份验证的 mobileconfig 文件。 必须在将连接到 Azure 的每个 Mac 上手动配置本机 IKEv2 VPN 客户端。 Generic 文件夹包含配置该客户端所需的全部信息。 如果在下载中没有看到 Generic 文件夹，则可能 IKEv2 未选作隧道类型。 选择 IKEv2 后，再次生成 zip 文件，检索 Generic 文件夹。 Generic 文件夹包含以下文件：

- VpnSettings.xml：包含服务器地址和隧道类型等重要设置。
- VpnServerRoot.cer：包含在 P2S 连接设置过程中验证 Azure VPN 网关所需的根证书。

<a name="viewwan"></a>
## <a name="9-view-your-virtual-wan"></a>9.查看虚拟 WAN

1. 导航到虚拟 WAN。
2. 在“概述”页上，地图中的每个点表示一个中心。 将鼠标悬停在任一点上可以查看中心运行状况的摘要。
3. 在“中心和连接”部分，可以查看中心状态、站点、区域、VPN 连接状态和传入与传出字节数。

<a name="viewhealth"></a>
## <a name="10-view-your-resource-health"></a>10.查看资源运行状况

1. 导航到 WAN。
2. 在“WAN”页上的“支持 + 故障排除”部分，单击“运行状况”并查看资源。  

<a name="connectmon"></a>
## <a name="11-monitor-a-connection"></a>11.监视连接

创建一个连接，用于监视 Azure VM 与远程站点之间的通信。 有关如何设置连接监视器的信息，请参阅[监视网络通信](~/articles/network-watcher/connection-monitor.md)。 源字段是 Azure 中的 VM IP，目标 IP 是站点 IP。

<a name="cleanup"></a>
## <a name="12-clean-up-resources"></a>12.清理资源

不再需要这些资源时，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 删除资源组及其包含的所有资源。 将“myResourceGroup”替换为资源组的名称，并运行以下 PowerShell 命令：

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：

> [!div class="checklist"]
> * 创建 WAN
> * 创建站点
> * 创建中心
> * 将中心连接到站点
> * 将 VNet 连接到中心
> * 下载并应用 VPN 设备配置
> * 查看虚拟 WAN
> * 查看资源运行状况
> * 监视连接

若要详细了解虚拟 WAN，请参阅[虚拟 WAN 概述](virtual-wan-about.md)页。

<!--Update_Description: new articles on virtual wan point to site portal -->
<!--ms.date: 07/01/2019-->