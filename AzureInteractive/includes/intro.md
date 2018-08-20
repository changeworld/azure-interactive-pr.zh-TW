---
title: 包含檔案
description: 包含檔案
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
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079122"
---
在本教學課程中，您會部署簡單的 Web 應用程式，來展現以 HTML 為基礎的使用者介面。 無伺服器後端可讓應用程式上傳影像，並自動取得可描述這些影像的標題。

![執行 Web 應用程式](media/functions-first-serverless-web-app/0-app-screenshot-finished.png)

下圖顯示應用程式所使用的 Azure 服務：

1. Blob 儲存體可提供靜態 Web 內容 (HTML、CSS、JS)，並儲存影像。
2. Azure Functions 可管理影像上傳、大小調整和中繼資料儲存體。
3. Cosmos DB 可儲存影像中繼資料。
4. Logic Apps 可從電腦視覺 API 取得影像標題。
5. Azure Active Directory 可管理使用者驗證。

![方案架構圖表](media/functions-first-serverless-web-app/0-architecture.jpg)

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 設定 Azure Blob 儲存體來裝載靜態網站和所上傳的影像。
> * 使用 Azure Functions 將影像上傳到 Azure Blob 儲存體。
> * 使用 Azure Functions 調整影像大小。
> * 在 Azure Cosmos DB 中儲存影像中繼資料。
> * 使用認知服務視覺 API 來自動產生影像標題。
> * 使用 Azure Active Directory 以透過對使用者進行驗證來保護 Web 應用程式。