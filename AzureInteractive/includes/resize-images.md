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
ms.openlocfilehash: 88b0ac838dfa8e097a30cc6cef591377e4a95ad8
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079061"
---
<span data-ttu-id="ca3e2-103">上一模块介绍了如何使用无服务器函数将图像从 Web 应用程序安全地上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-103">In the previous module, you saw how a serverless function can facilitate the secure uploading of images to Blob storage from a web application.</span></span> <span data-ttu-id="ca3e2-104">本模块将创建另一无服务器函数，用于监视上传的图像并根据图像创建缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-104">In this module, you create another serverless function to watch for uploaded images and create thumbnails from them.</span></span>

## <a name="create-a-blob-storage-container-for-thumbnails"></a><span data-ttu-id="ca3e2-105">为缩略图创建 Blob 存储容器</span><span class="sxs-lookup"><span data-stu-id="ca3e2-105">Create a blob storage container for thumbnails</span></span>

<span data-ttu-id="ca3e2-106">完整大小的图像存储在名为 **images** 的容器中。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-106">The full size images are stored in a container named **images**.</span></span> <span data-ttu-id="ca3e2-107">需要另一容器来存储这些图像的缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-107">You need another container to store thumbnails of those images.</span></span>

1. <span data-ttu-id="ca3e2-108">确保仍登录到 Cloud Shell (bash)。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-108">Ensure you are still signed in to the Cloud Shell (bash).</span></span>  <span data-ttu-id="ca3e2-109">否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-109">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="ca3e2-110">在可以公开访问所有 Blob 的存储帐户中创建新的名为 **thumbnails** 的容器。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-110">Create a new container named **thumbnails** in your storage account with public access to all blobs.</span></span>

    ```azurecli
    az storage container create -n thumbnails --account-name <storage account name> --public-access blob
    ```


## <a name="create-a-blob-triggered-serverless-function"></a><span data-ttu-id="ca3e2-111">创建 Blob 触发的无服务器函数</span><span class="sxs-lookup"><span data-stu-id="ca3e2-111">Create a blob-triggered serverless function</span></span>

<span data-ttu-id="ca3e2-112">触发器定义函数的调用方式。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-112">A trigger defines how a function is invoked.</span></span> <span data-ttu-id="ca3e2-113">接下来创建的函数使用 Blob 触发器：当某个 Blob（图像文件）上传到 **images** 容器时，系统会自动调用此函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-113">The function you create next uses a blob trigger: the function is automatically invoked when a blob (image file) is uploaded to the **images** container.</span></span> <span data-ttu-id="ca3e2-114">一个函数必须有一个触发器。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-114">A function must have one trigger.</span></span> <span data-ttu-id="ca3e2-115">触发器具有关联数据，该数据通常是触发函数的有效负载。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-115">Triggers have associated data, which is usually the payload that triggered the function.</span></span>

<span data-ttu-id="ca3e2-116">绑定定义函数在 Azure 或第三方服务中读取或写入数据的方式。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-116">Bindings define how a function reads or writes data in Azure or third-party services.</span></span> <span data-ttu-id="ca3e2-117">此函数创建触发它的图像的缩略图版本，并将缩略图保存在 *thumbnails* 容器中。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-117">This function creates a thumbnail version of the image that triggers it and saves the thumbnail in a *thumbnails* container.</span></span>

1. <span data-ttu-id="ca3e2-118">在 Azure 门户中打开函数应用。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-118">Open your function app in the Azure Portal.</span></span>

1. <span data-ttu-id="ca3e2-119">在函数应用窗口的左侧导航栏中，将鼠标悬停在“函数”上，然后单击“+”开始创建新的无服务器函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-119">In the function app window's left hand navigation, hover over **Functions** and click **+** to start creating a new serverless function.</span></span> <span data-ttu-id="ca3e2-120">如果显示快速入门页，则单击“自定义函数”即可看到函数模板的列表。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-120">If a quickstart page appears, click **Custom function** to see a list of function templates.</span></span>

1. <span data-ttu-id="ca3e2-121">找到 **BlobTrigger** 模板并将其选中。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-121">Find the **BlobTrigger** template and select it.</span></span>

1. <span data-ttu-id="ca3e2-122">使用这些值创建一个函数，以便在图像上传后创建缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-122">Use these values to create a function that creates thumbnails as images are uploaded.</span></span>

    | <span data-ttu-id="ca3e2-123">设置</span><span class="sxs-lookup"><span data-stu-id="ca3e2-123">Setting</span></span>      |  <span data-ttu-id="ca3e2-124">建议的值</span><span class="sxs-lookup"><span data-stu-id="ca3e2-124">Suggested value</span></span>   | <span data-ttu-id="ca3e2-125">Description</span><span class="sxs-lookup"><span data-stu-id="ca3e2-125">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="ca3e2-126">**语言**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-126">**Language**</span></span> | <span data-ttu-id="ca3e2-127">C# 或 JavaScript</span><span class="sxs-lookup"><span data-stu-id="ca3e2-127">C# or JavaScript</span></span> | <span data-ttu-id="ca3e2-128">选择首选语言。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-128">Choose your preferred language.</span></span> |
    | <span data-ttu-id="ca3e2-129">**为函数命名**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-129">**Name your function**</span></span> | <span data-ttu-id="ca3e2-130">ResizeImage</span><span class="sxs-lookup"><span data-stu-id="ca3e2-130">ResizeImage</span></span> | <span data-ttu-id="ca3e2-131">根据显示准确地键入此名称，使应用程序能够发现此函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-131">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="ca3e2-132">**路径**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-132">**Path**</span></span> | <span data-ttu-id="ca3e2-133">images/{name}</span><span class="sxs-lookup"><span data-stu-id="ca3e2-133">images/{name}</span></span> | <span data-ttu-id="ca3e2-134">当某个文件出现在 **images** 容器中时执行此函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-134">Execute the function when a file appears in the **images** container.</span></span> |
    | <span data-ttu-id="ca3e2-135">**存储帐户信息**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-135">**Storage account information**</span></span> | <span data-ttu-id="ca3e2-136">AZURE_STORAGE_CONNECTION_STRING</span><span class="sxs-lookup"><span data-stu-id="ca3e2-136">AZURE_STORAGE_CONNECTION_STRING</span></span> | <span data-ttu-id="ca3e2-137">将以前创建的环境变量与此连接字符串配合使用。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-137">Use the environment variable previously created with the connection string.</span></span> |

    ![输入新函数的设置](media/functions-first-serverless-web-app/3-new-function.png)

1. <span data-ttu-id="ca3e2-139">单击“创建”创建该函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-139">Click **Create** to create the function.</span></span>

1. <span data-ttu-id="ca3e2-140">创建函数后，单击“集成”即可查看其触发器、输入和输出绑定。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-140">When the function is created, click **Integrate** to view its trigger, input, and output bindings.</span></span>

1. <span data-ttu-id="ca3e2-141">单击“新建输出”创建新的输出绑定。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-141">Click **New output** to create a new output binding.</span></span>

    ![在“集成”选项卡上选择“新建输出”](media/functions-first-serverless-web-app/3-new-output.jpg)

1. <span data-ttu-id="ca3e2-143">选择“Azure Blob 存储”，然后单击“选择”。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-143">Select **Azure Blob Storage** and click **Select**.</span></span> <span data-ttu-id="ca3e2-144">可能需要向下滚动才能看到“选择”按钮。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-144">You may have to scroll down to see the **Select** button.</span></span>

    ![选择“Azure Blob 存储”](media/functions-first-serverless-web-app/3-storage-output.jpg)

1. <span data-ttu-id="ca3e2-146">输入以下值。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-146">Enter the following values.</span></span>

    | <span data-ttu-id="ca3e2-147">设置</span><span class="sxs-lookup"><span data-stu-id="ca3e2-147">Setting</span></span>      |  <span data-ttu-id="ca3e2-148">建议的值</span><span class="sxs-lookup"><span data-stu-id="ca3e2-148">Suggested value</span></span>   | <span data-ttu-id="ca3e2-149">Description</span><span class="sxs-lookup"><span data-stu-id="ca3e2-149">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="ca3e2-150">**Blob 参数名称**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-150">**Blob parameter name**</span></span> | <span data-ttu-id="ca3e2-151">thumbnail</span><span class="sxs-lookup"><span data-stu-id="ca3e2-151">thumbnail</span></span> | <span data-ttu-id="ca3e2-152">函数将参数与此名称配合使用，以便写入缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-152">The function uses the parameter with this name to write the thumbnail.</span></span> |
    | <span data-ttu-id="ca3e2-153">**使用函数返回值**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-153">**Use function return value**</span></span> | <span data-ttu-id="ca3e2-154">否</span><span class="sxs-lookup"><span data-stu-id="ca3e2-154">No</span></span> |  |
    | <span data-ttu-id="ca3e2-155">**路径**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-155">**Path**</span></span> | <span data-ttu-id="ca3e2-156">thumbnails/{name}</span><span class="sxs-lookup"><span data-stu-id="ca3e2-156">thumbnails/{name}</span></span> | <span data-ttu-id="ca3e2-157">缩略图会写入一个名为 **thumbnails** 的容器中。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-157">The thumbnails are written to a container named **thumbnails**.</span></span> |
    | <span data-ttu-id="ca3e2-158">**存储帐户连接**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-158">**Storage account connection**</span></span> | <span data-ttu-id="ca3e2-159">AZURE_STORAGE_CONNECTION_STRING</span><span class="sxs-lookup"><span data-stu-id="ca3e2-159">AZURE_STORAGE_CONNECTION_STRING</span></span> | <span data-ttu-id="ca3e2-160">将以前创建的环境变量与此连接字符串配合使用。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-160">Use the environment variable previously created with the connection string.</span></span> |

    ![输入 Blob 输出绑定的设置](media/functions-first-serverless-web-app/3-blob-output.png)

1. <span data-ttu-id="ca3e2-162">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-162">**JavaScript**</span></span>

    1. <span data-ttu-id="ca3e2-163">(JavaScript) 单击窗口右上角的“高级编辑器”以显示表示绑定的 JSON。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-163">(JavaScript) Click on **Advanced editor** in the top right corner of the window to reveal the JSON representing the bindings.</span></span>

    1. <span data-ttu-id="ca3e2-164">(JavaScript) 在 `blobTrigger` 绑定中，添加名称为 `dataType` 且值为 `binary` 的属性。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-164">(JavaScript) In the `blobTrigger` binding, add a property named `dataType` with a value of `binary`.</span></span> <span data-ttu-id="ca3e2-165">这样就可以配置绑定，以便将 Blob 内容以二进制数据的形式传递给函数。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-165">This configures the binding to pass the blob contents to the function as binary data.</span></span>

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

1. <span data-ttu-id="ca3e2-166">单击“保存”创建新绑定。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-166">Click **Save** to create the new binding.</span></span>

1. <span data-ttu-id="ca3e2-167">**C#**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-167">**C#**</span></span>

    1. <span data-ttu-id="ca3e2-168">(C#) 在左导航栏中选择 **ResizeImage** 函数名称，打开函数的源代码。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-168">(C#) Select the **ResizeImage** function name on the left navigation to open the function's source code.</span></span>

    1. <span data-ttu-id="ca3e2-169">(C#) 此函数需要名为 **ImageResizer** 的 NuGet 包来生成缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-169">(C#) The function requires a NuGet package called **ImageResizer** to generate the thumbnails.</span></span> <span data-ttu-id="ca3e2-170">NuGet 包是使用 **project.json** 文件添加到 C# 函数的。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-170">NuGet packages are added to C# functions using a **project.json** file.</span></span> <span data-ttu-id="ca3e2-171">若要创建此文件，请单击右侧的“查看文件”以显示构成函数的文件。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-171">To create the file, click **View Files** on the right to reveal the files that make up the function.</span></span>
    
    1. <span data-ttu-id="ca3e2-172">(C#) 单击“添加”，添加名为 **project.json** 的新文件。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-172">(C#) Click **Add** to add a new file named **project.json**.</span></span>
    
    1. <span data-ttu-id="ca3e2-173">(C#) 将 [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) 的内容复制到新创建的文件中。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-173">(C#) Copy the contents of [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) into the newly created file.</span></span> <span data-ttu-id="ca3e2-174">保存文件。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-174">Save the file.</span></span> <span data-ttu-id="ca3e2-175">文件更新后，包会自动还原。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-175">Packages are automatically restored when the file is updated.</span></span>
    
        ![包含 ImageResizer 的 project.json 文件](media/functions-first-serverless-web-app/3-project-json.png)
    
    1. <span data-ttu-id="ca3e2-177">(C#) 在“查看文件”下选择 **run.csx**，将其内容替换为 [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx) 中的内容。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-177">(C#) Select **run.csx** under **View Files** and replace its content with the content in [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx).</span></span>

1. <span data-ttu-id="ca3e2-178">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="ca3e2-178">**JavaScript**</span></span> 

    1. <span data-ttu-id="ca3e2-179">(JavaScript) 此函数需要 npm 中的 `jimp` 包来重设照片大小。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-179">(JavaScript) This function requires the `jimp` package from npm to resize the photo.</span></span> <span data-ttu-id="ca3e2-180">若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-180">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="ca3e2-181">(JavaScript) 单击“控制台”以显示控制台窗口。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-181">(JavaScript) Click **Console** to reveal a console window.</span></span>

    1. <span data-ttu-id="ca3e2-182">(JavaScript) 在控制台中运行 `npm install jimp` 命令。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-182">(JavaScript) Run the command `npm install jimp` in the console.</span></span> <span data-ttu-id="ca3e2-183">完成此操作可能需要一到两分钟。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-183">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="ca3e2-184">(JavaScript) 在左侧导航栏中单击函数名称 **ResizeImage** 以显示该函数，将所有 **index.js** 替换为 [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js) 的内容。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-184">(JavaScript) Click on the **ResizeImage** function name in the left navigation to reveal the function, replace all of **index.js** with the content of [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js).</span></span>

1. <span data-ttu-id="ca3e2-185">单击代码窗口下的“日志”，展开日志面板。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-185">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="ca3e2-186">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-186">Click **Save**.</span></span> <span data-ttu-id="ca3e2-187">查看日志面板，确保函数已成功保存且没有错误。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-187">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-serverless-function"></a><span data-ttu-id="ca3e2-188">测试无服务器函数</span><span class="sxs-lookup"><span data-stu-id="ca3e2-188">Test the serverless function</span></span>

1. <span data-ttu-id="ca3e2-189">在浏览器中打开应用程序。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-189">Open the application in a browser.</span></span> <span data-ttu-id="ca3e2-190">选择一个图像文件并将其上传。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-190">Select an image file and upload it.</span></span> <span data-ttu-id="ca3e2-191">上传完成，但由于尚未添加显示图像的功能，应用不显示上传的照片。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-191">The upload completes, but because we haven't added the ability to display images yet, the app doesn't show the uploaded photo.</span></span>

1. <span data-ttu-id="ca3e2-192">在 Cloud Shell 中，确认图像已上传到 **images** 容器。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-192">In the Cloud Shell, confirm the image was uploaded to the **images** container.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. <span data-ttu-id="ca3e2-193">确认缩略图已在名为 **thumbnails** 的容器中创建。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-193">Confirm the thumbnail was created in a container named **thumbnails**.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c thumbnails -o table
    ```

1. <span data-ttu-id="ca3e2-194">获取缩略图的 URL。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-194">Obtain the thumbnail's URL.</span></span>

    ```azurecli
    az storage blob url --account-name <storage account name> -c thumbnails -n <filename> --output tsv
    ```

    <span data-ttu-id="ca3e2-195">在浏览器中打开该 URL，确认缩略图已正确创建。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-195">Open the URL in a browser and confirm the thumbnail was properly created.</span></span>

1. <span data-ttu-id="ca3e2-196">在转到下一教程之前，请删除 **images** 和 **thumbnails** 容器中的所有文件。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-196">Before moving on to the next tutorial, delete all files in the **images** and **thumbnails** containers.</span></span>

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```
    ```azurecli
    az storage blob delete-batch -s thumbnails --account-name <storage account name>
    ```


## <a name="summary"></a><span data-ttu-id="ca3e2-197">摘要</span><span class="sxs-lookup"><span data-stu-id="ca3e2-197">Summary</span></span>

<span data-ttu-id="ca3e2-198">在本模块中，你创建了一个无服务器函数。只要有图像上传到 Blob 存储容器，该函数就会创建缩略图。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-198">In this module, you created a serverless function to create a thumbnail whenever an image is uploaded to a Blob storage container.</span></span> <span data-ttu-id="ca3e2-199">接下来，请学习如何使用 Azure Cosmos DB 来存储和列出图像元数据。</span><span class="sxs-lookup"><span data-stu-id="ca3e2-199">Next, you learn how to use Azure Cosmos DB to store and list image metadata.</span></span>
