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
ms.openlocfilehash: f51b864cab14273c1e88dd85d22400e0e76ef770
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47459986"
---
<span data-ttu-id="234ec-103">目前，此应用程序是一个功能正常的库，允许上传和查看图像。</span><span class="sxs-lookup"><span data-stu-id="234ec-103">At this point, the application is a functional gallery that allows you to upload and view images.</span></span> <span data-ttu-id="234ec-104">本模块介绍如何使用 Microsoft 认知服务中的计算机视觉 API，以便为上传的图像生成标题并将标题与图像元数据一起保存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="234ec-104">In this module, you learn how to use the Computer Vision API from Microsoft Cognitive Services to generate captions for uploaded images and save the captions with the image metadata in Cosmos DB.</span></span>

## <a name="create-a-computer-vision-account"></a><span data-ttu-id="234ec-105">创建计算机视觉帐户</span><span class="sxs-lookup"><span data-stu-id="234ec-105">Create a Computer Vision account</span></span>

<span data-ttu-id="234ec-106">Microsoft 认知服务是一个提供给开发人员的服务集合，可以使其应用程序更为智能。</span><span class="sxs-lookup"><span data-stu-id="234ec-106">Microsoft Cognitive Services is a collection of services available to developers to make their applications more intelligent.</span></span> <span data-ttu-id="234ec-107">计算机视觉 API 是一项无服务器服务，可以使用高级算法处理图像并返回每个图像的信息。</span><span class="sxs-lookup"><span data-stu-id="234ec-107">The Computer Vision API is a serverless service that processes images using advanced algorithms and returns information about each image.</span></span> <span data-ttu-id="234ec-108">它有一个免费层，每个月最多可以提供 5000 个 API 调用。</span><span class="sxs-lookup"><span data-stu-id="234ec-108">It has a free tier that provides up to 5000 API calls per month.</span></span>

1. <span data-ttu-id="234ec-109">确保仍登录到 Cloud Shell。</span><span class="sxs-lookup"><span data-stu-id="234ec-109">Ensure you are still signed into the Cloud Shell.</span></span> <span data-ttu-id="234ec-110">否则，请选择“进入焦点模式”，以便打开 Cloud Shell 窗口。</span><span class="sxs-lookup"><span data-stu-id="234ec-110">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="234ec-111">在资源组中创建一个新的类型为 **ComputerVision** 且名称唯一的认知服务帐户。</span><span class="sxs-lookup"><span data-stu-id="234ec-111">Create a new Cognitive Services account of kind **ComputerVision** with a unique name in your resource group.</span></span> <span data-ttu-id="234ec-112">对于免费层，请使用 **F0** 作为 SKU。</span><span class="sxs-lookup"><span data-stu-id="234ec-112">For the free tier, use **F0** as the SKU.</span></span> <span data-ttu-id="234ec-113">如果已经有现有的计算机视觉帐户，则可能需要创建一个标准帐户 (S1)；不过，这可能会产生一些费用。</span><span class="sxs-lookup"><span data-stu-id="234ec-113">If you already have an existing Computer Vision account, you may need to create a Standard account (S1); however, this may incur some costs.</span></span>

    ```azurecli
    az cognitiveservices account create -g first-serverless-app -n <computer vision account name> --kind ComputerVision --sku F0 -l westcentralus
    ```


## <a name="create-function-app-settings-for-computer-vision-url-and-key"></a><span data-ttu-id="234ec-114">为计算机视觉 URL 和密钥创建函数应用设置</span><span class="sxs-lookup"><span data-stu-id="234ec-114">Create Function App settings for Computer Vision URL and key</span></span>

<span data-ttu-id="234ec-115">若要调用计算机视觉 API，需提供 URL 和密钥。</span><span class="sxs-lookup"><span data-stu-id="234ec-115">To call the Computer Vision API, a URL and key are required.</span></span>

1. <span data-ttu-id="234ec-116">获取计算机视觉 API 密钥和 URL，并将其保存到 bash 变量中。</span><span class="sxs-lookup"><span data-stu-id="234ec-116">Obtain the Computer Vision API key and URL and save them in bash variables.</span></span>

    ```azurecli
    export COMP_VISION_KEY=$(az cognitiveservices account keys list -g first-serverless-app -n <computer vision account name> --query key1 --output tsv)
    ```
    ```azurecli
    export COMP_VISION_URL=$(az cognitiveservices account show -g first-serverless-app -n <computer vision account name> --query endpoint --output tsv)
    ```

1. <span data-ttu-id="234ec-117">在函数应用中分别创建名为 **COMP_VISION_KEY** 和 **COMP_VISION_URL** 的应用设置。</span><span class="sxs-lookup"><span data-stu-id="234ec-117">Create app settings named **COMP_VISION_KEY** and **COMP_VISION_URL**, respectively, in the function app.</span></span>

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings COMP_VISION_KEY=$COMP_VISION_KEY COMP_VISION_URL=$COMP_VISION_URL -o table
    ```


## <a name="call-computer-vision-api-from-resizeimage-function"></a><span data-ttu-id="234ec-118">从 ResizeImage 函数调用计算机视觉 API</span><span class="sxs-lookup"><span data-stu-id="234ec-118">Call Computer Vision API from ResizeImage function</span></span>

<span data-ttu-id="234ec-119">在以下步骤中，请修改用于调用计算机视觉 API 的 **ResizeImage** 函数，以便对每个上传的图像进行说明并将说明保存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="234ec-119">In the following steps, you modify the **ResizeImage** function to call the Computer Vision API to describe each uploaded image and save the description in Cosmos DB.</span></span>

1. <span data-ttu-id="234ec-120">在 Azure 门户中打开函数应用。</span><span class="sxs-lookup"><span data-stu-id="234ec-120">Open your function app in the Azure Portal.</span></span>

1. <span data-ttu-id="234ec-121">**C#**</span><span class="sxs-lookup"><span data-stu-id="234ec-121">**C#**</span></span>

    1. <span data-ttu-id="234ec-122">(C#) 使用左导航栏找到 **ResizeImage** 函数并打开其代码窗口。</span><span class="sxs-lookup"><span data-stu-id="234ec-122">(C#) Using the left navigation, locate the **ResizeImage** function and open its code window.</span></span>

    1. <span data-ttu-id="234ec-123">(C#) 将代码替换为 [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx) 的内容。</span><span class="sxs-lookup"><span data-stu-id="234ec-123">(C#) Replace the code with the contents of [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx).</span></span> <span data-ttu-id="234ec-124">此代码使用 `HttpClient` 来调用计算机视觉 API 并将其结果保存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="234ec-124">This code uses `HttpClient` to call Computer Vision API and save its result in Cosmos DB.</span></span>

1. <span data-ttu-id="234ec-125">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="234ec-125">**JavaScript**</span></span>

    1. <span data-ttu-id="234ec-126">(JavaScript) 此函数需要 npm 中的 `axios` 包对计算机视觉 API 进行 HTTP 调用。</span><span class="sxs-lookup"><span data-stu-id="234ec-126">(JavaScript) This function requires the `axios` package from npm to make an HTTP call to the Computer Vision API.</span></span> <span data-ttu-id="234ec-127">若要安装 npm 包，请在左侧导航栏中单击函数应用的名称，然后单击“平台功能”。</span><span class="sxs-lookup"><span data-stu-id="234ec-127">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="234ec-128">(JavaScript) 单击“控制台”以显示控制台窗口。</span><span class="sxs-lookup"><span data-stu-id="234ec-128">(JavaScript) Click **Console** to reveal a console window.</span></span>

    1. <span data-ttu-id="234ec-129">(JavaScript) 在控制台中运行 `npm install axios` 命令。</span><span class="sxs-lookup"><span data-stu-id="234ec-129">(JavaScript) Run the command `npm install axios` in the console.</span></span> <span data-ttu-id="234ec-130">完成此操作可能需要一到两分钟。</span><span class="sxs-lookup"><span data-stu-id="234ec-130">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="234ec-131">(JavaScript) 在左侧导航栏中单击函数名称 (**ResizeImage**) 以显示该函数，然后将所有 **index.js** 替换为 [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js) 的内容。</span><span class="sxs-lookup"><span data-stu-id="234ec-131">(JavaScript) Click on the function name (**ResizeImage**) in the left navigation to reveal the function, and then replace all of **index.js** with the contents of [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js).</span></span>

1. <span data-ttu-id="234ec-132">单击代码窗口下的“日志”，展开日志面板。</span><span class="sxs-lookup"><span data-stu-id="234ec-132">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="234ec-133">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="234ec-133">Click **Save**.</span></span> <span data-ttu-id="234ec-134">查看日志面板，确保函数已成功保存且没有错误。</span><span class="sxs-lookup"><span data-stu-id="234ec-134">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-application"></a><span data-ttu-id="234ec-135">测试应用程序</span><span class="sxs-lookup"><span data-stu-id="234ec-135">Test the application</span></span>

1. <span data-ttu-id="234ec-136">在浏览器中打开应用程序。</span><span class="sxs-lookup"><span data-stu-id="234ec-136">Open the application in a browser.</span></span> <span data-ttu-id="234ec-137">选择一个图像文件并将其上传。</span><span class="sxs-lookup"><span data-stu-id="234ec-137">Select an image file, and upload it.</span></span>

1. <span data-ttu-id="234ec-138">几秒钟后，新图像的缩略图会显示在页面中。</span><span class="sxs-lookup"><span data-stu-id="234ec-138">After a few seconds, the thumbnail of the new image should appear on the page.</span></span> <span data-ttu-id="234ec-139">将鼠标悬停在图像上方即可查看计算机视觉生成的说明。</span><span class="sxs-lookup"><span data-stu-id="234ec-139">Hover over the image to see the description generated by Computer Vision.</span></span>


## <a name="summary"></a><span data-ttu-id="234ec-140">摘要</span><span class="sxs-lookup"><span data-stu-id="234ec-140">Summary</span></span>

<span data-ttu-id="234ec-141">本模块介绍了如何使用 Microsoft 认知服务计算机视觉 API 为每个上传的图像自动生成标题。</span><span class="sxs-lookup"><span data-stu-id="234ec-141">In this module, you learned how to automatically generate captions for each uploaded image using Microsoft Cognitive Services Computer Vision API.</span></span> <span data-ttu-id="234ec-142">接下来，请学习如何使用 Azure 应用服务身份验证向应用程序添加身份验证。</span><span class="sxs-lookup"><span data-stu-id="234ec-142">Next, you learn how to add authentication to the application using Azure App Service Authentication.</span></span>
