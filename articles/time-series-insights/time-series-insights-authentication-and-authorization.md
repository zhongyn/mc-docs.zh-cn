---
title: 在 Azure 时序见解中使用 API 进行身份验证和授权 | Microsoft Docs
description: 本文介绍如何为调用 Azure 时序见解 API 的自定义应用程序配置身份验证和授权。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: dpalled
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 05/07/2019
ms.custom: seodec18
ms.openlocfilehash: fc7ba8a02732b21bdd3eb0656bae6e7fad719af0
ms.sourcegitcommit: c0f7c439184efa26597e97e5431500a2a43c81a5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67456388"
---
# <a name="authentication-and-authorization-for-azure-time-series-insights-api"></a>Azure 时序见解 API 的身份验证和授权

本文介绍如何配置调用 Azure 时序见解 API 的自定义应用程序中使用的身份验证和授权。

> [!TIP]
> 了解如何在 Azure Active Directory 中授予对时序见解环境的[数据访问权限](./time-series-insights-data-access.md)。

## <a name="service-principal"></a>服务主体

以下部分介绍如何将应用程序配置为访问应用程序的时序见解 API。 然后，该应用程序可以使用应用程序凭据（而不是用户凭据）在时序见解环境中查询或发布参考数据。

## <a name="best-practices"></a>最佳实践

当应用程序须访问时序见解时：

1. 设置一个 Azure Active Directory 应用。
1. 在时序见解环境中[分配数据访问策略](./time-series-insights-data-access.md)。

之所以需要使用应用程序凭据而不是用户凭据，是因为：

* 可将权限分配给应用标识，这些权限不同于你自己的权限。 通常情况下，这些权限仅限于应用需要的权限。 例如，可仅允许应用读取特定时序见解环境中的数据。
* 职责发生变化时，无需更改应用的凭据。
* 执行无人参与的脚本时，可使用证书或应用程序密钥自动进行身份验证。

以下部分演示如何通过 Azure 门户执行这些步骤。 文中重点介绍了单租户应用程序，其中应用程序只应在一个组织内运行。 通常会将单租户应用程序作为在组织中运行的业务线应用程序使用。

## <a name="setup-summary"></a>设置摘要

设置流程包括三个步骤：

1. 在 Azure Active Directory 中创建应用程序。
1. 授权此应用程序访问时序见解环境。
1. 使用应用程序 ID 和密钥从 `https://api.timeseries.azure.com/` 获取令牌。 然后可以使用该令牌调用时序见解 API。

## <a name="detailed-setup"></a>详细设置

1. 在 Azure 门户中，依次选择 Azure Active Directory > “应用注册” > “新应用程序注册”    。

   [![在 Azure Active Directory 中新建应用程序注册](media/authentication-and-authorization/active-directory-new-application-registration.png)](media/authentication-and-authorization/active-directory-new-application-registration.png#lightbox)

1. 命名应用程序，选择类型“Web 应用/API”，为“登录 URL”选择任意有效 URI，然后选择“创建”    。

   [![在 Azure Active Directory 中创建应用程序](media/authentication-and-authorization/active-directory-create-web-api-application.png)](media/authentication-and-authorization/active-directory-create-web-api-application.png#lightbox)

1. 选择新建的应用程序，将其应用程序 ID 复制到你偏好的文本编辑器中。

   [![复制应用程序 ID](media/authentication-and-authorization/active-directory-copy-application-id.png)](media/authentication-and-authorization/active-directory-copy-application-id.png#lightbox)

1. 选择“密钥”，输入密钥名称，选择过期时间，然后选择“保存”   。

   [![选择应用程序密钥](media/authentication-and-authorization/active-directory-application-keys.png)](media/authentication-and-authorization/active-directory-application-keys.png#lightbox)

   [![输入密钥名称和到期时间，然后选择“保存”](media/authentication-and-authorization/active-directory-application-keys-save.png)](media/authentication-and-authorization/active-directory-application-keys-save.png#lightbox)

1. 将密钥复制到喜爱的文本编辑器中。

   [![复制应用程序密钥](media/authentication-and-authorization/active-directory-copy-application-key.png)](media/authentication-and-authorization/active-directory-copy-application-key.png#lightbox)

1. 对于时序见解环境，请选择“数据访问策略”，然后选择“添加”   。

   [![将新的数据访问策略添加到时序见解环境](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png#lightbox)

1. 将步骤 2 中的应用程序名称或步骤 3 中的应用程序 ID 粘贴到“选择用户”对话框中  。

   [![在“选择用户”对话框中查找应用程序](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png#lightbox)

1. 选择角色。 选择“读取者”以查询数据，或选择“参与者”以查询数据和更改参考数据。   选择“确定”  。

   [![在“选择用户角色”对话框中选择“读取者”或“参与者”](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png#lightbox)

1. 选择“确定”以保存策略。 

1. 使用步骤 3 中的应用程序 ID 和步骤 5 中的应用程序密钥获取应用程序的令牌。 然后可在应用程序调用时序见解 API 时，将令牌传入 `Authorization` 标头。

    如果使用 C#，可使用以下代码获取应用程序的令牌。 有关完整示例，请参阅[使用 C# 查询数据](time-series-insights-query-data-csharp.md)。

    ```csharp
    // Enter your Active Directory tenant domain name
    var tenant = "YOUR_AD_TENANT.onmicrosoft.com";
    var authenticationContext = new AuthenticationContext(
        $"https://login.microsoftonline.com/{tenant}",
        TokenCache.DefaultShared);

    AuthenticationResult token = await authenticationContext.AcquireTokenAsync(
        // Set the resource URI to the Azure Time Series Insights API
        resource: "https://api.timeseries.azure.com/",
        clientCredential: new ClientCredential(
            // Application ID of application registered in Azure Active Directory
            clientId: "YOUR_APPLICATION_ID",
            // Application key of the application that's registered in Azure Active Directory
            clientSecret: "YOUR_CLIENT_APPLICATION_KEY"));

    string accessToken = token.AccessToken;
    ```

在应用程序中使用**应用程序 ID** 和**密钥**在 Azure 时序见解中进行身份验证。

## <a name="next-steps"></a>后续步骤

- 有关调用时序见解 API 的示例代码，请参阅[使用 C# 查询数据](time-series-insights-query-data-csharp.md)。
- 有关 API 参考信息，请参阅[查询 API 参考](https://docs.microsoft.com/rest/api/time-series-insights/ga-query-api)。
- 了解如何[创建服务主体](../active-directory/develop/howto-create-service-principal-portal.md)。
