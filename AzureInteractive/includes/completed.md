---
title: 包含檔案
description: 包含檔案
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
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47459987"
---
您已成功建立使用 Azure 服務的全功能無伺服器應用程式。

## <a name="clean-up-resources"></a>清除資源

此應用程式使用完畢時，您可以使用下列命令來刪除教學課程進行期間所建立的所有資源：

```azurecli
az group delete --name first-serverless-app
```

在出現提示時輸入 `y`。  

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：
> [!div class="checklist"]
> * 設定 Azure Blob 儲存體來裝載靜態網站和所上傳的影像。
> * 使用 Azure Functions 將影像上傳到 Azure Blob 儲存體。
> * 使用 Azure Functions 調整影像大小。
> * 在 Azure Cosmos DB 中儲存影像中繼資料。
> * 使用認知服務視覺 API 來自動產生影像標題。
> * 使用 Azure Active Directory 以透過對使用者進行驗證來保護 Web 應用程式。

若要了解如何將更多的服務連線至 Functions，請繼續閱讀 Logic Apps 教學課程。 

> [!div class="nextstepaction"]
> [搭配使用 Functions 與 Logic Apps](https://docs.microsoft.com/azure/azure-functions/functions-twitter-email)
