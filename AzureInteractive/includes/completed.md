---
title: include 文件
description: include 文件
services: functions
author: tdykstra
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 06/25/2018
ms.author: tdykstra
ms.custom: include file
ms.openlocfilehash: 1c4aaf7afd9fbc54dd34c2cca13a8b74e947c16a
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079128"
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