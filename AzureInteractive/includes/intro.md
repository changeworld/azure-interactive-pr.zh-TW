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
<span data-ttu-id="c889e-103">在本教學課程中，您會部署簡單的 Web 應用程式，來展現以 HTML 為基礎的使用者介面。</span><span class="sxs-lookup"><span data-stu-id="c889e-103">In this tutorial, you deploy a simple web application that presents an HTML-based user interface.</span></span> <span data-ttu-id="c889e-104">無伺服器後端可讓應用程式上傳影像，並自動取得可描述這些影像的標題。</span><span class="sxs-lookup"><span data-stu-id="c889e-104">A serverless backend enables the application to upload images and automatically get captions describing them.</span></span>

![執行 Web 應用程式](media/functions-first-serverless-web-app/0-app-screenshot-finished.png)

<span data-ttu-id="c889e-106">下圖顯示應用程式所使用的 Azure 服務：</span><span class="sxs-lookup"><span data-stu-id="c889e-106">The following diagram shows the Azure services used by the application:</span></span>

1. <span data-ttu-id="c889e-107">Blob 儲存體可提供靜態 Web 內容 (HTML、CSS、JS)，並儲存影像。</span><span class="sxs-lookup"><span data-stu-id="c889e-107">Blob Storage serves static web content (HTML, CSS, JS) and stores images.</span></span>
2. <span data-ttu-id="c889e-108">Azure Functions 可管理影像上傳、大小調整和中繼資料儲存體。</span><span class="sxs-lookup"><span data-stu-id="c889e-108">Azure Functions manages image uploads, resizing, and metadata storage.</span></span>
3. <span data-ttu-id="c889e-109">Cosmos DB 可儲存影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="c889e-109">Cosmos DB stores image metadata.</span></span>
4. <span data-ttu-id="c889e-110">Logic Apps 可從電腦視覺 API 取得影像標題。</span><span class="sxs-lookup"><span data-stu-id="c889e-110">Logic Apps gets image captions from Computer Vision API.</span></span>
5. <span data-ttu-id="c889e-111">Azure Active Directory 可管理使用者驗證。</span><span class="sxs-lookup"><span data-stu-id="c889e-111">Azure Active Directory manages user authentication.</span></span>

![方案架構圖表](media/functions-first-serverless-web-app/0-architecture.jpg)

<span data-ttu-id="c889e-113">在本教學課程中，您了解如何：</span><span class="sxs-lookup"><span data-stu-id="c889e-113">In this tutorial, you learn how to:</span></span>
> [!div class="checklist"]
> * <span data-ttu-id="c889e-114">設定 Azure Blob 儲存體來裝載靜態網站和所上傳的影像。</span><span class="sxs-lookup"><span data-stu-id="c889e-114">Configure Azure Blob storage to host a static website and uploaded images.</span></span>
> * <span data-ttu-id="c889e-115">使用 Azure Functions 將影像上傳到 Azure Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="c889e-115">Upload images to Azure Blob storage using Azure Functions.</span></span>
> * <span data-ttu-id="c889e-116">使用 Azure Functions 調整影像大小。</span><span class="sxs-lookup"><span data-stu-id="c889e-116">Resize images using Azure Functions.</span></span>
> * <span data-ttu-id="c889e-117">在 Azure Cosmos DB 中儲存影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="c889e-117">Store image metadata in Azure Cosmos DB.</span></span>
> * <span data-ttu-id="c889e-118">使用認知服務視覺 API 來自動產生影像標題。</span><span class="sxs-lookup"><span data-stu-id="c889e-118">Use Cognitive Services Vision API to auto-generate image captions.</span></span>
> * <span data-ttu-id="c889e-119">使用 Azure Active Directory 以透過對使用者進行驗證來保護 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="c889e-119">Use Azure Active Directory to secure the web app by authenticating users.</span></span>