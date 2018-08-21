---
title: include 文件
description: include 文件
services: functions
author: tdykstra
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 06/21/2018
ms.author: tdykstra
ms.custom: include file
ms.openlocfilehash: d1f9a07ce3d3b096b498e48b5c4f68c3454b2b37
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079132"
---
可以通过 Azure 应用服务身份验证在 Azure 函数应用中获得统包式身份验证支持。 它与 Facebook、Twitter、Microsoft 帐户、Google 和 Azure Active Directory 无缝集成。 需添加应用服务身份验证来保护 Web 应用的后端 API。

## <a name="enable-app-service-authentication"></a>启用应用服务身份验证

1. 在 Azure 门户中打开函数应用。

1. 在“平台功能”下，选择“身份验证/授权”。

    ![选择“身份验证/授权”](media/functions-first-serverless-web-app/6-authorization.jpg)

1. 选择以下值：
    
    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | **应用服务身份验证** | 启用 | 启用身份验证 |
    | **请求未经身份验证时的操作** | 使用 Azure Active Directory 登录 | 选择配置的身份验证方法（见下）。 |
    | **身份验证提供程序** | 请参阅下文 | 请参阅下文 |
    | **令牌存储** | 启用 | 允许应用服务存储和管理令牌。 |
    | **允许的外部重定向 URL** | 应用程序的 URL，例如：https://firstserverlessweb.z4.web.core.windows.net/ | 在用户进行身份验证后，允许将应用服务重定向到的 URL。 |

1. 选择“Azure Active Directory”，以便显示“Azure Active Directory 设置”。

    1. 单击“快速”作为“管理模式”并填写以下信息。
    
        | 设置      |  建议的值   | Description                                        |
        | --- | --- | ---|
        | **管理模式** | 快速、创建新的 AD 应用 | 自动设置服务主体和 Azure Active Directory 身份验证。 |
        | **创建应用** | my-serverless-webapp | 输入唯一的应用程序名称。 |
    
    1. 单击“确定”，保存 Azure Active Directory 设置。

    ![身份验证/授权和 Azure Active Directory 设置](media/functions-first-serverless-web-app/6-create-aad.png)

1. 单击“ **保存**”。


## <a name="modify-the-web-app-to-enable-authentication"></a>修改 Web 应用，以便启用身份验证

1. 在 Cloud Shell 中，确保当前目录为 **www/dist** 文件夹。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. 若要在函数应用中启用身份验证，请将以下代码行追加到 **settings.js**。

    `window.authEnabled = true`

    进行更改和查看结果时，可以使用以下命令，也可以使用命令行编辑器（例如 VIM）。

    ```azurecli
    echo "window.authEnabled = true" >> settings.js
    ```

    确认已对文件进行更改。

    ```azurecli
    cat settings.js
    ```

1. 将文件上传到 Blob 存储。

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-application"></a>测试应用程序

1. 在浏览器中打开应用程序。 单击“登录”即可登录。

1. 选择一个图像文件并将其上传。

    ![登录页](media/functions-first-serverless-web-app/6-aad-auth.png)
    

## <a name="summary"></a>摘要

本模块介绍了如何使用 Azure 应用服务身份验证向应用程序添加身份验证。
