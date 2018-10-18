---
title: include 文件
description: include 文件
services: functions
author: ggailey777
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 10/12/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 3779c2e130afa7ee8d5879f30a924e258b7a41e9
ms.sourcegitcommit: fdb43556b8dcf67cb39c18e532b5fab7ac53eaee
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/15/2018
ms.locfileid: "49315970"
---
要生成的应用程序是一个照片库。 它使用客户端 JavaScript 来调用 API，以便上传和显示图像。 在此模块中，请使用无服务器函数来创建 API，该函数生成用于上传图像的具有时间限制的 URL。 Web 应用程序使用 [Blob 存储 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 通过生成的 URL 将图像上传到 Blob 存储。

## <a name="create-a-blob-storage-container-for-images"></a>为图像创建 Blob 存储容器

应用程序需要单独的存储容器来上传并托管图像。

1. 确保仍登录到 Cloud Shell (bash)。 否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。

1.  在可以公开访问所有 Blob 的存储帐户中创建新的名为 **images** 的容器。

    ```azurecli
    az storage container create -n images --account-name <storage account name> --public-access blob
    ```

## <a name="create-an-azure-function-app"></a>创建 Azure Function App

Azure Functions 是一项用于运行无服务器函数的服务。 无服务器函数可以通过事件（例如发出了 HTTP 请求，或者在存储容器中创建了 Blob）触发（调用）。

Azure 函数应用是一个或多个无服务器函数的容器。

在此前创建的资源组中使用唯一名称创建名为 **first-serverless-app** 的新 Azure 函数应用。 函数应用需要一个存储帐户；在本教程中，请使用现有的存储帐户。

```azurecli
az functionapp create -n <function app name> -g first-serverless-app -s <storage account name> -c westcentralus
```

## <a name="configure-the-function-app"></a>配置函数应用

本教程中的函数应用需要 1.x 版的 Functions 运行时。 将 `FUNCTIONS_EXTENSION_VERSION` 应用程序设置为 `~1` 会将函数应用固定到最新的 1.x 版本。 使用 [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) 命令设置应用程序设置。

在以下 Azure CLI 命令中，<app_name> 是函数应用的名称。

```azurecli
az functionapp config appsettings set --name <function app name> --g first-serverless-app --settings FUNCTIONS_EXTENSION_VERSION=~1
```

## <a name="create-an-http-triggered-serverless-function"></a>创建 HTTP 触发的无服务器函数

照片库 Web 应用程序向无服务器函数发出 HTTP 请求，请求生成一个具有时间限制的 URL，以便将图像安全地上传到 Blob 存储。 此函数由 HTTP 请求触发，可以使用 Azure 存储 SDK 生成并返回安全的 URL。

1. 创建函数应用以后，请使用搜索框在 Azure 门户中搜索它，然后以单击方式将它打开。

    ![打开函数应用](media/functions-first-serverless-web-app/2-search-function-app.png)

1. 在函数应用窗口的左侧导航栏中，将鼠标悬停在“函数”上，然后单击“+”开始创建新的无服务器函数。

    ![创建新函数](media/functions-first-serverless-web-app/2-new-function.png)

1. 单击“自定义函数”即可看到函数模板的列表。

1. 找到 **HttpTrigger** 模板，单击要使用的语言（C# 或 JavaScript）。

1. 使用这些值创建一个可生成 Blob 上传 URL 的函数。

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | **语言** | C# 或 JavaScript | 选择要使用的语言。 |
    | **为函数命名** | GetUploadUrl | 根据显示准确地键入此名称，使应用程序能够发现此函数。 |
    | **授权级别** | 匿名 | 允许公开访问此函数。 |

    ![为 HTTP 触发的新函数输入设置](media/functions-first-serverless-web-app/2-new-function-httptrigger.png)

1. 单击“创建”创建该函数。

1. **C#** 

    1. 当函数的源代码显示时，请将所有 **run.csx** 替换为 [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx) 的内容。

1. **JavaScript** 

    1. (JavaScript) 此函数需要 npm 中的 `azure-storage` 包来生成共享访问签名 (SAS) 令牌，该令牌是生成安全 URL 所需的。 若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。

    1. (JavaScript) 单击“控制台”以显示控制台窗口。

        ![打开控制台窗口](media/functions-first-serverless-web-app/2-open-console.jpg)

    1. (JavaScript) 运行 `cd d:\home\site\wwwroot` 命令，确保当前目录为 **d:\home\site\wwwroot**。

    1. (JavaScript) 运行 `npm init -y` 命令，创建空的 **package.json** 文件。

    1. (JavaScript) 在控制台中运行 `npm install --save azure-storage` 命令，以便安装包并将其保存到 **package.json** 中。 完成此操作可能需要一到两分钟。

    1. (JavaScript) 在左侧导航栏中单击函数名称 (**GetUploadUrl**) 以显示该函数，将所有 **index.js** 替换为 [**javascript/GetUploadUrl/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js) 的内容。
    
        ![更新后的 index.js](media/functions-first-serverless-web-app/2-paste-js.jpg)

1. 单击代码窗口下的“日志”，展开日志面板。

1. 单击“ **保存**”。 查看日志面板，确保函数已成功编译。

此函数生成的东西称为共享访问签名 (SAS) URL，用于将文件上传到 Blob 存储。 此 SAS URL 在短时间内有效，仅允许上传单个文件。 若要详细了解如何[使用共享访问签名](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)，请查看 Blob 存储文档。


## <a name="add-an-environment-variable-for-the-storage-connection-string"></a>添加存储连接字符串的环境变量

刚创建的函数需要一个用于存储帐户的连接字符串，以便生成 SAS URL。 可以将连接字符串存储为应用程序设置，而不要在函数的正文中对其进行硬编码。 在函数应用中，应用程序设置可以作为环境变量供所有函数访问。

1. 在 Cloud Shell 中，查询存储帐户连接字符串并将其保存在名为 **STORAGE_CONNECTION_STRING** 的 bash 变量中。

    ```azurecli
    export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n <storage account name> -g first-serverless-app --query "connectionString" --output tsv)
    ```

    确认变量已成功设置。

    ```azurecli
    echo $STORAGE_CONNECTION_STRING
    ```

1. 使用在上一步保存的值，创建名为 **AZURE_STORAGE_CONNECTION_STRING** 的新应用程序设置。

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING -o table
    ```

    确认命令的输出包含新应用程序设置且值正确。


## <a name="test-the-serverless-function"></a>测试无服务器函数

除了创建和编辑函数，Azure 门户还提供内置的用于测试函数的工具。

1. 若要测试 HTTP 无服务器函数，请单击代码窗口右侧的“测试”选项卡，展开测试面板。

1. 将“Http 方法”更改为 **GET**。

1. 在“查询”下单击“添加参数”*，然后添加以下参数：

    | 名称      |  值   | 
    | --- | --- |
    | filename | image1.jpg |

1. 单击测试面板中的“运行”，向函数发送 HTTP 请求。

1. 函数在输出中返回上传 URL。 函数的执行情况显示在日志面板中。

    ![显示函数已成功运行的日志](media/functions-first-serverless-web-app/2-test-function.png)


## <a name="configure-cors-in-the-function-app"></a>在函数应用中配置 CORS

由于此应用的前端托管在 Blob 存储中，因此其域名不同于 Azure 函数应用的域名。 需将函数应用配置为进行跨域资源共享 (CORS)，以便客户端 JavaScript 成功调用刚创建的函数。

1. 在函数应用窗口的左导航栏中，单击函数应用的名称。

1. 单击“平台功能”，查看高级功能的列表。

1. 在“API”下，单击“CORS”。

    ![选择 CORS](media/functions-first-serverless-web-app/2-open-cors.jpg)

1. 添加在上一模块中提供的应用程序 URL 的允许源，省略尾随 **/**（例如：`https://firstserverlessweb.z4.web.core.windows.net`）。

    ![显示无服务器 Web 应用 URL 已添加的 CORS 设置](media/functions-first-serverless-web-app/2-add-cors.png)

1. 单击“ **保存**”。

1. C#

   1. (C#) 导航回 `GetUploadUrl` 函数，然后选择“集成”选项卡。

   1. (C#) 在“选定 HTTP 方法”下，选择“OPTIONS”。

      “GET”、“POST”和“OPTIONS”应该均处于选定状态。 CORS 使用 OPTIONS 方法。对 C# 函数来说，此方法并未默认选定。  

   1. (C#) 单击“保存”。

1. 此时仍在门户中，导航到函数应用，选择“概览”选项卡，然后单击“重启”，确保 CORS 的更改生效。

## <a name="configure-cors-in-the-storage-account"></a>在存储帐户中配置 CORS

由于应用还对 Blob 存储进行客户端 JavaScript 调用，以便上传文件，因此还需为 CORS 配置存储帐户。

1. 运行以下命令，以便所有源将文件上传到存储帐户。

    ```azurecli
    az storage cors add --methods OPTIONS PUT --origins '*' --exposed-headers '*' --allowed-headers '*' --services b --account-name <storage account name>
    ```


## <a name="modify-the-web-app-to-upload-images"></a>修改用于上传图像的 Web 应用

Web 应用从名为 **settings.js** 的文件中检索设置。 在以下步骤中，请使用 Cloud Shell 创建该文件，然后将 `window.apiBaseUrl` 设置为函数应用的 URL，将 `window.blobBaseUrl` 设置为 Azure Blob 存储终结点的 URL。

1. 在 Cloud Shell 中，确保当前目录为 **www/dist** 文件夹。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. 查询函数应用的 URL，确保将其存储在名为 **FUNCTION_APP_URL** 的 bash 变量中。

    ```azurecli
    export FUNCTION_APP_URL="https://"$(az functionapp show -n <function app name> -g first-serverless-app --query "defaultHostName" --output tsv)
    ```

    确认变量已正确设置。

    ```azurecli
    echo $FUNCTION_APP_URL
    ```

1. 若要设置对函数应用进行的 API 调用的基 URI，请创建 **settings.js** 并添加函数应用 URL，如下所示。

    `window.apiBaseUrl = 'https://fnapp@lab.GlobalLabInstanceId.azurewebsites.net'`

    进行更改时，可以运行以下命令，也可以使用命令行编辑器（例如 VIM）。

    ```azurecli
    echo "window.apiBaseUrl = '$FUNCTION_APP_URL'" > settings.js
    ```

    确认文件已成功编写。

    ```azurecli
    cat settings.js
    ```

1. 查询 Blob 存储基 URL 并将其存储在名为 **BLOB_BASE_URL** 的 bash 变量中。

    ```azurecli
    export BLOB_BASE_URL=$(az storage account show -n <storage account name> -g first-serverless-app --query primaryEndpoints.blob -o tsv | sed 's/\/$//')
    ```

    确认变量已正确设置。

    ```azurecli
    echo $BLOB_BASE_URL
    ```

1. 若要设置对函数应用进行的 API 调用的基 URI，请将存储 URL（如以下代码行所示）追加到 **settings.js**。

    `window.blobBaseUrl = 'https://mystorage.blob.core.windows.net'`

    进行更改时，可以运行以下命令，也可以使用命令行编辑器（例如 VIM）。

    ```azurecli
    echo "window.blobBaseUrl = '$BLOB_BASE_URL'" >> settings.js
    ```

    确认文件已成功编写，它现在包含 2 行内容。

    ```azurecli
    cat settings.js
    ```

1. 将文件上传到 Blob 存储。

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-web-application"></a>测试 Web 应用程序

目前，库应用程序能够将图像上传到 Blob 存储，但还不能显示图像。 它会尝试调用 `GetImages` 函数，该函数尚不存在，因为是在以后的模块中创建的。 该调用会失败，而网页似乎卡在“正在分析...”处，但所选图像会成功上传。

可以在 Azure 门户中查看 **images** 容器的内容，验证图像是否已成功上传。

1. 在浏览器窗口中浏览到应用程序。 选择一个图像文件并将其上传。 上传完成，但由于尚未添加显示图像的功能，应用不显示上传的照片。 （网页似乎卡在“正在分析图像...”处；稍后会修复该问题。）

1. 在 Cloud Shell 中，确认图像已上传到 **images** 容器。

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. 在转到下一教程之前，请删除 **images** 容器中的所有文件。

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```


## <a name="summary"></a>摘要

在本模块中，你创建了一个 Azure 函数应用，并学习了如何使用无服务器函数通过 Web 应用程序将图像上传到 Blob 存储。 接下来，请学习如何使用 Blob 触发的无服务器函数为上传的图像创建缩略图。
