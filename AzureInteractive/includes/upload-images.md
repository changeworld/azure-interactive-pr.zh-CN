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
ms.openlocfilehash: 56cfb4c2893977086309660f4f6941fd0d648913
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079127"
---
<span data-ttu-id="64fb0-103">要生成的应用程序是一个照片库。</span><span class="sxs-lookup"><span data-stu-id="64fb0-103">The application that you're building is a photo gallery.</span></span> <span data-ttu-id="64fb0-104">它使用客户端 JavaScript 来调用 API，以便上传和显示图像。</span><span class="sxs-lookup"><span data-stu-id="64fb0-104">It uses client-side JavaScript to call APIs to upload and display images.</span></span> <span data-ttu-id="64fb0-105">在此模块中，请使用无服务器函数来创建 API，该函数生成用于上传图像的具有时间限制的 URL。</span><span class="sxs-lookup"><span data-stu-id="64fb0-105">In this module, you create an API using a serverless function that generates a time-limited URL to upload an image.</span></span> <span data-ttu-id="64fb0-106">Web 应用程序使用 [Blob 存储 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 通过生成的 URL 将图像上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="64fb0-106">The web application uses the generated URL to upload an image to Blob storage using the [Blob storage REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api).</span></span>

## <a name="create-a-blob-storage-container-for-images"></a><span data-ttu-id="64fb0-107">为图像创建 Blob 存储容器</span><span class="sxs-lookup"><span data-stu-id="64fb0-107">Create a blob storage container for images</span></span>

<span data-ttu-id="64fb0-108">应用程序需要单独的存储容器来上传并托管图像。</span><span class="sxs-lookup"><span data-stu-id="64fb0-108">The application requires a separate storage container to upload and host images.</span></span>

1. <span data-ttu-id="64fb0-109">确保仍登录到 Cloud Shell (bash)。</span><span class="sxs-lookup"><span data-stu-id="64fb0-109">Ensure you are still signed in to the Cloud Shell (bash).</span></span> <span data-ttu-id="64fb0-110">否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。</span><span class="sxs-lookup"><span data-stu-id="64fb0-110">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span>

1.  <span data-ttu-id="64fb0-111">在可以公开访问所有 Blob 的存储帐户中创建新的名为 **images** 的容器。</span><span class="sxs-lookup"><span data-stu-id="64fb0-111">Create a new container named **images** in your storage account with public access to all blobs.</span></span>

    ```azurecli
    az storage container create -n images --account-name <storage account name> --public-access blob
    ```

## <a name="create-an-azure-function-app"></a><span data-ttu-id="64fb0-112">创建 Azure Function App</span><span class="sxs-lookup"><span data-stu-id="64fb0-112">Create an Azure Function app</span></span>

<span data-ttu-id="64fb0-113">Azure Functions 是一项用于运行无服务器函数的服务。</span><span class="sxs-lookup"><span data-stu-id="64fb0-113">Azure Functions is a service for running serverless functions.</span></span> <span data-ttu-id="64fb0-114">无服务器函数可以通过事件（例如发出了 HTTP 请求，或者在存储容器中创建了 Blob）触发（调用）。</span><span class="sxs-lookup"><span data-stu-id="64fb0-114">A serverless function can be triggered (called) by events such as an HTTP request or when a blob is created in a storage container.</span></span>

<span data-ttu-id="64fb0-115">Azure 函数应用是一个或多个无服务器函数的容器。</span><span class="sxs-lookup"><span data-stu-id="64fb0-115">An Azure Function app is a container for one or more serverless functions.</span></span>

1. <span data-ttu-id="64fb0-116">在此前创建的资源组中使用唯一名称创建名为 **first-serverless-app** 的新 Azure 函数应用。</span><span class="sxs-lookup"><span data-stu-id="64fb0-116">Create a new Azure Function app with a unique name in the resource group you created earlier named **first-serverless-app**.</span></span> <span data-ttu-id="64fb0-117">函数应用需要一个存储帐户；在本教程中，请使用现有的存储帐户。</span><span class="sxs-lookup"><span data-stu-id="64fb0-117">Function apps require a Storage account; in this tutorial, you use the existing storage account.</span></span>

    ```azurecli
    az functionapp create -n <function app name> -g first-serverless-app -s <storage account name> -c westcentralus
    ```


## <a name="create-an-http-triggered-serverless-function"></a><span data-ttu-id="64fb0-118">创建 HTTP 触发的无服务器函数</span><span class="sxs-lookup"><span data-stu-id="64fb0-118">Create an HTTP-triggered serverless function</span></span>

<span data-ttu-id="64fb0-119">照片库 Web 应用程序向无服务器函数发出 HTTP 请求，请求生成一个具有时间限制的 URL，以便将图像安全地上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="64fb0-119">The photo gallery web application makes an HTTP request to serverless function to generate a time-limited URL to securely upload an image to Blob storage.</span></span> <span data-ttu-id="64fb0-120">此函数由 HTTP 请求触发，可以使用 Azure 存储 SDK 生成并返回安全的 URL。</span><span class="sxs-lookup"><span data-stu-id="64fb0-120">The function is triggered by an HTTP request and uses the Azure Storage SDK to generate and return the secure URL.</span></span>

1. <span data-ttu-id="64fb0-121">创建函数应用以后，请使用搜索框在 Azure 门户中搜索它，然后以单击方式将它打开。</span><span class="sxs-lookup"><span data-stu-id="64fb0-121">After the Function app is created, search for it in the Azure Portal using the Search box and click to open it.</span></span>

    ![打开函数应用](media/functions-first-serverless-web-app/2-search-function-app.png)

1. <span data-ttu-id="64fb0-123">在函数应用窗口的左侧导航栏中，将鼠标悬停在“函数”上，然后单击“+”开始创建新的无服务器函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-123">In the function app window's left hand navigation, hover over **Functions** and click **+** to start creating a new serverless function.</span></span>

    ![创建新函数](media/functions-first-serverless-web-app/2-new-function.png)

1. <span data-ttu-id="64fb0-125">单击“自定义函数”即可看到函数模板的列表。</span><span class="sxs-lookup"><span data-stu-id="64fb0-125">Click **Custom function** to see a list of function templates.</span></span>

1. <span data-ttu-id="64fb0-126">找到 **HttpTrigger** 模板，单击要使用的语言（C# 或 JavaScript）。</span><span class="sxs-lookup"><span data-stu-id="64fb0-126">Find the **HttpTrigger** template and click the language to use (C# or JavaScript).</span></span>

1. <span data-ttu-id="64fb0-127">使用这些值创建一个可生成 Blob 上传 URL 的函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-127">Use these values to create a function that generates a blob upload URL.</span></span>

    | <span data-ttu-id="64fb0-128">设置</span><span class="sxs-lookup"><span data-stu-id="64fb0-128">Setting</span></span>      |  <span data-ttu-id="64fb0-129">建议的值</span><span class="sxs-lookup"><span data-stu-id="64fb0-129">Suggested value</span></span>   | <span data-ttu-id="64fb0-130">Description</span><span class="sxs-lookup"><span data-stu-id="64fb0-130">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="64fb0-131">**语言**</span><span class="sxs-lookup"><span data-stu-id="64fb0-131">**Language**</span></span> | <span data-ttu-id="64fb0-132">C# 或 JavaScript</span><span class="sxs-lookup"><span data-stu-id="64fb0-132">C# or JavaScript</span></span> | <span data-ttu-id="64fb0-133">选择要使用的语言。</span><span class="sxs-lookup"><span data-stu-id="64fb0-133">Select the language you want to use.</span></span> |
    | <span data-ttu-id="64fb0-134">**为函数命名**</span><span class="sxs-lookup"><span data-stu-id="64fb0-134">**Name your function**</span></span> | <span data-ttu-id="64fb0-135">GetUploadUrl</span><span class="sxs-lookup"><span data-stu-id="64fb0-135">GetUploadUrl</span></span> | <span data-ttu-id="64fb0-136">根据显示准确地键入此名称，使应用程序能够发现此函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-136">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="64fb0-137">**授权级别**</span><span class="sxs-lookup"><span data-stu-id="64fb0-137">**Authorization level**</span></span> | <span data-ttu-id="64fb0-138">匿名</span><span class="sxs-lookup"><span data-stu-id="64fb0-138">Anonymous</span></span> | <span data-ttu-id="64fb0-139">允许公开访问此函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-139">Allow the function to be accessed publicly.</span></span> |

    ![为 HTTP 触发的新函数输入设置](media/functions-first-serverless-web-app/2-new-function-httptrigger.png)

1. <span data-ttu-id="64fb0-141">单击“创建”创建该函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-141">Click **Create** to create the function.</span></span>

1. <span data-ttu-id="64fb0-142">**C#**</span><span class="sxs-lookup"><span data-stu-id="64fb0-142">**C#**</span></span> 

    1. <span data-ttu-id="64fb0-143">当函数的源代码显示时，请将所有 **run.csx** 替换为 [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx) 的内容。</span><span class="sxs-lookup"><span data-stu-id="64fb0-143">When the function's source code appears, replace all of **run.csx** with the content of [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx).</span></span>

1. <span data-ttu-id="64fb0-144">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="64fb0-144">**JavaScript**</span></span> 

    1. <span data-ttu-id="64fb0-145">(JavaScript) 此函数需要 npm 中的 `azure-storage` 包来生成共享访问签名 (SAS) 令牌，该令牌是生成安全 URL 所需的。</span><span class="sxs-lookup"><span data-stu-id="64fb0-145">(JavaScript) This function requires the `azure-storage` package from npm to generate the shared access signature (SAS) token required to build the secure URL.</span></span> <span data-ttu-id="64fb0-146">若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-146">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="64fb0-147">(JavaScript) 单击“控制台”以显示控制台窗口。</span><span class="sxs-lookup"><span data-stu-id="64fb0-147">(JavaScript) Click **Console** to reveal a console window.</span></span>

        ![打开控制台窗口](media/functions-first-serverless-web-app/2-open-console.jpg)

    1. <span data-ttu-id="64fb0-149">(JavaScript) 运行 `cd d:\home\site\wwwroot` 命令，确保当前目录为 **d:\home\site\wwwroot**。</span><span class="sxs-lookup"><span data-stu-id="64fb0-149">(JavaScript) Ensure the current directory is **d:\home\site\wwwroot** by running the command `cd d:\home\site\wwwroot`.</span></span>

    1. <span data-ttu-id="64fb0-150">(JavaScript) 运行 `npm init -y` 命令，创建空的 **package.json** 文件。</span><span class="sxs-lookup"><span data-stu-id="64fb0-150">(JavaScript) Run the command `npm init -y` to create an empty **package.json** file.</span></span>

    1. <span data-ttu-id="64fb0-151">(JavaScript) 在控制台中运行 `npm install --save azure-storage` 命令，以便安装包并将其保存到 **package.json** 中。</span><span class="sxs-lookup"><span data-stu-id="64fb0-151">(JavaScript) Run the command `npm install --save azure-storage` in the console to install the package and save it in **package.json**.</span></span> <span data-ttu-id="64fb0-152">完成此操作可能需要一到两分钟。</span><span class="sxs-lookup"><span data-stu-id="64fb0-152">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="64fb0-153">(JavaScript) 在左侧导航栏中单击函数名称 (**GetUploadUrl**) 以显示该函数，将所有 **index.js** 替换为 [**javascript/GetUploadUrl/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js) 的内容。</span><span class="sxs-lookup"><span data-stu-id="64fb0-153">(JavaScript) Click on the function name (**GetUploadUrl**) in the left navigation to reveal the function, replace all of **index.js** with the content of [**javascript/GetUploadUrl/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js).</span></span>
    
        ![更新后的 index.js](media/functions-first-serverless-web-app/2-paste-js.jpg)

1. <span data-ttu-id="64fb0-155">单击代码窗口下的“日志”，展开日志面板。</span><span class="sxs-lookup"><span data-stu-id="64fb0-155">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="64fb0-156">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-156">Click **Save**.</span></span> <span data-ttu-id="64fb0-157">查看日志面板，确保函数已成功编译。</span><span class="sxs-lookup"><span data-stu-id="64fb0-157">Check the logs panel to ensure the function is successfully compiled.</span></span>

<span data-ttu-id="64fb0-158">此函数生成的东西称为共享访问签名 (SAS) URL，用于将文件上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="64fb0-158">The function generates what is called a shared access signature (SAS) URL that is used to upload a file to Blob storage.</span></span> <span data-ttu-id="64fb0-159">此 SAS URL 在短时间内有效，仅允许上传单个文件。</span><span class="sxs-lookup"><span data-stu-id="64fb0-159">The SAS URL is valid for a short period of time and only allows a single file to be uploaded.</span></span> <span data-ttu-id="64fb0-160">若要详细了解如何[使用共享访问签名](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)，请查看 Blob 存储文档。</span><span class="sxs-lookup"><span data-stu-id="64fb0-160">Consult the Blob storage documentation for more information on [using shared access signatures](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1).</span></span>


## <a name="add-an-environment-variable-for-the-storage-connection-string"></a><span data-ttu-id="64fb0-161">添加存储连接字符串的环境变量</span><span class="sxs-lookup"><span data-stu-id="64fb0-161">Add an environment variable for the storage connection string</span></span>

<span data-ttu-id="64fb0-162">刚创建的函数需要一个用于存储帐户的连接字符串，以便生成 SAS URL。</span><span class="sxs-lookup"><span data-stu-id="64fb0-162">The function you just created requires a connection string for the Storage account so that it can generate the SAS URL.</span></span> <span data-ttu-id="64fb0-163">可以将连接字符串存储为应用程序设置，而不要在函数的正文中对其进行硬编码。</span><span class="sxs-lookup"><span data-stu-id="64fb0-163">Instead of hardcoding the connection string in the function's body, it can be stored as an application setting.</span></span> <span data-ttu-id="64fb0-164">在函数应用中，应用程序设置可以作为环境变量供所有函数访问。</span><span class="sxs-lookup"><span data-stu-id="64fb0-164">Application settings are accessible as environment variables by all functions in the Function app.</span></span>

1. <span data-ttu-id="64fb0-165">在 Cloud Shell 中，查询存储帐户连接字符串并将其保存在名为 **STORAGE_CONNECTION_STRING** 的 bash 变量中。</span><span class="sxs-lookup"><span data-stu-id="64fb0-165">In the Cloud Shell, query the Storage account connection string and save it to a bash variable named **STORAGE_CONNECTION_STRING**.</span></span>

    ```azurecli
    export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n <storage account name> -g first-serverless-app --query "connectionString" --output tsv)
    ```

    <span data-ttu-id="64fb0-166">确认变量已成功设置。</span><span class="sxs-lookup"><span data-stu-id="64fb0-166">Confirm the variable is set successfully.</span></span>

    ```azurecli
    echo $STORAGE_CONNECTION_STRING
    ```

1. <span data-ttu-id="64fb0-167">使用在上一步保存的值，创建名为 **AZURE_STORAGE_CONNECTION_STRING** 的新应用程序设置。</span><span class="sxs-lookup"><span data-stu-id="64fb0-167">Create a new application setting named **AZURE_STORAGE_CONNECTION_STRING** using the value saved from the previous step.</span></span>

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING -o table
    ```

    <span data-ttu-id="64fb0-168">确认命令的输出包含新应用程序设置且值正确。</span><span class="sxs-lookup"><span data-stu-id="64fb0-168">Confirm that the command's output contains the new application setting with the correct value.</span></span>


## <a name="test-the-serverless-function"></a><span data-ttu-id="64fb0-169">测试无服务器函数</span><span class="sxs-lookup"><span data-stu-id="64fb0-169">Test the serverless function</span></span>

<span data-ttu-id="64fb0-170">除了创建和编辑函数，Azure 门户还提供内置的用于测试函数的工具。</span><span class="sxs-lookup"><span data-stu-id="64fb0-170">In addition to creating and editing functions, the Azure portal also provides an built-in tool for testing functions.</span></span>

1. <span data-ttu-id="64fb0-171">若要测试 HTTP 无服务器函数，请单击代码窗口右侧的“测试”选项卡，展开测试面板。</span><span class="sxs-lookup"><span data-stu-id="64fb0-171">To test the HTTP serverless function, click on the **Test** tab on the right of the code window to expand the test panel.</span></span>

1. <span data-ttu-id="64fb0-172">将“Http 方法”更改为 **GET**。</span><span class="sxs-lookup"><span data-stu-id="64fb0-172">Change the **Http method** to **GET**.</span></span>

1. <span data-ttu-id="64fb0-173">在“查询”下单击“添加参数”\*，然后添加以下参数：</span><span class="sxs-lookup"><span data-stu-id="64fb0-173">Under **Query**, click *Add parameter*\* and add the following parameter:</span></span>

    | <span data-ttu-id="64fb0-174">名称</span><span class="sxs-lookup"><span data-stu-id="64fb0-174">name</span></span>      |  <span data-ttu-id="64fb0-175">值</span><span class="sxs-lookup"><span data-stu-id="64fb0-175">value</span></span>   | 
    | --- | --- |
    | <span data-ttu-id="64fb0-176">filename</span><span class="sxs-lookup"><span data-stu-id="64fb0-176">filename</span></span> | <span data-ttu-id="64fb0-177">image1.jpg</span><span class="sxs-lookup"><span data-stu-id="64fb0-177">image1.jpg</span></span> |

1. <span data-ttu-id="64fb0-178">单击测试面板中的“运行”，向函数发送 HTTP 请求。</span><span class="sxs-lookup"><span data-stu-id="64fb0-178">Click **Run** in the test panel to send an HTTP request to the function.</span></span>

1. <span data-ttu-id="64fb0-179">函数在输出中返回上传 URL。</span><span class="sxs-lookup"><span data-stu-id="64fb0-179">The function returns an upload URL in the output.</span></span> <span data-ttu-id="64fb0-180">函数的执行情况显示在日志面板中。</span><span class="sxs-lookup"><span data-stu-id="64fb0-180">The function execution appears in the logs panel.</span></span>

    ![显示函数已成功运行的日志](media/functions-first-serverless-web-app/2-test-function.png)


## <a name="configure-cors-in-the-function-app"></a><span data-ttu-id="64fb0-182">在函数应用中配置 CORS</span><span class="sxs-lookup"><span data-stu-id="64fb0-182">Configure CORS in the function app</span></span>

<span data-ttu-id="64fb0-183">由于此应用的前端托管在 Blob 存储中，因此其域名不同于 Azure 函数应用的域名。</span><span class="sxs-lookup"><span data-stu-id="64fb0-183">Because the app's frontend is hosted in Blob storage, it has a different domain name than the Azure Function app.</span></span> <span data-ttu-id="64fb0-184">需将函数应用配置为进行跨域资源共享 (CORS)，以便客户端 JavaScript 成功调用刚创建的函数。</span><span class="sxs-lookup"><span data-stu-id="64fb0-184">For the client-side JavaScript to successfully call the function you just created, the function app needs to be configured for cross-origin resource sharing (CORS).</span></span>

1. <span data-ttu-id="64fb0-185">在函数应用窗口的左导航栏中，单击函数应用的名称。</span><span class="sxs-lookup"><span data-stu-id="64fb0-185">In the left navigation bar of the function app window, click on the name of your function app.</span></span>

1. <span data-ttu-id="64fb0-186">单击“平台功能”，查看高级功能的列表。</span><span class="sxs-lookup"><span data-stu-id="64fb0-186">Click on **Platform features** to view a list of advanced features.</span></span>

1. <span data-ttu-id="64fb0-187">在“API”下，单击“CORS”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-187">Under **API**, click **CORS**.</span></span>

    ![选择 CORS](media/functions-first-serverless-web-app/2-open-cors.jpg)

1. <span data-ttu-id="64fb0-189">添加在上一模块中提供的应用程序 URL 的允许源，省略尾随 **/**（例如：`https://firstserverlessweb.z4.web.core.windows.net`）。</span><span class="sxs-lookup"><span data-stu-id="64fb0-189">Add an allow origin for the application URL from the previous module, omitting the trailing **/** (for example: `https://firstserverlessweb.z4.web.core.windows.net`).</span></span>

    ![显示无服务器 Web 应用 URL 已添加的 CORS 设置](media/functions-first-serverless-web-app/2-add-cors.png)

1. <span data-ttu-id="64fb0-191">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-191">Click **Save**.</span></span>

1. <span data-ttu-id="64fb0-192">C#</span><span class="sxs-lookup"><span data-stu-id="64fb0-192">C#</span></span>

   1. <span data-ttu-id="64fb0-193">(C#) 导航回 `GetUploadUrl` 函数，然后选择“集成”选项卡。</span><span class="sxs-lookup"><span data-stu-id="64fb0-193">(C#) Navigate back to the `GetUploadUrl` function, and then select the **Integrate** tab.</span></span>

   1. <span data-ttu-id="64fb0-194">(C#) 在“选定 HTTP 方法”下，选择“OPTIONS”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-194">(C#) Under Selected HTTP methods, select **OPTIONS**.</span></span>

      <span data-ttu-id="64fb0-195">“GET”、“POST”和“OPTIONS”应该均处于选定状态。</span><span class="sxs-lookup"><span data-stu-id="64fb0-195">**GET**, **POST**, and **OPTIONS** should all be selected.</span></span> <span data-ttu-id="64fb0-196">CORS 使用 OPTIONS 方法。对 C# 函数来说，此方法并未默认选定。</span><span class="sxs-lookup"><span data-stu-id="64fb0-196">CORS uses the OPTIONS method, which is not selected by default for C# functions.</span></span>  

   1. <span data-ttu-id="64fb0-197">(C#) 单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="64fb0-197">(C#) Click **Save**.</span></span>

1. <span data-ttu-id="64fb0-198">此时仍在门户中，导航到函数应用，选择“概览”选项卡，然后单击“重启”，确保 CORS 的更改生效。</span><span class="sxs-lookup"><span data-stu-id="64fb0-198">Still in the portal, navigate to the function app, select the **Overview** tab, and then click **Restart** to make sure that the changes for CORS take effect.</span></span>

## <a name="configure-cors-in-the-storage-account"></a><span data-ttu-id="64fb0-199">在存储帐户中配置 CORS</span><span class="sxs-lookup"><span data-stu-id="64fb0-199">Configure CORS in the Storage account</span></span>

<span data-ttu-id="64fb0-200">由于应用还对 Blob 存储进行客户端 JavaScript 调用，以便上传文件，因此还需为 CORS 配置存储帐户。</span><span class="sxs-lookup"><span data-stu-id="64fb0-200">Because the app also makes client-side JavaScript calls to Blob Storage to upload files, you also have to configure the Storage account for CORS.</span></span>

1. <span data-ttu-id="64fb0-201">运行以下命令，以便所有源将文件上传到存储帐户。</span><span class="sxs-lookup"><span data-stu-id="64fb0-201">Run the following command to allow all origins to upload files to the Storage account.</span></span>

    ```azurecli
    az storage cors add --methods OPTIONS PUT --origins '*' --exposed-headers '*' --allowed-headers '*' --services b --account-name <storage account name>
    ```


## <a name="modify-the-web-app-to-upload-images"></a><span data-ttu-id="64fb0-202">修改用于上传图像的 Web 应用</span><span class="sxs-lookup"><span data-stu-id="64fb0-202">Modify the web app to upload images</span></span>

<span data-ttu-id="64fb0-203">Web 应用从名为 **settings.js** 的文件中检索设置。</span><span class="sxs-lookup"><span data-stu-id="64fb0-203">The web app retrieves settings from a file named **settings.js**.</span></span> <span data-ttu-id="64fb0-204">在以下步骤中，请使用 Cloud Shell 创建该文件，然后将 `window.apiBaseUrl` 设置为函数应用的 URL，将 `window.blobBaseUrl` 设置为 Azure Blob 存储终结点的 URL。</span><span class="sxs-lookup"><span data-stu-id="64fb0-204">In the following steps, you create the file using Cloud Shell, then set `window.apiBaseUrl` to the URL of the Function app and `window.blobBaseUrl` to the URL of the Azure Blob Storage endpoint.</span></span>

1. <span data-ttu-id="64fb0-205">在 Cloud Shell 中，确保当前目录为 **www/dist** 文件夹。</span><span class="sxs-lookup"><span data-stu-id="64fb0-205">In the Cloud Shell, ensure that the current directory is the **www/dist** folder.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. <span data-ttu-id="64fb0-206">查询函数应用的 URL，确保将其存储在名为 **FUNCTION_APP_URL** 的 bash 变量中。</span><span class="sxs-lookup"><span data-stu-id="64fb0-206">Query the function app's URL and store it in a bash variable named **FUNCTION_APP_URL**.</span></span>

    ```azurecli
    export FUNCTION_APP_URL="https://"$(az functionapp show -n <function app name> -g first-serverless-app --query "defaultHostName" --output tsv)
    ```

    <span data-ttu-id="64fb0-207">确认变量已正确设置。</span><span class="sxs-lookup"><span data-stu-id="64fb0-207">Confirm the variable is correctly set.</span></span>

    ```azurecli
    echo $FUNCTION_APP_URL
    ```

1. <span data-ttu-id="64fb0-208">若要设置对函数应用进行的 API 调用的基 URI，请创建 **settings.js** 并添加函数应用 URL，如下所示。</span><span class="sxs-lookup"><span data-stu-id="64fb0-208">To set the base URI of API calls to your function app, create **settings.js** and add the function app URL like the following.</span></span>

    `window.apiBaseUrl = 'https://fnapp@lab.GlobalLabInstanceId.azurewebsites.net'`

    <span data-ttu-id="64fb0-209">进行更改时，可以运行以下命令，也可以使用命令行编辑器（例如 VIM）。</span><span class="sxs-lookup"><span data-stu-id="64fb0-209">You can make the change by running the following command or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.apiBaseUrl = '$FUNCTION_APP_URL'" > settings.js
    ```

    <span data-ttu-id="64fb0-210">确认文件已成功编写。</span><span class="sxs-lookup"><span data-stu-id="64fb0-210">Confirm the file was successfully written.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="64fb0-211">查询 Blob 存储基 URL 并将其存储在名为 **BLOB_BASE_URL** 的 bash 变量中。</span><span class="sxs-lookup"><span data-stu-id="64fb0-211">Query the Blob Storage base URL and store it in a bash variable named **BLOB_BASE_URL**.</span></span>

    ```azurecli
    export BLOB_BASE_URL=$(az storage account show -n <storage account name> -g first-serverless-app --query primaryEndpoints.blob -o tsv | sed 's/\/$//')
    ```

    <span data-ttu-id="64fb0-212">确认变量已正确设置。</span><span class="sxs-lookup"><span data-stu-id="64fb0-212">Confirm the variable is correctly set.</span></span>

    ```azurecli
    echo $BLOB_BASE_URL
    ```

1. <span data-ttu-id="64fb0-213">若要设置对函数应用进行的 API 调用的基 URI，请将存储 URL（如以下代码行所示）追加到 **settings.js**。</span><span class="sxs-lookup"><span data-stu-id="64fb0-213">To set the base URI of API calls to your function app, append the storage URL like the following line of code to **settings.js**.</span></span>

    `window.blobBaseUrl = 'https://mystorage.blob.core.windows.net'`

    <span data-ttu-id="64fb0-214">进行更改时，可以运行以下命令，也可以使用命令行编辑器（例如 VIM）。</span><span class="sxs-lookup"><span data-stu-id="64fb0-214">You can make the change by running the following command or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.blobBaseUrl = '$BLOB_BASE_URL'" >> settings.js
    ```

    <span data-ttu-id="64fb0-215">确认文件已成功编写，它现在包含 2 行内容。</span><span class="sxs-lookup"><span data-stu-id="64fb0-215">Confirm the file was successfully written and it now contains 2 lines.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="64fb0-216">将文件上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="64fb0-216">Upload the file to Blob storage.</span></span>

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-web-application"></a><span data-ttu-id="64fb0-217">测试 Web 应用程序</span><span class="sxs-lookup"><span data-stu-id="64fb0-217">Test the web application</span></span>

<span data-ttu-id="64fb0-218">目前，库应用程序能够将图像上传到 Blob 存储，但还不能显示图像。</span><span class="sxs-lookup"><span data-stu-id="64fb0-218">At this point, the gallery application is able to upload an image to Blob storage, but it can't display images yet.</span></span> <span data-ttu-id="64fb0-219">它会尝试调用 `GetImages` 函数，该函数尚不存在，因为是在以后的模块中创建的。</span><span class="sxs-lookup"><span data-stu-id="64fb0-219">It will try to call a `GetImages` function that doesn't exist yet because you create it in a later module.</span></span> <span data-ttu-id="64fb0-220">该调用会失败，而网页似乎卡在“正在分析...”处，但所选图像会成功上传。</span><span class="sxs-lookup"><span data-stu-id="64fb0-220">That call will fail, and the web page will appear to be stuck on "Analyzing...", but the image you select will be successfully uploaded.</span></span>

<span data-ttu-id="64fb0-221">可以在 Azure 门户中查看 **images** 容器的内容，验证图像是否已成功上传。</span><span class="sxs-lookup"><span data-stu-id="64fb0-221">You can verify an image is successfully uploaded by checking the contents of the **images** container in the Azure portal.</span></span>

1. <span data-ttu-id="64fb0-222">在浏览器窗口中浏览到应用程序。</span><span class="sxs-lookup"><span data-stu-id="64fb0-222">In a browser window, browse to the application.</span></span> <span data-ttu-id="64fb0-223">选择一个图像文件并将其上传。</span><span class="sxs-lookup"><span data-stu-id="64fb0-223">Select an image file and upload it.</span></span> <span data-ttu-id="64fb0-224">上传完成，但由于尚未添加显示图像的功能，应用不显示上传的照片。</span><span class="sxs-lookup"><span data-stu-id="64fb0-224">The upload completes, but because we haven't added the ability to display images yet, the app doesn't show the uploaded photo.</span></span> <span data-ttu-id="64fb0-225">（网页似乎卡在“正在分析图像...”处；稍后会修复该问题。）</span><span class="sxs-lookup"><span data-stu-id="64fb0-225">(The web page appears to be stuck on "Analyzing image..."; you'll fix that later.)</span></span>

1. <span data-ttu-id="64fb0-226">在 Cloud Shell 中，确认图像已上传到 **images** 容器。</span><span class="sxs-lookup"><span data-stu-id="64fb0-226">In the Cloud Shell, confirm the image was uploaded to the **images** container.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. <span data-ttu-id="64fb0-227">在转到下一教程之前，请删除 **images** 容器中的所有文件。</span><span class="sxs-lookup"><span data-stu-id="64fb0-227">Before moving on to the next tutorial, delete all files in the **images** container.</span></span>

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```


## <a name="summary"></a><span data-ttu-id="64fb0-228">摘要</span><span class="sxs-lookup"><span data-stu-id="64fb0-228">Summary</span></span>

<span data-ttu-id="64fb0-229">在本模块中，你创建了一个 Azure 函数应用，并学习了如何使用无服务器函数通过 Web 应用程序将图像上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="64fb0-229">In this module, you created an Azure Function app and learned how to use a serverless function to allow a web application to upload images to Blob storage.</span></span> <span data-ttu-id="64fb0-230">接下来，请学习如何使用 Blob 触发的无服务器函数为上传的图像创建缩略图。</span><span class="sxs-lookup"><span data-stu-id="64fb0-230">Next, you learn how to create thumbnails for the uploaded images using a Blob triggered serverless function.</span></span>
