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
你已使用 Azure 服务成功地创建了一个全功能的无服务器应用程序。

## <a name="clean-up-resources"></a>清理资源

使用完此应用程序以后，即可通过以下命令删除在教程中创建的所有资源：

```azurecli
az group delete --name first-serverless-app
```

出现提示时请键入 `y`。  

## <a name="next-steps"></a>后续步骤

本教程介绍了如何：
> [!div class="checklist"]
> * 配置 Azure Blob 存储，以便托管静态网站和上传的图像。
> * 使用 Azure Functions 将图像上传到 Azure Blob 存储。
> * 使用 Azure Functions 重设图像大小
> * 在 Azure Cosmos DB 中存储图像元数据。
> * 使用认知服务视觉 API 自动生成图像标题。
> * 使用 Azure Active Directory 通过用户身份验证来确保 Web 应用的安全。

若要了解如何将更多的服务连接到 Functions，请继续学习逻辑应用教程。 

> [!div class="nextstepaction"]
> [Functions 与逻辑应用](https://docs.microsoft.com/azure/azure-functions/functions-twitter-email)
