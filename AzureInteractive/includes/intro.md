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
ms.openlocfilehash: 0f86f2698a3a0c1e20272c335b63faf03b4b92d6
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460018"
---
<span data-ttu-id="cc956-103">在本教程中，请部署一个简单的 Web 应用程序，以便呈现基于 HTML 的用户界面。</span><span class="sxs-lookup"><span data-stu-id="cc956-103">In this tutorial, you deploy a simple web application that presents an HTML-based user interface.</span></span> <span data-ttu-id="cc956-104">应用程序可以通过无服务器后端来上传图像并自动获取说明它们的标题。</span><span class="sxs-lookup"><span data-stu-id="cc956-104">A serverless backend enables the application to upload images and automatically get captions describing them.</span></span>

![运行 Web 应用](media/functions-first-serverless-web-app/0-app-screenshot-finished.png)

<span data-ttu-id="cc956-106">下图显示了应用程序使用的 Azure 服务：</span><span class="sxs-lookup"><span data-stu-id="cc956-106">The following diagram shows the Azure services used by the application:</span></span>

1. <span data-ttu-id="cc956-107">Blob 存储提供静态 Web 内容（HTML、CSS、JS）并存储图像。</span><span class="sxs-lookup"><span data-stu-id="cc956-107">Blob Storage serves static web content (HTML, CSS, JS) and stores images.</span></span>
2. <span data-ttu-id="cc956-108">Azure Functions 管理图像上传、重设大小和元数据存储。</span><span class="sxs-lookup"><span data-stu-id="cc956-108">Azure Functions manages image uploads, resizing, and metadata storage.</span></span>
3. <span data-ttu-id="cc956-109">Cosmos DB 存储图像元数据。</span><span class="sxs-lookup"><span data-stu-id="cc956-109">Cosmos DB stores image metadata.</span></span>
4. <span data-ttu-id="cc956-110">逻辑应用从计算机视觉 API 获取图像标题。</span><span class="sxs-lookup"><span data-stu-id="cc956-110">Logic Apps gets image captions from Computer Vision API.</span></span>
5. <span data-ttu-id="cc956-111">Azure Active Directory 管理用户身份验证。</span><span class="sxs-lookup"><span data-stu-id="cc956-111">Azure Active Directory manages user authentication.</span></span>

![解决方案体系结构示意图](media/functions-first-serverless-web-app/0-architecture.jpg)

<span data-ttu-id="cc956-113">本教程介绍如何执行下列操作：</span><span class="sxs-lookup"><span data-stu-id="cc956-113">In this tutorial, you learn how to:</span></span>
> [!div class="checklist"]
> * <span data-ttu-id="cc956-114">配置 Azure Blob 存储，以便托管静态网站和上传的图像。</span><span class="sxs-lookup"><span data-stu-id="cc956-114">Configure Azure Blob storage to host a static website and uploaded images.</span></span>
> * <span data-ttu-id="cc956-115">使用 Azure Functions 将图像上传到 Azure Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="cc956-115">Upload images to Azure Blob storage using Azure Functions.</span></span>
> * <span data-ttu-id="cc956-116">使用 Azure Functions 重设图像大小</span><span class="sxs-lookup"><span data-stu-id="cc956-116">Resize images using Azure Functions.</span></span>
> * <span data-ttu-id="cc956-117">在 Azure Cosmos DB 中存储图像元数据。</span><span class="sxs-lookup"><span data-stu-id="cc956-117">Store image metadata in Azure Cosmos DB.</span></span>
> * <span data-ttu-id="cc956-118">使用认知服务视觉 API 自动生成图像标题。</span><span class="sxs-lookup"><span data-stu-id="cc956-118">Use Cognitive Services Vision API to auto-generate image captions.</span></span>
> * <span data-ttu-id="cc956-119">使用 Azure Active Directory 通过用户身份验证来确保 Web 应用的安全。</span><span class="sxs-lookup"><span data-stu-id="cc956-119">Use Azure Active Directory to secure the web app by authenticating users.</span></span>
