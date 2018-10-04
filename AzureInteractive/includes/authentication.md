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
ms.openlocfilehash: 426a7287458a48d1bda220ad1a5f067be2ce77d6
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460035"
---
<span data-ttu-id="100ec-103">可以通过 Azure 应用服务身份验证在 Azure 函数应用中获得统包式身份验证支持。</span><span class="sxs-lookup"><span data-stu-id="100ec-103">Azure App Service Authentication enables turn-key authentication support in an Azure Function app.</span></span> <span data-ttu-id="100ec-104">它与 Facebook、Twitter、Microsoft 帐户、Google 和 Azure Active Directory 无缝集成。</span><span class="sxs-lookup"><span data-stu-id="100ec-104">It integrates seamlessly with Facebook, Twitter, Microsoft Account, Google, and Azure Active Directory.</span></span> <span data-ttu-id="100ec-105">需添加应用服务身份验证来保护 Web 应用的后端 API。</span><span class="sxs-lookup"><span data-stu-id="100ec-105">You'll add App Service Authentication to protect the backend APIs of your web app.</span></span>

## <a name="enable-app-service-authentication"></a><span data-ttu-id="100ec-106">启用应用服务身份验证</span><span class="sxs-lookup"><span data-stu-id="100ec-106">Enable App Service Authentication</span></span>

1. <span data-ttu-id="100ec-107">在 Azure 门户中打开函数应用。</span><span class="sxs-lookup"><span data-stu-id="100ec-107">Open the function app in the Azure Portal.</span></span>

1. <span data-ttu-id="100ec-108">在“平台功能”下，选择“身份验证/授权”。</span><span class="sxs-lookup"><span data-stu-id="100ec-108">Under **Platform features**, select **Authentication/Authorization**.</span></span>

    ![选择“身份验证/授权”](media/functions-first-serverless-web-app/6-authorization.jpg)

1. <span data-ttu-id="100ec-110">选择以下值：</span><span class="sxs-lookup"><span data-stu-id="100ec-110">Select the following values:</span></span>
    
    | <span data-ttu-id="100ec-111">设置</span><span class="sxs-lookup"><span data-stu-id="100ec-111">Setting</span></span>      |  <span data-ttu-id="100ec-112">建议的值</span><span class="sxs-lookup"><span data-stu-id="100ec-112">Suggested value</span></span>   | <span data-ttu-id="100ec-113">Description</span><span class="sxs-lookup"><span data-stu-id="100ec-113">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="100ec-114">**应用服务身份验证**</span><span class="sxs-lookup"><span data-stu-id="100ec-114">**App Service Authentication**</span></span> | <span data-ttu-id="100ec-115">启用</span><span class="sxs-lookup"><span data-stu-id="100ec-115">On</span></span> | <span data-ttu-id="100ec-116">启用身份验证</span><span class="sxs-lookup"><span data-stu-id="100ec-116">Enable authentication.</span></span> |
    | <span data-ttu-id="100ec-117">**请求未经身份验证时的操作**</span><span class="sxs-lookup"><span data-stu-id="100ec-117">**Action when request is not authenticated**</span></span> | <span data-ttu-id="100ec-118">使用 Azure Active Directory 登录</span><span class="sxs-lookup"><span data-stu-id="100ec-118">Log in with Azure Active Directory</span></span> | <span data-ttu-id="100ec-119">选择配置的身份验证方法（见下）。</span><span class="sxs-lookup"><span data-stu-id="100ec-119">Select a configured authentication method (below).</span></span> |
    | <span data-ttu-id="100ec-120">**身份验证提供程序**</span><span class="sxs-lookup"><span data-stu-id="100ec-120">**Authentication Providers**</span></span> | <span data-ttu-id="100ec-121">请参阅下文</span><span class="sxs-lookup"><span data-stu-id="100ec-121">See below</span></span> | <span data-ttu-id="100ec-122">请参阅下文</span><span class="sxs-lookup"><span data-stu-id="100ec-122">See below</span></span> |
    | <span data-ttu-id="100ec-123">**令牌存储**</span><span class="sxs-lookup"><span data-stu-id="100ec-123">**Token store**</span></span> | <span data-ttu-id="100ec-124">启用</span><span class="sxs-lookup"><span data-stu-id="100ec-124">On</span></span> | <span data-ttu-id="100ec-125">允许应用服务存储和管理令牌。</span><span class="sxs-lookup"><span data-stu-id="100ec-125">Allow App Service to store and manage tokens.</span></span> |
    | <span data-ttu-id="100ec-126">**允许的外部重定向 URL**</span><span class="sxs-lookup"><span data-stu-id="100ec-126">**Allowed external redirect URLs**</span></span> | <span data-ttu-id="100ec-127">应用程序的 URL，例如： https://firstserverlessweb.z4.web.core.windows.net/</span><span class="sxs-lookup"><span data-stu-id="100ec-127">The URL of your application, for example: https://firstserverlessweb.z4.web.core.windows.net/</span></span> | <span data-ttu-id="100ec-128">在用户进行身份验证后，允许将应用服务重定向到的 URL。</span><span class="sxs-lookup"><span data-stu-id="100ec-128">URL(s) that App Service is allowed to redirect to after a user is authenticated.</span></span> |

1. <span data-ttu-id="100ec-129">选择“Azure Active Directory”，以便显示“Azure Active Directory 设置”。</span><span class="sxs-lookup"><span data-stu-id="100ec-129">Select **Azure Active Directory** to reveal **Azure Active Directory Settings**.</span></span>

    1. <span data-ttu-id="100ec-130">单击“快速”作为“管理模式”并填写以下信息。</span><span class="sxs-lookup"><span data-stu-id="100ec-130">Select **Express** as the **Management Mode** and fill in the following information.</span></span>
    
        | <span data-ttu-id="100ec-131">设置</span><span class="sxs-lookup"><span data-stu-id="100ec-131">Setting</span></span>      |  <span data-ttu-id="100ec-132">建议的值</span><span class="sxs-lookup"><span data-stu-id="100ec-132">Suggested value</span></span>   | <span data-ttu-id="100ec-133">Description</span><span class="sxs-lookup"><span data-stu-id="100ec-133">Description</span></span>                                        |
        | --- | --- | ---|
        | <span data-ttu-id="100ec-134">**管理模式**</span><span class="sxs-lookup"><span data-stu-id="100ec-134">**Management mode**</span></span> | <span data-ttu-id="100ec-135">快速、创建新的 AD 应用</span><span class="sxs-lookup"><span data-stu-id="100ec-135">Express, Create new AD app</span></span> | <span data-ttu-id="100ec-136">自动设置服务主体和 Azure Active Directory 身份验证。</span><span class="sxs-lookup"><span data-stu-id="100ec-136">Automatically set up a service principal and Azure Active Directory authentication.</span></span> |
        | <span data-ttu-id="100ec-137">**创建应用**</span><span class="sxs-lookup"><span data-stu-id="100ec-137">**Create app**</span></span> | <span data-ttu-id="100ec-138">my-serverless-webapp</span><span class="sxs-lookup"><span data-stu-id="100ec-138">my-serverless-webapp</span></span> | <span data-ttu-id="100ec-139">输入唯一的应用程序名称。</span><span class="sxs-lookup"><span data-stu-id="100ec-139">Enter a unique application name.</span></span> |
    
    1. <span data-ttu-id="100ec-140">单击“确定”，保存 Azure Active Directory 设置。</span><span class="sxs-lookup"><span data-stu-id="100ec-140">Click **OK** to save the Azure Active Directory settings.</span></span>

    ![身份验证/授权和 Azure Active Directory 设置](media/functions-first-serverless-web-app/6-create-aad.png)

1. <span data-ttu-id="100ec-142">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="100ec-142">Click **Save**.</span></span>


## <a name="modify-the-web-app-to-enable-authentication"></a><span data-ttu-id="100ec-143">修改 Web 应用，以便启用身份验证</span><span class="sxs-lookup"><span data-stu-id="100ec-143">Modify the web app to enable authentication</span></span>

1. <span data-ttu-id="100ec-144">在 Cloud Shell 中，确保当前目录为 **www/dist** 文件夹。</span><span class="sxs-lookup"><span data-stu-id="100ec-144">In the Cloud Shell, ensure that the current directory is the **www/dist** folder.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. <span data-ttu-id="100ec-145">若要在函数应用中启用身份验证，请将以下代码行追加到 **settings.js**。</span><span class="sxs-lookup"><span data-stu-id="100ec-145">To enable authentication in your function app, append the following line of code to **settings.js**.</span></span>

    `window.authEnabled = true`

    <span data-ttu-id="100ec-146">进行更改和查看结果时，可以使用以下命令，也可以使用命令行编辑器（例如 VIM）。</span><span class="sxs-lookup"><span data-stu-id="100ec-146">Make the change and view the result by using the following commands or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.authEnabled = true" >> settings.js
    ```

    <span data-ttu-id="100ec-147">确认已对文件进行更改。</span><span class="sxs-lookup"><span data-stu-id="100ec-147">Confirm the change was made to the file.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="100ec-148">将文件上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="100ec-148">Upload the file to Blob storage.</span></span>

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-application"></a><span data-ttu-id="100ec-149">测试应用程序</span><span class="sxs-lookup"><span data-stu-id="100ec-149">Test the application</span></span>

1. <span data-ttu-id="100ec-150">在浏览器中打开应用程序。</span><span class="sxs-lookup"><span data-stu-id="100ec-150">Open the application in a browser.</span></span> <span data-ttu-id="100ec-151">单击“登录”即可登录。</span><span class="sxs-lookup"><span data-stu-id="100ec-151">Click **Log in** and log in.</span></span>

1. <span data-ttu-id="100ec-152">选择一个图像文件并将其上传。</span><span class="sxs-lookup"><span data-stu-id="100ec-152">Select an image file and upload it.</span></span>

    ![登录页](media/functions-first-serverless-web-app/6-aad-auth.png)
    

## <a name="summary"></a><span data-ttu-id="100ec-154">摘要</span><span class="sxs-lookup"><span data-stu-id="100ec-154">Summary</span></span>

<span data-ttu-id="100ec-155">本模块介绍了如何使用 Azure 应用服务身份验证向应用程序添加身份验证。</span><span class="sxs-lookup"><span data-stu-id="100ec-155">In this module, you learned how to add authentication to the applcation using Azure App Service Authentication.</span></span>
