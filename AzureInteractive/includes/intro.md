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
ms.openlocfilehash: d651fd3d03f2678d625e60f9ab1e9f59e623964f
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079123"
---
在本教程中，请部署一个简单的 Web 应用程序，以便呈现基于 HTML 的用户界面。 应用程序可以通过无服务器后端来上传图像并自动获取说明它们的标题。

![运行 Web 应用](media/functions-first-serverless-web-app/0-app-screenshot-finished.png)

下图显示了应用程序使用的 Azure 服务：

1. Blob 存储提供静态 Web 内容（HTML、CSS、JS）并存储图像。
2. Azure Functions 管理图像上传、重设大小和元数据存储。
3. Cosmos DB 存储图像元数据。
4. 逻辑应用从计算机视觉 API 获取图像标题。
5. Azure Active Directory 管理用户身份验证。

![解决方案体系结构示意图](media/functions-first-serverless-web-app/0-architecture.jpg)

本教程介绍如何执行下列操作：
> [!div class="checklist"]
> * 配置 Azure Blob 存储，以便托管静态网站和上传的图像。
> * 使用 Azure Functions 将图像上传到 Azure Blob 存储。
> * 使用 Azure Functions 重设图像大小
> * 在 Azure Cosmos DB 中存储图像元数据。
> * 使用认知服务视觉 API 自动生成图像标题。
> * 使用 Azure Active Directory 通过用户身份验证来确保 Web 应用的安全。