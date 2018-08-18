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
ms.openlocfilehash: 7e51d3cd0533b4fb64d7dfa783af55266d536f54
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079135"
---
目前，此应用程序是一个功能正常的库，允许上传和查看图像。 本模块介绍如何使用 Microsoft 认知服务中的计算机视觉 API，以便为上传的图像生成标题并将标题与图像元数据一起保存在 Cosmos DB 中。

## <a name="create-a-computer-vision-account"></a>创建计算机视觉帐户

Microsoft 认知服务是一个提供给开发人员的服务集合，可以使其应用程序更为智能。 计算机视觉 API 是一项无服务器服务，可以使用高级算法处理图像并返回每个图像的信息。 它有一个免费层，每个月最多可以提供 5000 个 API 调用。

1. 确保仍登录到 Cloud Shell。 否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。 

1. 在资源组中创建一个新的类型为 **ComputerVision** 且名称唯一的认知服务帐户。 对于免费层，请使用 **F0** 作为 SKU。 如果已经有现有的计算机视觉帐户，则可能需要创建一个标准帐户 (S1)；不过，这可能会产生一些费用。

    ```azurecli
    az cognitiveservices account create -g first-serverless-app -n <computer vision account name> --kind ComputerVision --sku F0 -l westcentralus
    ```


## <a name="create-function-app-settings-for-computer-vision-url-and-key"></a>为计算机视觉 URL 和密钥创建函数应用设置

若要调用计算机视觉 API，需提供 URL 和密钥。

1. 获取计算机视觉 API 密钥和 URL，并将其保存到 bash 变量中。

    ```azurecli
    export COMP_VISION_KEY=$(az cognitiveservices account keys list -g first-serverless-app -n <computer vision account name> --query key1 --output tsv)
    ```
    ```azurecli
    export COMP_VISION_URL=$(az cognitiveservices account show -g first-serverless-app -n <computer vision account name> --query endpoint --output tsv)
    ```

1. 在函数应用中分别创建名为 **COMP_VISION_KEY** 和 **COMP_VISION_URL** 的应用设置。

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings COMP_VISION_KEY=$COMP_VISION_KEY COMP_VISION_URL=$COMP_VISION_URL -o table
    ```


## <a name="call-computer-vision-api-from-resizeimage-function"></a>从 ResizeImage 函数调用计算机视觉 API

在以下步骤中，请修改用于调用计算机视觉 API 的 **ResizeImage** 函数，以便对每个上传的图像进行说明并将说明保存在 Cosmos DB 中。

1. 在 Azure 门户中打开函数应用。

1. **C#**

    1. (C#) 使用左导航栏找到 **ResizeImage** 函数并打开其代码窗口。

    1. (C#) 将代码替换为 [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx) 的内容。 此代码使用 `HttpClient` 来调用计算机视觉 API 并将其结果保存在 Cosmos DB 中。

1. **JavaScript**

    1. (JavaScript) 此函数需要 npm 中的 `axios` 包对计算机视觉 API 进行 HTTP 调用。 若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。

    1. (JavaScript) 单击“控制台”以显示控制台窗口。

    1. (JavaScript) 在控制台中运行 `npm install axios` 命令。 完成此操作可能需要一到两分钟。

    1. (JavaScript) 在左侧导航栏中单击函数名称 (**ResizeImage**) 以显示该函数，然后将所有 **index.js** 替换为 [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js) 的内容。

1. 单击代码窗口下的“日志”，展开日志面板。

1. 单击“ **保存**”。 查看日志面板，确保函数已成功保存且没有错误。


## <a name="test-the-application"></a>测试应用程序

1. 在浏览器中打开应用程序。 选择一个图像文件并将其上传。

1. 几秒钟后，新图像的缩略图会显示在页面中。 将鼠标悬停在图像上方即可查看计算机视觉生成的说明。


## <a name="summary"></a>摘要

本模块介绍了如何使用 Microsoft 认知服务计算机视觉 API 为每个上传的图像自动生成标题。 接下来，请学习如何使用 Azure 应用服务身份验证向应用程序添加身份验证。