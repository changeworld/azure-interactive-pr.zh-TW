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
<span data-ttu-id="fb919-103">您已成功建立使用 Azure 服務的全功能無伺服器應用程式。</span><span class="sxs-lookup"><span data-stu-id="fb919-103">You've successfully created a full-featured, serverless application using Azure services.</span></span>

## <a name="clean-up-resources"></a><span data-ttu-id="fb919-104">清除資源</span><span class="sxs-lookup"><span data-stu-id="fb919-104">Clean up resources</span></span>

<span data-ttu-id="fb919-105">此應用程式使用完畢時，您可以使用下列命令來刪除教學課程進行期間所建立的所有資源：</span><span class="sxs-lookup"><span data-stu-id="fb919-105">When you are done working with this application, you can use the following command to delete all resources created during the tutorial:</span></span>

```azurecli
az group delete --name first-serverless-app
```

<span data-ttu-id="fb919-106">在出現提示時輸入 `y`。</span><span class="sxs-lookup"><span data-stu-id="fb919-106">Type `y` when prompted.</span></span>  

## <a name="next-steps"></a><span data-ttu-id="fb919-107">後續步驟</span><span class="sxs-lookup"><span data-stu-id="fb919-107">Next steps</span></span>

<span data-ttu-id="fb919-108">在本教學課程中，您已了解如何：</span><span class="sxs-lookup"><span data-stu-id="fb919-108">In this tutorial, you learned how to:</span></span>
> [!div class="checklist"]
> * <span data-ttu-id="fb919-109">設定 Azure Blob 儲存體來裝載靜態網站和所上傳的影像。</span><span class="sxs-lookup"><span data-stu-id="fb919-109">Configure Azure Blob storage to host a static website and uploaded images.</span></span>
> * <span data-ttu-id="fb919-110">使用 Azure Functions 將影像上傳到 Azure Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="fb919-110">Upload images to Azure Blob storage using Azure Functions.</span></span>
> * <span data-ttu-id="fb919-111">使用 Azure Functions 調整影像大小。</span><span class="sxs-lookup"><span data-stu-id="fb919-111">Resize images using Azure Functions.</span></span>
> * <span data-ttu-id="fb919-112">在 Azure Cosmos DB 中儲存影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="fb919-112">Store image metadata in Azure Cosmos DB.</span></span>
> * <span data-ttu-id="fb919-113">使用認知服務視覺 API 來自動產生影像標題。</span><span class="sxs-lookup"><span data-stu-id="fb919-113">Use Cognitive Services Vision API to auto-generate image captions.</span></span>
> * <span data-ttu-id="fb919-114">使用 Azure Active Directory 以透過對使用者進行驗證來保護 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="fb919-114">Use Azure Active Directory to secure the web app by authenticating users.</span></span>

<span data-ttu-id="fb919-115">若要了解如何將更多的服務連線至 Functions，請繼續閱讀 Logic Apps 教學課程。</span><span class="sxs-lookup"><span data-stu-id="fb919-115">To learn how to connect even more services to Functions, continue to the Logic Apps tutorial.</span></span> 

> [!div class="nextstepaction"]
> [<span data-ttu-id="fb919-116">搭配使用 Functions 與 Logic Apps</span><span class="sxs-lookup"><span data-stu-id="fb919-116">Functions with Logic Apps</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-twitter-email)
