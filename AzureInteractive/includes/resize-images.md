---
title: include 文件
description: include 文件
services: functions
author: ggailey777
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 06/21/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: d19a9d0e7e0347b38fc16f85fa440281be5347f2
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460070"
---
上一模块介绍了如何使用无服务器函数将图像从 Web 应用程序安全地上传到 Blob 存储。 本模块将创建另一无服务器函数，用于监视上传的图像并根据图像创建缩略图。

## <a name="create-a-blob-storage-container-for-thumbnails"></a>为缩略图创建 Blob 存储容器

完整大小的图像存储在名为 **images** 的容器中。 需要另一容器来存储这些图像的缩略图。

1. 确保仍登录到 Cloud Shell (bash)。  否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。 

1. 在可以公开访问所有 Blob 的存储帐户中创建新的名为 **thumbnails** 的容器。

    ```azurecli
    az storage container create -n thumbnails --account-name <storage account name> --public-access blob
    ```


## <a name="create-a-blob-triggered-serverless-function"></a>创建 Blob 触发的无服务器函数

触发器定义函数的调用方式。 接下来创建的函数使用 Blob 触发器：当某个 Blob（图像文件）上传到 **images** 容器时，系统会自动调用此函数。 一个函数必须有一个触发器。 触发器具有关联数据，该数据通常是触发函数的有效负载。

绑定定义函数在 Azure 或第三方服务中读取或写入数据的方式。 此函数创建触发它的图像的缩略图版本，并将缩略图保存在 *thumbnails* 容器中。

1. 在 Azure 门户中打开函数应用。

1. 在函数应用窗口的左侧导航栏中，将鼠标悬停在“函数”上，然后单击“+”开始创建新的无服务器函数。 如果显示快速入门页，则单击“自定义函数”即可看到函数模板的列表。

1. 找到 **BlobTrigger** 模板并将其选中。

1. 使用这些值创建一个函数，以便在图像上传后创建缩略图。

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | **语言** | C# 或 JavaScript | 选择首选语言。 |
    | **为函数命名** | ResizeImage | 根据显示准确地键入此名称，使应用程序能够发现此函数。 |
    | **路径** | images/{name} | 当某个文件出现在 **images** 容器中时执行此函数。 |
    | **存储帐户信息** | AZURE_STORAGE_CONNECTION_STRING | 将以前创建的环境变量与此连接字符串配合使用。 |

    ![输入新函数的设置](media/functions-first-serverless-web-app/3-new-function.png)

1. 单击“创建”创建该函数。

1. 创建函数后，单击“集成”即可查看其触发器、输入和输出绑定。

1. 单击“新建输出”创建新的输出绑定。

    ![在“集成”选项卡上选择“新建输出”](media/functions-first-serverless-web-app/3-new-output.jpg)

1. 选择“Azure Blob 存储”，然后单击“选择”。 可能需要向下滚动才能看到“选择”按钮。

    ![选择“Azure Blob 存储”](media/functions-first-serverless-web-app/3-storage-output.jpg)

1. 输入以下值。

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | **Blob 参数名称** | thumbnail | 函数将参数与此名称配合使用，以便写入缩略图。 |
    | **使用函数返回值** | 否 |  |
    | **路径** | thumbnails/{name} | 缩略图会写入一个名为 **thumbnails** 的容器中。 |
    | **存储帐户连接** | AZURE_STORAGE_CONNECTION_STRING | 将以前创建的环境变量与此连接字符串配合使用。 |

    ![输入 Blob 输出绑定的设置](media/functions-first-serverless-web-app/3-blob-output.png)

1. **JavaScript**

    1. (JavaScript) 单击窗口右上角的“高级编辑器”以显示表示绑定的 JSON。

    1. (JavaScript) 在 `blobTrigger` 绑定中，添加名称为 `dataType` 且值为 `binary` 的属性。 这样就可以配置绑定，以便将 Blob 内容以二进制数据的形式传递给函数。

    ```json
    {
      "name": "myBlob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "images/{name}",
      "connection": "AZURE_STORAGE_CONNECTION_STRING",
      "dataType": "binary"
    }
    ```

1. 单击“保存”创建新绑定。

1. **C#**

    1. (C#) 在左导航栏中选择 **ResizeImage** 函数名称，打开函数的源代码。

    1. (C#) 此函数需要名为 **ImageResizer** 的 NuGet 包来生成缩略图。 NuGet 包是使用 **project.json** 文件添加到 C# 函数的。 若要创建此文件，请单击右侧的“查看文件”以显示构成函数的文件。
    
    1. (C#) 单击“添加”，添加名为 **project.json** 的新文件。
    
    1. (C#) 将 [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) 的内容复制到新创建的文件中。 保存文件。 文件更新后，包会自动还原。
    
        ![包含 ImageResizer 的 project.json 文件](media/functions-first-serverless-web-app/3-project-json.png)
    
    1. (C#) 在“查看文件”下选择 **run.csx**，将其内容替换为 [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx) 中的内容。

1. **JavaScript** 

    1. (JavaScript) 此函数需要 npm 中的 `jimp` 包来重设照片大小。 若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。

    1. (JavaScript) 单击“控制台”以显示控制台窗口。

    1. (JavaScript) 在控制台中运行 `npm install jimp` 命令。 完成此操作可能需要一到两分钟。

    1. (JavaScript) 在左侧导航栏中单击函数名称 **ResizeImage** 以显示该函数，将所有 **index.js** 替换为 [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js) 的内容。

1. 单击代码窗口下的“日志”，展开日志面板。

1. 单击“ **保存**”。 查看日志面板，确保函数已成功保存且没有错误。


## <a name="test-the-serverless-function"></a>测试无服务器函数

1. 在浏览器中打开应用程序。 选择一个图像文件并将其上传。 上传完成，但由于尚未添加显示图像的功能，应用不显示上传的照片。

1. 在 Cloud Shell 中，确认图像已上传到 **images** 容器。

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. 确认缩略图已在名为 **thumbnails** 的容器中创建。

    ```azurecli
    az storage blob list --account-name <storage account name> -c thumbnails -o table
    ```

1. 获取缩略图的 URL。

    ```azurecli
    az storage blob url --account-name <storage account name> -c thumbnails -n <filename> --output tsv
    ```

    在浏览器中打开该 URL，确认缩略图已正确创建。

1. 在转到下一教程之前，请删除 **images** 和 **thumbnails** 容器中的所有文件。

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```
    ```azurecli
    az storage blob delete-batch -s thumbnails --account-name <storage account name>
    ```


## <a name="summary"></a>摘要

在本模块中，你创建了一个无服务器函数。只要有图像上传到 Blob 存储容器，该函数就会创建缩略图。 接下来，请学习如何使用 Azure Cosmos DB 来存储和列出图像元数据。
