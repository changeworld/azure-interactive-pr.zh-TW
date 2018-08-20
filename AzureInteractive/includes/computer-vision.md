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
ms.openlocfilehash: 7e51d3cd0533b4fb64d7dfa783af55266d536f54
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079134"
---
<span data-ttu-id="14ee2-103">到目前為止，此應用程式已是一個可運作的影像中心，可讓您上傳和檢視影像。</span><span class="sxs-lookup"><span data-stu-id="14ee2-103">At this point, the application is a functional gallery that allows you to upload and view images.</span></span> <span data-ttu-id="14ee2-104">在本單元中，您會了解如何使用 Microsoft 認知服務的電腦視覺 API，為所上傳的影像產生標題，並使用影像中繼資料將標題儲存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="14ee2-104">In this module, you learn how to use the Computer Vision API from Microsoft Cognitive Services to generate captions for uploaded images and save the captions with the image metadata in Cosmos DB.</span></span>

## <a name="create-a-computer-vision-account"></a><span data-ttu-id="14ee2-105">建立電腦視覺帳戶</span><span class="sxs-lookup"><span data-stu-id="14ee2-105">Create a Computer Vision account</span></span>

<span data-ttu-id="14ee2-106">Microsoft 認知服務是一組可供開發人員使用的服務，可讓其應用程式更有智慧。</span><span class="sxs-lookup"><span data-stu-id="14ee2-106">Microsoft Cognitive Services is a collection of services available to developers to make their applications more intelligent.</span></span> <span data-ttu-id="14ee2-107">電腦視覺 API 是一種無伺服器服務，會使用進階演算法處理影像，並傳回每個影像的相關資訊。</span><span class="sxs-lookup"><span data-stu-id="14ee2-107">The Computer Vision API is a serverless service that processes images using advanced algorithms and returns information about each image.</span></span> <span data-ttu-id="14ee2-108">它有免費層可提供每月最多 5000 個 API 呼叫。</span><span class="sxs-lookup"><span data-stu-id="14ee2-108">It has a free tier that provides up to 5000 API calls per month.</span></span>

1. <span data-ttu-id="14ee2-109">確定您仍登入 Cloud Shell。</span><span class="sxs-lookup"><span data-stu-id="14ee2-109">Ensure you are still signed into the Cloud Shell.</span></span> <span data-ttu-id="14ee2-110">如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。</span><span class="sxs-lookup"><span data-stu-id="14ee2-110">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="14ee2-111">使用唯一名稱在資源群組中建立 **ComputerVision** 種類的新認知服務帳戶。</span><span class="sxs-lookup"><span data-stu-id="14ee2-111">Create a new Cognitive Services account of kind **ComputerVision** with a unique name in your resource group.</span></span> <span data-ttu-id="14ee2-112">在免費層中，請使用 **F0** 作為 SKU。</span><span class="sxs-lookup"><span data-stu-id="14ee2-112">For the free tier, use **F0** as the SKU.</span></span> <span data-ttu-id="14ee2-113">如果您已有既有的電腦視覺帳戶，則可能需要建立標準帳戶 (S1)；不過，這麼做可能會產生一些成本。</span><span class="sxs-lookup"><span data-stu-id="14ee2-113">If you already have an existing Computer Vision account, you may need to create a Standard account (S1); however, this may incur some costs.</span></span>

    ```azurecli
    az cognitiveservices account create -g first-serverless-app -n <computer vision account name> --kind ComputerVision --sku F0 -l westcentralus
    ```


## <a name="create-function-app-settings-for-computer-vision-url-and-key"></a><span data-ttu-id="14ee2-114">建立函式應用程式設定以取得電腦視覺 URL 和金鑰</span><span class="sxs-lookup"><span data-stu-id="14ee2-114">Create Function App settings for Computer Vision URL and key</span></span>

<span data-ttu-id="14ee2-115">若要呼叫電腦視覺 API，您需要有 URL 和金鑰。</span><span class="sxs-lookup"><span data-stu-id="14ee2-115">To call the Computer Vision API, a URL and key are required.</span></span>

1. <span data-ttu-id="14ee2-116">請取得電腦視覺 API 金鑰和 URL，並將其儲存在 Bash 變數中。</span><span class="sxs-lookup"><span data-stu-id="14ee2-116">Obtain the Computer Vision API key and URL and save them in bash variables.</span></span>

    ```azurecli
    export COMP_VISION_KEY=$(az cognitiveservices account keys list -g first-serverless-app -n <computer vision account name> --query key1 --output tsv)
    ```
    ```azurecli
    export COMP_VISION_URL=$(az cognitiveservices account show -g first-serverless-app -n <computer vision account name> --query endpoint --output tsv)
    ```

1. <span data-ttu-id="14ee2-117">在函式應用程式中分別建立名為 **COMP_VISION_KEY** 和 **COMP_VISION_URL** 的應用程式設定。</span><span class="sxs-lookup"><span data-stu-id="14ee2-117">Create app settings named **COMP_VISION_KEY** and **COMP_VISION_URL**, respectively, in the function app.</span></span>

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings COMP_VISION_KEY=$COMP_VISION_KEY COMP_VISION_URL=$COMP_VISION_URL -o table
    ```


## <a name="call-computer-vision-api-from-resizeimage-function"></a><span data-ttu-id="14ee2-118">從 ResizeImage 函式呼叫電腦視覺 API</span><span class="sxs-lookup"><span data-stu-id="14ee2-118">Call Computer Vision API from ResizeImage function</span></span>

<span data-ttu-id="14ee2-119">在下列步驟中，您會修改 **ResizeImage** 函式來呼叫電腦視覺 API，以描述每個所上傳的影像，並將描述儲存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="14ee2-119">In the following steps, you modify the **ResizeImage** function to call the Computer Vision API to describe each uploaded image and save the description in Cosmos DB.</span></span>

1. <span data-ttu-id="14ee2-120">在 Azure 入口網站中開啟函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="14ee2-120">Open your function app in the Azure Portal.</span></span>

1. <span data-ttu-id="14ee2-121">**C#**</span><span class="sxs-lookup"><span data-stu-id="14ee2-121">**C#**</span></span>

    1. <span data-ttu-id="14ee2-122">(C#) 使用左側導覽找出 **ResizeImage** 函式，並開啟其程式碼視窗。</span><span class="sxs-lookup"><span data-stu-id="14ee2-122">(C#) Using the left navigation, locate the **ResizeImage** function and open its code window.</span></span>

    1. <span data-ttu-id="14ee2-123">(C#) 將程式碼更換為 [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx) 的內容。</span><span class="sxs-lookup"><span data-stu-id="14ee2-123">(C#) Replace the code with the contents of [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx).</span></span> <span data-ttu-id="14ee2-124">此程式碼使用 `HttpClient` 來呼叫電腦視覺 API，並會將其結果儲存在 Cosmos DB 中。</span><span class="sxs-lookup"><span data-stu-id="14ee2-124">This code uses `HttpClient` to call Computer Vision API and save its result in Cosmos DB.</span></span>

1. <span data-ttu-id="14ee2-125">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="14ee2-125">**JavaScript**</span></span>

    1. <span data-ttu-id="14ee2-126">(JavaScript) 此函式需要 npm 的 `axios` 套件，才能對電腦視覺 API 發出 HTTP 呼叫。</span><span class="sxs-lookup"><span data-stu-id="14ee2-126">(JavaScript) This function requires the `axios` package from npm to make an HTTP call to the Computer Vision API.</span></span> <span data-ttu-id="14ee2-127">若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。</span><span class="sxs-lookup"><span data-stu-id="14ee2-127">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="14ee2-128">(JavaScript) 按一下 [主控台] 以顯示主控台視窗。</span><span class="sxs-lookup"><span data-stu-id="14ee2-128">(JavaScript) Click **Console** to reveal a console window.</span></span>

    1. <span data-ttu-id="14ee2-129">(JavaScript) 在主控台中執行 `npm install axios` 命令。</span><span class="sxs-lookup"><span data-stu-id="14ee2-129">(JavaScript) Run the command `npm install axios` in the console.</span></span> <span data-ttu-id="14ee2-130">此作業可能需要一兩分鐘的時間才能完成。</span><span class="sxs-lookup"><span data-stu-id="14ee2-130">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="14ee2-131">(JavaScript) 在左側導覽中按一下函式名稱 (**ResizeImage**) 來顯示函式，然後將 **index.js** 全部更換為 [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js) 的內容。</span><span class="sxs-lookup"><span data-stu-id="14ee2-131">(JavaScript) Click on the function name (**ResizeImage**) in the left navigation to reveal the function, and then replace all of **index.js** with the contents of [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js).</span></span>

1. <span data-ttu-id="14ee2-132">按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。</span><span class="sxs-lookup"><span data-stu-id="14ee2-132">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="14ee2-133">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="14ee2-133">Click **Save**.</span></span> <span data-ttu-id="14ee2-134">檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。</span><span class="sxs-lookup"><span data-stu-id="14ee2-134">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-application"></a><span data-ttu-id="14ee2-135">測試應用程式</span><span class="sxs-lookup"><span data-stu-id="14ee2-135">Test the application</span></span>

1. <span data-ttu-id="14ee2-136">在瀏覽器中開啟應用程式。</span><span class="sxs-lookup"><span data-stu-id="14ee2-136">Open the application in a browser.</span></span> <span data-ttu-id="14ee2-137">選取影像檔，並將它上傳。</span><span class="sxs-lookup"><span data-stu-id="14ee2-137">Select an image file, and upload it.</span></span>

1. <span data-ttu-id="14ee2-138">幾秒後，新影像的縮圖應該就會出現在頁面上。</span><span class="sxs-lookup"><span data-stu-id="14ee2-138">After a few seconds, the thumbnail of the new image should appear on the page.</span></span> <span data-ttu-id="14ee2-139">將滑鼠停留在影像上，以查看電腦願景所產生的描述。</span><span class="sxs-lookup"><span data-stu-id="14ee2-139">Hover over the image to see the description generated by Computer Vision.</span></span>


## <a name="summary"></a><span data-ttu-id="14ee2-140">總結</span><span class="sxs-lookup"><span data-stu-id="14ee2-140">Summary</span></span>

<span data-ttu-id="14ee2-141">在本單元中，您已了解如何使用 Microsoft 認知服務的電腦視覺 API，為每個已上傳的影像自動產生標題。</span><span class="sxs-lookup"><span data-stu-id="14ee2-141">In this module, you learned how to automatically generate captions for each uploaded image using Microsoft Cognitive Services Computer Vision API.</span></span> <span data-ttu-id="14ee2-142">接下來，您將會了解如何使用 Azure App Service 驗證，對應用程式新增驗證。</span><span class="sxs-lookup"><span data-stu-id="14ee2-142">Next, you learn how to add authentication to the application using Azure App Service Authentication.</span></span>