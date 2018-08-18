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
ms.openlocfilehash: 2202cdebe77668972372983a0e802d00edabf6dd
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079124"
---
Azure Cosmos DB 是 Microsoft 的无服务器全局分布式多模型数据库。 本模块介绍如何使用 Azure Functions 在 Cosmos DB 中存储和检索 JSON 文档形式的图像元数据。

## <a name="create-a-cosmos-db-account-database-and-collection"></a>创建 Cosmos DB 帐户、数据库和集合

Cosmos DB 帐户是包含 Cosmos DB 数据库的 Azure 资源。

1. 确保仍登录到 Cloud Shell。  否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。 

1. 在本教程的其他资源所在的资源组中使用唯一名称创建一个 Cosmos DB 帐户。

    ```azurecli
    az cosmosdb create -g first-serverless-app -n <cosmos db account name>
    ```

1. 在创建 Cosmos DB 帐户后，请在帐户中创建名为 **imagesdb** 的新数据库。

    ```azurecli
    az cosmosdb database create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb
    ```

1. 在创建数据库后，请在吞吐量为 400 个请求单位 (RU) 的数据库中创建名为 **images** 的新集合。

    ```azurecli
    az cosmosdb collection create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb --collection-name images --throughput 400
    ```


## <a name="save-a-document-to-cosmos-db-when-a-thumbnail-is-created"></a>创建缩略图时将文档保存到 Cosmos DB

Cosmos DB 输出绑定允许在 Azure Functions 的 Cosmos DB 集合中创建文档。 以下步骤在 **ResizeImage** 函数中配置 Cosmos DB 输出绑定，并修改该函数，以便返回要保存的文档（对象）。

1. 在 Azure 门户中打开函数应用。

1. 在左侧导航栏中展开 **ResizeImage** 函数，然后选择“集成”。

1. 在“输出”下，单击“新建输出”。

1. 找到“Azure Cosmos DB”项并将其选中。 然后单击“选择”。

    ![选择“新建输出”](media/functions-first-serverless-web-app/4-new-output.jpg)

1. 使用以下值填写“Azure Cosmos DB 输出”下的字段。

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | 文档参数名称 | 选择“使用函数返回值” | 文本框的值自动设置为 **$return**。 |
    | **数据库名称** | imagesdb | 使用已创建的数据库的名称。 |
    | 集合名称 | images | 使用已创建的集合的名称。 |

1. 在“Azure Cosmos DB 帐户连接”旁边，单击“新建”。 选择此前创建的 Cosmos DB 帐户。

    ![输入 Azure Cosmos DB 输出绑定的设置](media/functions-first-serverless-web-app/4-cosmos-db-output.png)

1. 单击“保存”创建 Cosmos DB 输出绑定。

1. 单击左侧的“ResizeImage”函数名称，打开该函数。

1. **C#**

    1. (C#) 将函数的返回类型从 **void** 更改为 **object**。

    1. (C#) 在函数末尾添加以下代码块，以便返回要保存的文档：
    
        ```csharp
        return new {
            id = name,
            imgPath = "/images/" + name,
            thumbnailPath = "/thumbnails/" + name
        };
        ```
    
        ![ResizeImages 函数的 run.csx（修改后）](media/functions-first-serverless-web-app/4-update-function.png)

1. **JavaScript**

    1. (JavaScript) 更改 `else` 子句中的 `context.done()` 语句，以便返回要保存到 Cosmos DB 的文档。

    ```javascript
    if (error) {
        context.done(error);
    } else {
        context.bindings.thumbnail = stream;
        context.done(null, {
            id: context.bindingData.name,
            imgPath: "/images/" + context.bindingData.name,
            thumbnailPath: "/thumbnails/" + context.bindingData.name
        });
    }
    ```

1. 单击代码窗口下的“日志”，展开日志面板。

1. 单击“ **保存**”。 查看日志面板，确保函数已成功保存且没有错误。


## <a name="create-a-function-to-list-images-from-cosmos-db"></a>创建一个可以列出 Cosmos DB 中图像的函数

Web 应用程序需要使用 API 从 Cosmos DB 检索图像元数据。 在以下步骤中， 请创建一个 HTTP 触发的函数，以便使用 Cosmos DB 输入绑定来查询数据库集合。

1. 在函数应用中，将鼠标悬停在左侧的“函数”上方，然后单击“+”以创建新的函数。

1. 找到 **HttpTrigger** 模板并将其选中。

1. 使用这些值创建一个可生成获取图像 URL 的函数。

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | **为函数命名** | GetImages | 根据显示准确地键入此名称，使应用程序能够发现此函数。 |
    | **授权级别** | 匿名 | 允许公开访问此函数。 |

1. 单击“创建”。

1. 创建新函数后，单击左导航栏中函数名称下的“集成”。

1. 单击“新建输入”，然后选择“Azure Cosmos DB”。 

    ![选择“新建输入”](media/functions-first-serverless-web-app/4-new-input.jpg)

1. 单击“选择”。

1. 填充以下值：

    | 设置      |  建议的值   | Description                                        |
    | --- | --- | ---|
    | 文档参数名称 | documents | 与函数中的参数名称匹配。 |
    | **数据库名称** | imagesdb |  |
    | 集合名称 | images |  |
    | **SQL 查询** | select * from c order by c._ts desc | 获取文档，最新文档优先。 |
    | **Azure Cosmos DB 帐户连接** | 选择现有连接字符串 |  |

1. 单击“保存”创建输入绑定。

1. **C#**

    1. 单击函数的名称以打开代码窗口，然后将所有 **run.csx** 替换为 [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx) 中的内容。

1. **JavaScript**

    1. 单击函数的名称以打开代码窗口，然后将所有 **index.js** 替换为 [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js) 中的内容。

1. 单击代码窗口下的“日志”，展开日志面板。

1. 单击“ **保存**”。 查看日志面板，确保函数已成功保存且没有错误。


## <a name="test-the-application"></a>测试应用程序

1. 在浏览器中打开应用程序。 选择一个图像文件并将其上传。

1. 几秒钟后，新图像的缩略图会显示在页面中。

1. 在 Azure 门户中，使用搜索框按名称搜索 Cosmos DB 帐户。 以单击方式将其打开。

1. 单击左侧的“数据资源管理器”，浏览集合和文档。

1. 在“imagesdb”数据库下，选择“images”集合。

1. 确认已为上传的图像创建一个文档。

    ![数据资源管理器，显示已上传图像的文档](media/functions-first-serverless-web-app/4-data-explorer.png)



## <a name="summary"></a>摘要

本模块介绍了如何创建 Cosmos DB 帐户、数据库和集合， 另外还介绍了如何使用 Cosmos DB 绑定在 Cosmos DB 集合中保存和检索图像元数据。 接下来介绍如何使用 Microsoft 认知服务为每个上传的图像自动生成标题。