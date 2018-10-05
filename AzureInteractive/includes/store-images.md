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
ms.openlocfilehash: 194a25dbf9abda80379aa5aab408ac4ffe9ab7f5
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460052"
---
<span data-ttu-id="6c299-103">Azure Cosmos DB 是 Microsoft 的无服务器全局分布式多模型数据库。</span><span class="sxs-lookup"><span data-stu-id="6c299-103">Azure Cosmos DB is Microsoft's serverless, globally distributed, multi-model database.</span></span> <span data-ttu-id="6c299-104">本模块介绍如何使用 Azure Functions 在 Cosmos DB 中存储和检索 JSON 文档形式的图像元数据。</span><span class="sxs-lookup"><span data-stu-id="6c299-104">In this module, you learn how to use Azure Functions to store and retrieve image metadata as JSON documents in Cosmos DB.</span></span>

## <a name="create-a-cosmos-db-account-database-and-collection"></a><span data-ttu-id="6c299-105">创建 Cosmos DB 帐户、数据库和集合</span><span class="sxs-lookup"><span data-stu-id="6c299-105">Create a Cosmos DB account, database, and collection</span></span>

<span data-ttu-id="6c299-106">Cosmos DB 帐户是包含 Cosmos DB 数据库的 Azure 资源。</span><span class="sxs-lookup"><span data-stu-id="6c299-106">A Cosmos DB account is an Azure resource that contains Cosmos DB databases.</span></span>

1. <span data-ttu-id="6c299-107">确保仍登录到 Cloud Shell。</span><span class="sxs-lookup"><span data-stu-id="6c299-107">Ensure you are still signed into the Cloud Shell.</span></span>  <span data-ttu-id="6c299-108">否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。</span><span class="sxs-lookup"><span data-stu-id="6c299-108">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="6c299-109">在本教程的其他资源所在的资源组中使用唯一名称创建一个 Cosmos DB 帐户。</span><span class="sxs-lookup"><span data-stu-id="6c299-109">Create a Cosmos DB account with a unique name in the same resource group as the other resources in this tutorial.</span></span>

    ```azurecli
    az cosmosdb create -g first-serverless-app -n <cosmos db account name>
    ```

1. <span data-ttu-id="6c299-110">在创建 Cosmos DB 帐户后，请在帐户中创建名为 **imagesdb** 的新数据库。</span><span class="sxs-lookup"><span data-stu-id="6c299-110">After the Cosmos DB account is created, create a new database named **imagesdb** in the account.</span></span>

    ```azurecli
    az cosmosdb database create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb
    ```

1. <span data-ttu-id="6c299-111">在创建数据库后，请在吞吐量为 400 个请求单位 (RU) 的数据库中创建名为 **images** 的新集合。</span><span class="sxs-lookup"><span data-stu-id="6c299-111">After the database is created, create a new collection named **images** in the database with a throughput of 400 request units (RUs).</span></span>

    ```azurecli
    az cosmosdb collection create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb --collection-name images --throughput 400
    ```


## <a name="save-a-document-to-cosmos-db-when-a-thumbnail-is-created"></a><span data-ttu-id="6c299-112">创建缩略图时将文档保存到 Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="6c299-112">Save a document to Cosmos DB when a thumbnail is created</span></span>

<span data-ttu-id="6c299-113">Cosmos DB 输出绑定允许在 Azure Functions 的 Cosmos DB 集合中创建文档。</span><span class="sxs-lookup"><span data-stu-id="6c299-113">The Cosmos DB output binding lets you create documents in a Cosmos DB collection from Azure Functions.</span></span> <span data-ttu-id="6c299-114">以下步骤在 **ResizeImage** 函数中配置 Cosmos DB 输出绑定，并修改该函数，以便返回要保存的文档（对象）。</span><span class="sxs-lookup"><span data-stu-id="6c299-114">In the following steps, you configure a Cosmos DB output binding in the **ResizeImage** function and modify the function to return a document (object) to be saved.</span></span>

1. <span data-ttu-id="6c299-115">在 Azure 门户中打开函数应用。</span><span class="sxs-lookup"><span data-stu-id="6c299-115">Open the function app in the Azure Portal.</span></span>

1. <span data-ttu-id="6c299-116">在左侧导航栏中展开 **ResizeImage** 函数，然后选择“集成”。</span><span class="sxs-lookup"><span data-stu-id="6c299-116">In the left hand navigation, expand the **ResizeImage** function, and then select **Integrate**.</span></span>

1. <span data-ttu-id="6c299-117">在“输出”下，单击“新建输出”。</span><span class="sxs-lookup"><span data-stu-id="6c299-117">Under **Outputs**, click **New Output**.</span></span>

1. <span data-ttu-id="6c299-118">找到“Azure Cosmos DB”项并将其选中。</span><span class="sxs-lookup"><span data-stu-id="6c299-118">Find the **Azure Cosmos DB** item and select it.</span></span> <span data-ttu-id="6c299-119">然后单击“选择”。</span><span class="sxs-lookup"><span data-stu-id="6c299-119">Then click **Select**.</span></span>

    ![选择“新建输出”](media/functions-first-serverless-web-app/4-new-output.jpg)

1. <span data-ttu-id="6c299-121">使用以下值填写“Azure Cosmos DB 输出”下的字段。</span><span class="sxs-lookup"><span data-stu-id="6c299-121">Fill out the fields under **Azure Cosmos DB output** with the following values.</span></span>

    | <span data-ttu-id="6c299-122">设置</span><span class="sxs-lookup"><span data-stu-id="6c299-122">Setting</span></span>      |  <span data-ttu-id="6c299-123">建议的值</span><span class="sxs-lookup"><span data-stu-id="6c299-123">Suggested value</span></span>   | <span data-ttu-id="6c299-124">Description</span><span class="sxs-lookup"><span data-stu-id="6c299-124">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="6c299-125">文档参数名称</span><span class="sxs-lookup"><span data-stu-id="6c299-125">**Document parameter name**</span></span> | <span data-ttu-id="6c299-126">选择“使用函数返回值”</span><span class="sxs-lookup"><span data-stu-id="6c299-126">Select **Use function return value**</span></span> | <span data-ttu-id="6c299-127">文本框的值自动设置为 **$return**。</span><span class="sxs-lookup"><span data-stu-id="6c299-127">The value of the textbox is automatically set to **$return**.</span></span> |
    | <span data-ttu-id="6c299-128">**数据库名称**</span><span class="sxs-lookup"><span data-stu-id="6c299-128">**Database name**</span></span> | <span data-ttu-id="6c299-129">imagesdb</span><span class="sxs-lookup"><span data-stu-id="6c299-129">imagesdb</span></span> | <span data-ttu-id="6c299-130">使用已创建的数据库的名称。</span><span class="sxs-lookup"><span data-stu-id="6c299-130">Use the name of the database that you created.</span></span> |
    | <span data-ttu-id="6c299-131">集合名称</span><span class="sxs-lookup"><span data-stu-id="6c299-131">**Collection name**</span></span> | <span data-ttu-id="6c299-132">images</span><span class="sxs-lookup"><span data-stu-id="6c299-132">images</span></span> | <span data-ttu-id="6c299-133">使用已创建的集合的名称。</span><span class="sxs-lookup"><span data-stu-id="6c299-133">Use the name of the collection that you created.</span></span> |

1. <span data-ttu-id="6c299-134">在“Azure Cosmos DB 帐户连接”旁边，单击“新建”。</span><span class="sxs-lookup"><span data-stu-id="6c299-134">Next to **Azure Cosmos DB account connection**, click **new**.</span></span> <span data-ttu-id="6c299-135">选择此前创建的 Cosmos DB 帐户。</span><span class="sxs-lookup"><span data-stu-id="6c299-135">Select the Cosmos DB account you previously created.</span></span>

    ![输入 Azure Cosmos DB 输出绑定的设置](media/functions-first-serverless-web-app/4-cosmos-db-output.png)

1. <span data-ttu-id="6c299-137">单击“保存”创建 Cosmos DB 输出绑定。</span><span class="sxs-lookup"><span data-stu-id="6c299-137">Click **Save** to create the Cosmos DB output binding.</span></span>

1. <span data-ttu-id="6c299-138">单击左侧的“ResizeImage”函数名称，打开该函数。</span><span class="sxs-lookup"><span data-stu-id="6c299-138">Click on the **ResizeImage** function name on the left to open the function.</span></span>

1. <span data-ttu-id="6c299-139">**C#**</span><span class="sxs-lookup"><span data-stu-id="6c299-139">**C#**</span></span>

    1. <span data-ttu-id="6c299-140">(C#) 将函数的返回类型从 **void** 更改为 **object**。</span><span class="sxs-lookup"><span data-stu-id="6c299-140">(C#) Change the return type of the function from **void** to **object**.</span></span>

    1. <span data-ttu-id="6c299-141">(C#) 在函数末尾添加以下代码块，以便返回要保存的文档：</span><span class="sxs-lookup"><span data-stu-id="6c299-141">(C#) At the end of the function, add the following code block to return the document to be saved:</span></span>
    
        ```csharp
        return new {
            id = name,
            imgPath = "/images/" + name,
            thumbnailPath = "/thumbnails/" + name
        };
        ```
    
        ![ResizeImages 函数的 run.csx（修改后）](media/functions-first-serverless-web-app/4-update-function.png)

1. <span data-ttu-id="6c299-143">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="6c299-143">**JavaScript**</span></span>

    1. <span data-ttu-id="6c299-144">(JavaScript) 更改 `else` 子句中的 `context.done()` 语句，以便返回要保存到 Cosmos DB 的文档。</span><span class="sxs-lookup"><span data-stu-id="6c299-144">(JavaScript) Change the `context.done()` statement in the `else` clause to return the document to be saved to Cosmos DB.</span></span>

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

1. <span data-ttu-id="6c299-145">单击代码窗口下的“日志”，展开日志面板。</span><span class="sxs-lookup"><span data-stu-id="6c299-145">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="6c299-146">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="6c299-146">Click **Save**.</span></span> <span data-ttu-id="6c299-147">查看日志面板，确保函数已成功保存且没有错误。</span><span class="sxs-lookup"><span data-stu-id="6c299-147">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="create-a-function-to-list-images-from-cosmos-db"></a><span data-ttu-id="6c299-148">创建一个可以列出 Cosmos DB 中图像的函数</span><span class="sxs-lookup"><span data-stu-id="6c299-148">Create a function to list images from Cosmos DB</span></span>

<span data-ttu-id="6c299-149">Web 应用程序需要使用 API 从 Cosmos DB 检索图像元数据。</span><span class="sxs-lookup"><span data-stu-id="6c299-149">The web application requires an API to retrieve image metadata from Cosmos DB.</span></span> <span data-ttu-id="6c299-150">在以下步骤中，</span><span class="sxs-lookup"><span data-stu-id="6c299-150">In the following steps.</span></span> <span data-ttu-id="6c299-151">请创建一个 HTTP 触发的函数，以便使用 Cosmos DB 输入绑定来查询数据库集合。</span><span class="sxs-lookup"><span data-stu-id="6c299-151">you create an HTTP triggered function that uses a Cosmos DB input binding to query the database collection.</span></span>

1. <span data-ttu-id="6c299-152">在函数应用中，将鼠标悬停在左侧的“函数”上方，然后单击“+”以创建新的函数。</span><span class="sxs-lookup"><span data-stu-id="6c299-152">In your function app, hover over **Functions** on the left and click **+** to create a new function.</span></span>

1. <span data-ttu-id="6c299-153">找到 **HttpTrigger** 模板并将其选中。</span><span class="sxs-lookup"><span data-stu-id="6c299-153">Find the **HttpTrigger** template and select it.</span></span>

1. <span data-ttu-id="6c299-154">使用这些值创建一个可生成获取图像 URL 的函数。</span><span class="sxs-lookup"><span data-stu-id="6c299-154">Use these values to create a function that generates a get images URL.</span></span>

    | <span data-ttu-id="6c299-155">设置</span><span class="sxs-lookup"><span data-stu-id="6c299-155">Setting</span></span>      |  <span data-ttu-id="6c299-156">建议的值</span><span class="sxs-lookup"><span data-stu-id="6c299-156">Suggested value</span></span>   | <span data-ttu-id="6c299-157">Description</span><span class="sxs-lookup"><span data-stu-id="6c299-157">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="6c299-158">**为函数命名**</span><span class="sxs-lookup"><span data-stu-id="6c299-158">**Name your function**</span></span> | <span data-ttu-id="6c299-159">GetImages</span><span class="sxs-lookup"><span data-stu-id="6c299-159">GetImages</span></span> | <span data-ttu-id="6c299-160">根据显示准确地键入此名称，使应用程序能够发现此函数。</span><span class="sxs-lookup"><span data-stu-id="6c299-160">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="6c299-161">**授权级别**</span><span class="sxs-lookup"><span data-stu-id="6c299-161">**Authorization level**</span></span> | <span data-ttu-id="6c299-162">匿名</span><span class="sxs-lookup"><span data-stu-id="6c299-162">Anonymous</span></span> | <span data-ttu-id="6c299-163">允许公开访问此函数。</span><span class="sxs-lookup"><span data-stu-id="6c299-163">Allow the function to be accessed publicly.</span></span> |

1. <span data-ttu-id="6c299-164">单击“创建”。</span><span class="sxs-lookup"><span data-stu-id="6c299-164">Click **Create**.</span></span>

1. <span data-ttu-id="6c299-165">创建新函数后，单击左导航栏中函数名称下的“集成”。</span><span class="sxs-lookup"><span data-stu-id="6c299-165">When the new function is created, click **Integrate** under the function's name on the left navigation.</span></span>

1. <span data-ttu-id="6c299-166">单击“新建输入”，然后选择“Azure Cosmos DB”。</span><span class="sxs-lookup"><span data-stu-id="6c299-166">Click **New Input** and select **Azure Cosmos DB**.</span></span> 

    ![选择“新建输入”](media/functions-first-serverless-web-app/4-new-input.jpg)

1. <span data-ttu-id="6c299-168">单击“选择”。</span><span class="sxs-lookup"><span data-stu-id="6c299-168">Click **Select**.</span></span>

1. <span data-ttu-id="6c299-169">填充以下值：</span><span class="sxs-lookup"><span data-stu-id="6c299-169">Fill out the following values:</span></span>

    | <span data-ttu-id="6c299-170">设置</span><span class="sxs-lookup"><span data-stu-id="6c299-170">Setting</span></span>      |  <span data-ttu-id="6c299-171">建议的值</span><span class="sxs-lookup"><span data-stu-id="6c299-171">Suggested value</span></span>   | <span data-ttu-id="6c299-172">Description</span><span class="sxs-lookup"><span data-stu-id="6c299-172">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="6c299-173">文档参数名称</span><span class="sxs-lookup"><span data-stu-id="6c299-173">**Document parameter name**</span></span> | <span data-ttu-id="6c299-174">documents</span><span class="sxs-lookup"><span data-stu-id="6c299-174">documents</span></span> | <span data-ttu-id="6c299-175">与函数中的参数名称匹配。</span><span class="sxs-lookup"><span data-stu-id="6c299-175">Matches parameter name in the function.</span></span> |
    | <span data-ttu-id="6c299-176">**数据库名称**</span><span class="sxs-lookup"><span data-stu-id="6c299-176">**Database name**</span></span> | <span data-ttu-id="6c299-177">imagesdb</span><span class="sxs-lookup"><span data-stu-id="6c299-177">imagesdb</span></span> |  |
    | <span data-ttu-id="6c299-178">集合名称</span><span class="sxs-lookup"><span data-stu-id="6c299-178">**Collection name**</span></span> | <span data-ttu-id="6c299-179">images</span><span class="sxs-lookup"><span data-stu-id="6c299-179">images</span></span> |  |
    | <span data-ttu-id="6c299-180">**SQL 查询**</span><span class="sxs-lookup"><span data-stu-id="6c299-180">**SQL query**</span></span> | <span data-ttu-id="6c299-181">select \* from c order by c._ts desc</span><span class="sxs-lookup"><span data-stu-id="6c299-181">select \* from c order by c._ts desc</span></span> | <span data-ttu-id="6c299-182">获取文档，最新文档优先。</span><span class="sxs-lookup"><span data-stu-id="6c299-182">Get documents, latest documents first.</span></span> |
    | <span data-ttu-id="6c299-183">**Azure Cosmos DB 帐户连接**</span><span class="sxs-lookup"><span data-stu-id="6c299-183">**Azure Cosmos DB account connection**</span></span> | <span data-ttu-id="6c299-184">选择现有连接字符串</span><span class="sxs-lookup"><span data-stu-id="6c299-184">Select existing connection string</span></span> |  |

1. <span data-ttu-id="6c299-185">单击“保存”创建输入绑定。</span><span class="sxs-lookup"><span data-stu-id="6c299-185">Click **Save** to create the input binding.</span></span>

1. <span data-ttu-id="6c299-186">**C#**</span><span class="sxs-lookup"><span data-stu-id="6c299-186">**C#**</span></span>

    1. <span data-ttu-id="6c299-187">单击函数的名称以打开代码窗口，然后将所有 **run.csx** 替换为 [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx) 中的内容。</span><span class="sxs-lookup"><span data-stu-id="6c299-187">Click the function's name to open the code window, and then replace all of **run.csx** with the content in [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx).</span></span>

1. <span data-ttu-id="6c299-188">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="6c299-188">**JavaScript**</span></span>

    1. <span data-ttu-id="6c299-189">单击函数的名称以打开代码窗口，然后将所有 **index.js** 替换为 [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js) 中的内容。</span><span class="sxs-lookup"><span data-stu-id="6c299-189">Click the function's name to open the code window, and then replace all of **index.js** with the content in [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js).</span></span>

1. <span data-ttu-id="6c299-190">单击代码窗口下的“日志”，展开日志面板。</span><span class="sxs-lookup"><span data-stu-id="6c299-190">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="6c299-191">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="6c299-191">Click **Save**.</span></span> <span data-ttu-id="6c299-192">查看日志面板，确保函数已成功保存且没有错误。</span><span class="sxs-lookup"><span data-stu-id="6c299-192">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-application"></a><span data-ttu-id="6c299-193">测试应用程序</span><span class="sxs-lookup"><span data-stu-id="6c299-193">Test the application</span></span>

1. <span data-ttu-id="6c299-194">在浏览器中打开应用程序。</span><span class="sxs-lookup"><span data-stu-id="6c299-194">Open the application in a browser.</span></span> <span data-ttu-id="6c299-195">选择一个图像文件并将其上传。</span><span class="sxs-lookup"><span data-stu-id="6c299-195">Select an image file and upload it.</span></span>

1. <span data-ttu-id="6c299-196">几秒钟后，新图像的缩略图会显示在页面中。</span><span class="sxs-lookup"><span data-stu-id="6c299-196">After a few seconds, the thumbnail of the new image appears on the page.</span></span>

1. <span data-ttu-id="6c299-197">在 Azure 门户中，使用搜索框按名称搜索 Cosmos DB 帐户。</span><span class="sxs-lookup"><span data-stu-id="6c299-197">In the Azure portal, use the Search box to search for your Cosmos DB account by name.</span></span> <span data-ttu-id="6c299-198">以单击方式将其打开。</span><span class="sxs-lookup"><span data-stu-id="6c299-198">Click it to open it.</span></span>

1. <span data-ttu-id="6c299-199">单击左侧的“数据资源管理器”，浏览集合和文档。</span><span class="sxs-lookup"><span data-stu-id="6c299-199">Click **Data Explorer** on the left to browse collections and documents.</span></span>

1. <span data-ttu-id="6c299-200">在“imagesdb”数据库下，选择“images”集合。</span><span class="sxs-lookup"><span data-stu-id="6c299-200">Under the **imagesdb** database, select the **images** collection.</span></span>

1. <span data-ttu-id="6c299-201">确认已为上传的图像创建一个文档。</span><span class="sxs-lookup"><span data-stu-id="6c299-201">Confirm that a document was created for the uploaded image.</span></span>

    ![数据资源管理器，显示已上传图像的文档](media/functions-first-serverless-web-app/4-data-explorer.png)



## <a name="summary"></a><span data-ttu-id="6c299-203">摘要</span><span class="sxs-lookup"><span data-stu-id="6c299-203">Summary</span></span>

<span data-ttu-id="6c299-204">本模块介绍了如何创建 Cosmos DB 帐户、数据库和集合，</span><span class="sxs-lookup"><span data-stu-id="6c299-204">In this module, you learned how to create a Cosmos DB account, database, and collection.</span></span> <span data-ttu-id="6c299-205">另外还介绍了如何使用 Cosmos DB 绑定在 Cosmos DB 集合中保存和检索图像元数据。</span><span class="sxs-lookup"><span data-stu-id="6c299-205">You also learned how to use the Cosmos DB bindings to save and retrieve image metadata in the Cosmos DB collection.</span></span> <span data-ttu-id="6c299-206">接下来介绍如何使用 Microsoft 认知服务为每个上传的图像自动生成标题。</span><span class="sxs-lookup"><span data-stu-id="6c299-206">Next, you learn how to automatically generate a caption for each uploaded image using Microsoft Cognitive Services.</span></span>
