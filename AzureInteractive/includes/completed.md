---
title: include 文件
description: include 文件
services: functions
author: ggailey777
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 06/25/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: a66fcc3a406c79fcf9881ddaaaf8330f5b373043
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47459985"
---
<span data-ttu-id="757d4-103">你已使用 Azure 服务成功地创建了一个全功能的无服务器应用程序。</span><span class="sxs-lookup"><span data-stu-id="757d4-103">You've successfully created a full-featured, serverless application using Azure services.</span></span>

## <a name="clean-up-resources"></a><span data-ttu-id="757d4-104">清理资源</span><span class="sxs-lookup"><span data-stu-id="757d4-104">Clean up resources</span></span>

<span data-ttu-id="757d4-105">使用完此应用程序以后，即可通过以下命令删除在教程中创建的所有资源：</span><span class="sxs-lookup"><span data-stu-id="757d4-105">When you are done working with this application, you can use the following command to delete all resources created during the tutorial:</span></span>

```azurecli
az group delete --name first-serverless-app
```

<span data-ttu-id="757d4-106">出现提示时请键入 `y`。</span><span class="sxs-lookup"><span data-stu-id="757d4-106">Type `y` when prompted.</span></span>  

## <a name="next-steps"></a><span data-ttu-id="757d4-107">后续步骤</span><span class="sxs-lookup"><span data-stu-id="757d4-107">Next steps</span></span>

<span data-ttu-id="757d4-108">本教程介绍了如何：</span><span class="sxs-lookup"><span data-stu-id="757d4-108">In this tutorial, you learned how to:</span></span>
> [!div class="checklist"]
> * <span data-ttu-id="757d4-109">配置 Azure Blob 存储，以便托管静态网站和上传的图像。</span><span class="sxs-lookup"><span data-stu-id="757d4-109">Configure Azure Blob storage to host a static website and uploaded images.</span></span>
> * <span data-ttu-id="757d4-110">使用 Azure Functions 将图像上传到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="757d4-110">Upload images to Azure Blob storage using Azure Functions.</span></span>
> * <span data-ttu-id="757d4-111">使用 Azure Functions 重设图像大小</span><span class="sxs-lookup"><span data-stu-id="757d4-111">Resize images using Azure Functions.</span></span>
> * <span data-ttu-id="757d4-112">在 Azure Cosmos DB 中存储图像元数据。</span><span class="sxs-lookup"><span data-stu-id="757d4-112">Store image metadata in Azure Cosmos DB.</span></span>
> * <span data-ttu-id="757d4-113">使用认知服务视觉 API 自动生成图像标题。</span><span class="sxs-lookup"><span data-stu-id="757d4-113">Use Cognitive Services Vision API to auto-generate image captions.</span></span>
> * <span data-ttu-id="757d4-114">使用 Azure Active Directory 通过用户身份验证来确保 Web 应用的安全。</span><span class="sxs-lookup"><span data-stu-id="757d4-114">Use Azure Active Directory to secure the web app by authenticating users.</span></span>

<span data-ttu-id="757d4-115">若要了解如何将更多的服务连接到 Functions，请继续学习逻辑应用教程。</span><span class="sxs-lookup"><span data-stu-id="757d4-115">To learn how to connect even more services to Functions, continue to the Logic Apps tutorial.</span></span> 

> [!div class="nextstepaction"]
> [<span data-ttu-id="757d4-116">Functions 与逻辑应用</span><span class="sxs-lookup"><span data-stu-id="757d4-116">Functions with Logic Apps</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-twitter-email)
