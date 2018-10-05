---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 06/21/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 194a25dbf9abda80379aa5aab408ac4ffe9ab7f5
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460053"
---
<span data-ttu-id="77b78-103">Azure Cosmos DB 是 Microsoft 全球發行的多模型無伺服器資料庫。</span><span class="sxs-lookup"><span data-stu-id="77b78-103">Azure Cosmos DB is Microsoft's serverless, globally distributed, multi-model database.</span></span> <span data-ttu-id="77b78-104">在本單元中，您會了解如何使用 Azure Functions，在 Cosmos DB 中儲存和擷取 JSON 文件形式的影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="77b78-104">In this module, you learn how to use Azure Functions to store and retrieve image metadata as JSON documents in Cosmos DB.</span></span>

## <a name="create-a-cosmos-db-account-database-and-collection"></a><span data-ttu-id="77b78-105">建立 Cosmos DB 帳戶、資料庫和集合</span><span class="sxs-lookup"><span data-stu-id="77b78-105">Create a Cosmos DB account, database, and collection</span></span>

<span data-ttu-id="77b78-106">Cosmos DB 帳戶是 Azure 資源，內含 Cosmos DB 資料庫。</span><span class="sxs-lookup"><span data-stu-id="77b78-106">A Cosmos DB account is an Azure resource that contains Cosmos DB databases.</span></span>

1. <span data-ttu-id="77b78-107">確定您仍登入 Cloud Shell。</span><span class="sxs-lookup"><span data-stu-id="77b78-107">Ensure you are still signed into the Cloud Shell.</span></span>  <span data-ttu-id="77b78-108">如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。</span><span class="sxs-lookup"><span data-stu-id="77b78-108">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="77b78-109">在與本教學課程中其他資源相同的資源群組內，使用唯一名稱建立 Cosmos DB 帳戶。</span><span class="sxs-lookup"><span data-stu-id="77b78-109">Create a Cosmos DB account with a unique name in the same resource group as the other resources in this tutorial.</span></span>

    ```azurecli
    az cosmosdb create -g first-serverless-app -n <cosmos db account name>
    ```

1. <span data-ttu-id="77b78-110">建立 Cosmos DB 帳戶之後，於該帳戶內建立名為 **imagesdb** 的新資料庫。</span><span class="sxs-lookup"><span data-stu-id="77b78-110">After the Cosmos DB account is created, create a new database named **imagesdb** in the account.</span></span>

    ```azurecli
    az cosmosdb database create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb
    ```

1. <span data-ttu-id="77b78-111">建立資料庫之後，於資料庫內建立名為 **images**，且輸送量為 400 個要求單位 (RU) 的新集合。</span><span class="sxs-lookup"><span data-stu-id="77b78-111">After the database is created, create a new collection named **images** in the database with a throughput of 400 request units (RUs).</span></span>

    ```azurecli
    az cosmosdb collection create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb --collection-name images --throughput 400
    ```


## <a name="save-a-document-to-cosmos-db-when-a-thumbnail-is-created"></a><span data-ttu-id="77b78-112">在建立縮圖時將文件儲存至 Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="77b78-112">Save a document to Cosmos DB when a thumbnail is created</span></span>

<span data-ttu-id="77b78-113">Cosmos DB 輸出繫結可讓您從 Azure Functions 在 Cosmos DB 集合中建立文件。</span><span class="sxs-lookup"><span data-stu-id="77b78-113">The Cosmos DB output binding lets you create documents in a Cosmos DB collection from Azure Functions.</span></span> <span data-ttu-id="77b78-114">在下列步驟中，您會在 [ResizeImage] 函式中設定 Cosmos DB 輸出繫結，並修改函式以傳回要儲存的文件 (物件)。</span><span class="sxs-lookup"><span data-stu-id="77b78-114">In the following steps, you configure a Cosmos DB output binding in the **ResizeImage** function and modify the function to return a document (object) to be saved.</span></span>

1. <span data-ttu-id="77b78-115">在 Azure 入口網站中開啟函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="77b78-115">Open the function app in the Azure Portal.</span></span>

1. <span data-ttu-id="77b78-116">在左側導覽中，展開 [ResizeImage] 函式，然後選取 [整合]。</span><span class="sxs-lookup"><span data-stu-id="77b78-116">In the left hand navigation, expand the **ResizeImage** function, and then select **Integrate**.</span></span>

1. <span data-ttu-id="77b78-117">在 [輸出] 下，按一下 [新增輸出]。</span><span class="sxs-lookup"><span data-stu-id="77b78-117">Under **Outputs**, click **New Output**.</span></span>

1. <span data-ttu-id="77b78-118">尋找 [Azure Cosmos DB] 項目，然後加以選取。</span><span class="sxs-lookup"><span data-stu-id="77b78-118">Find the **Azure Cosmos DB** item and select it.</span></span> <span data-ttu-id="77b78-119">然後按一下 [選取] 。</span><span class="sxs-lookup"><span data-stu-id="77b78-119">Then click **Select**.</span></span>

    ![選取 [新增輸出]](media/functions-first-serverless-web-app/4-new-output.jpg)

1. <span data-ttu-id="77b78-121">使用下列值填寫 [Azure Cosmos DB 輸出] 下方的欄位。</span><span class="sxs-lookup"><span data-stu-id="77b78-121">Fill out the fields under **Azure Cosmos DB output** with the following values.</span></span>

    | <span data-ttu-id="77b78-122">設定</span><span class="sxs-lookup"><span data-stu-id="77b78-122">Setting</span></span>      |  <span data-ttu-id="77b78-123">建議的值</span><span class="sxs-lookup"><span data-stu-id="77b78-123">Suggested value</span></span>   | <span data-ttu-id="77b78-124">說明</span><span class="sxs-lookup"><span data-stu-id="77b78-124">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="77b78-125">**文件參數名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-125">**Document parameter name**</span></span> | <span data-ttu-id="77b78-126">選取 [使用函式傳回值]</span><span class="sxs-lookup"><span data-stu-id="77b78-126">Select **Use function return value**</span></span> | <span data-ttu-id="77b78-127">文字方塊的值會自動設定為 [$return]。</span><span class="sxs-lookup"><span data-stu-id="77b78-127">The value of the textbox is automatically set to **$return**.</span></span> |
    | <span data-ttu-id="77b78-128">**資料庫名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-128">**Database name**</span></span> | <span data-ttu-id="77b78-129">imagesdb</span><span class="sxs-lookup"><span data-stu-id="77b78-129">imagesdb</span></span> | <span data-ttu-id="77b78-130">使用所建立資料庫的名稱。</span><span class="sxs-lookup"><span data-stu-id="77b78-130">Use the name of the database that you created.</span></span> |
    | <span data-ttu-id="77b78-131">**集合名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-131">**Collection name**</span></span> | <span data-ttu-id="77b78-132">images</span><span class="sxs-lookup"><span data-stu-id="77b78-132">images</span></span> | <span data-ttu-id="77b78-133">使用所建立集合的名稱。</span><span class="sxs-lookup"><span data-stu-id="77b78-133">Use the name of the collection that you created.</span></span> |

1. <span data-ttu-id="77b78-134">按一下 [Azure Cosmos DB 帳戶連線] 旁的 [新增]。</span><span class="sxs-lookup"><span data-stu-id="77b78-134">Next to **Azure Cosmos DB account connection**, click **new**.</span></span> <span data-ttu-id="77b78-135">選取您先前建立的 Cosmos DB 帳戶。</span><span class="sxs-lookup"><span data-stu-id="77b78-135">Select the Cosmos DB account you previously created.</span></span>

    ![輸入 Azure Cosmos DB 輸出繫結的設定](media/functions-first-serverless-web-app/4-cosmos-db-output.png)

1. <span data-ttu-id="77b78-137">按一下 [儲存] 以建立 Cosmos DB 輸出繫結。</span><span class="sxs-lookup"><span data-stu-id="77b78-137">Click **Save** to create the Cosmos DB output binding.</span></span>

1. <span data-ttu-id="77b78-138">按一下左側的 [ResizeImage] 函式名稱來開啟該函式。</span><span class="sxs-lookup"><span data-stu-id="77b78-138">Click on the **ResizeImage** function name on the left to open the function.</span></span>

1. <span data-ttu-id="77b78-139">**C#**</span><span class="sxs-lookup"><span data-stu-id="77b78-139">**C#**</span></span>

    1. <span data-ttu-id="77b78-140">(C#) 將函式的傳回型別從 **void** 變更為 **object**。</span><span class="sxs-lookup"><span data-stu-id="77b78-140">(C#) Change the return type of the function from **void** to **object**.</span></span>

    1. <span data-ttu-id="77b78-141">(C#) 在函式結尾新增下列程式碼區塊，以傳回要儲存的文件：</span><span class="sxs-lookup"><span data-stu-id="77b78-141">(C#) At the end of the function, add the following code block to return the document to be saved:</span></span>
    
        ```csharp
        return new {
            id = name,
            imgPath = "/images/" + name,
            thumbnailPath = "/thumbnails/" + name
        };
        ```
    
        ![ResizeImages 函式在修改後的 run.csx](media/functions-first-serverless-web-app/4-update-function.png)

1. <span data-ttu-id="77b78-143">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="77b78-143">**JavaScript**</span></span>

    1. <span data-ttu-id="77b78-144">(JavaScript) 變更 `else` 子句中的 `context.done()` 陳述式，以傳回要儲存到 Cosmos DB 的文件。</span><span class="sxs-lookup"><span data-stu-id="77b78-144">(JavaScript) Change the `context.done()` statement in the `else` clause to return the document to be saved to Cosmos DB.</span></span>

    ```javascript
    if (error) {
        context.done(error);
    } else {
        context.bindings.thumbnail = stream;
        context.done(null, {
            id: context.bindingData.name,
            imgPath: "/images/" + context.bindingData.name,
            thumbnailPath: "/thumbnails/" + context.bindingData.name
        });
    }
    ```

1. <span data-ttu-id="77b78-145">按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。</span><span class="sxs-lookup"><span data-stu-id="77b78-145">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="77b78-146">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="77b78-146">Click **Save**.</span></span> <span data-ttu-id="77b78-147">檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。</span><span class="sxs-lookup"><span data-stu-id="77b78-147">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="create-a-function-to-list-images-from-cosmos-db"></a><span data-ttu-id="77b78-148">建立函式以列出 Cosmos DB 中的影像</span><span class="sxs-lookup"><span data-stu-id="77b78-148">Create a function to list images from Cosmos DB</span></span>

<span data-ttu-id="77b78-149">Web 應用程式需要 API，以擷取 Cosmos DB 中的影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="77b78-149">The web application requires an API to retrieve image metadata from Cosmos DB.</span></span> <span data-ttu-id="77b78-150">在下列步驟中。</span><span class="sxs-lookup"><span data-stu-id="77b78-150">In the following steps.</span></span> <span data-ttu-id="77b78-151">您會建立由 HTTP 觸發的函式，以使用 Cosmos DB 輸入繫結來查詢資料庫集合。</span><span class="sxs-lookup"><span data-stu-id="77b78-151">you create an HTTP triggered function that uses a Cosmos DB input binding to query the database collection.</span></span>

1. <span data-ttu-id="77b78-152">在函式應用程式中，將滑鼠停留在左邊的 [函式]，然後按一下 **+** 來建立新函式。</span><span class="sxs-lookup"><span data-stu-id="77b78-152">In your function app, hover over **Functions** on the left and click **+** to create a new function.</span></span>

1. <span data-ttu-id="77b78-153">尋找 **HttpTrigger** 樣板並加以選取。</span><span class="sxs-lookup"><span data-stu-id="77b78-153">Find the **HttpTrigger** template and select it.</span></span>

1. <span data-ttu-id="77b78-154">使用這些值來建立函式以產生取得影像 URL。</span><span class="sxs-lookup"><span data-stu-id="77b78-154">Use these values to create a function that generates a get images URL.</span></span>

    | <span data-ttu-id="77b78-155">設定</span><span class="sxs-lookup"><span data-stu-id="77b78-155">Setting</span></span>      |  <span data-ttu-id="77b78-156">建議的值</span><span class="sxs-lookup"><span data-stu-id="77b78-156">Suggested value</span></span>   | <span data-ttu-id="77b78-157">說明</span><span class="sxs-lookup"><span data-stu-id="77b78-157">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="77b78-158">**函式命名**</span><span class="sxs-lookup"><span data-stu-id="77b78-158">**Name your function**</span></span> | <span data-ttu-id="77b78-159">GetImages</span><span class="sxs-lookup"><span data-stu-id="77b78-159">GetImages</span></span> | <span data-ttu-id="77b78-160">輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。</span><span class="sxs-lookup"><span data-stu-id="77b78-160">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="77b78-161">**授權層級**</span><span class="sxs-lookup"><span data-stu-id="77b78-161">**Authorization level**</span></span> | <span data-ttu-id="77b78-162">匿名</span><span class="sxs-lookup"><span data-stu-id="77b78-162">Anonymous</span></span> | <span data-ttu-id="77b78-163">允許公開存取函式。</span><span class="sxs-lookup"><span data-stu-id="77b78-163">Allow the function to be accessed publicly.</span></span> |

1. <span data-ttu-id="77b78-164">按一下頁面底部的 [新增] 。</span><span class="sxs-lookup"><span data-stu-id="77b78-164">Click **Create**.</span></span>

1. <span data-ttu-id="77b78-165">在新函式建立時，按一下左側導覽上函式名稱底下的 [整合]。</span><span class="sxs-lookup"><span data-stu-id="77b78-165">When the new function is created, click **Integrate** under the function's name on the left navigation.</span></span>

1. <span data-ttu-id="77b78-166">按一下 [新增輸入]，然後選取 [Azure Cosmos DB]。</span><span class="sxs-lookup"><span data-stu-id="77b78-166">Click **New Input** and select **Azure Cosmos DB**.</span></span> 

    ![選取 [新增輸入]](media/functions-first-serverless-web-app/4-new-input.jpg)

1. <span data-ttu-id="77b78-168">按一下 [選取] 。</span><span class="sxs-lookup"><span data-stu-id="77b78-168">Click **Select**.</span></span>

1. <span data-ttu-id="77b78-169">填寫下列值：</span><span class="sxs-lookup"><span data-stu-id="77b78-169">Fill out the following values:</span></span>

    | <span data-ttu-id="77b78-170">設定</span><span class="sxs-lookup"><span data-stu-id="77b78-170">Setting</span></span>      |  <span data-ttu-id="77b78-171">建議的值</span><span class="sxs-lookup"><span data-stu-id="77b78-171">Suggested value</span></span>   | <span data-ttu-id="77b78-172">說明</span><span class="sxs-lookup"><span data-stu-id="77b78-172">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="77b78-173">**文件參數名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-173">**Document parameter name**</span></span> | <span data-ttu-id="77b78-174">documents</span><span class="sxs-lookup"><span data-stu-id="77b78-174">documents</span></span> | <span data-ttu-id="77b78-175">符合函式中的參數名稱。</span><span class="sxs-lookup"><span data-stu-id="77b78-175">Matches parameter name in the function.</span></span> |
    | <span data-ttu-id="77b78-176">**資料庫名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-176">**Database name**</span></span> | <span data-ttu-id="77b78-177">imagesdb</span><span class="sxs-lookup"><span data-stu-id="77b78-177">imagesdb</span></span> |  |
    | <span data-ttu-id="77b78-178">**集合名稱**</span><span class="sxs-lookup"><span data-stu-id="77b78-178">**Collection name**</span></span> | <span data-ttu-id="77b78-179">images</span><span class="sxs-lookup"><span data-stu-id="77b78-179">images</span></span> |  |
    | <span data-ttu-id="77b78-180">**SQL query**</span><span class="sxs-lookup"><span data-stu-id="77b78-180">**SQL query**</span></span> | <span data-ttu-id="77b78-181">select \* from c order by c._ts desc</span><span class="sxs-lookup"><span data-stu-id="77b78-181">select \* from c order by c._ts desc</span></span> | <span data-ttu-id="77b78-182">取得文件，最新的文件優先取得。</span><span class="sxs-lookup"><span data-stu-id="77b78-182">Get documents, latest documents first.</span></span> |
    | <span data-ttu-id="77b78-183">**Azure Cosmos DB 帳戶連線**</span><span class="sxs-lookup"><span data-stu-id="77b78-183">**Azure Cosmos DB account connection**</span></span> | <span data-ttu-id="77b78-184">選取現有連接字串</span><span class="sxs-lookup"><span data-stu-id="77b78-184">Select existing connection string</span></span> |  |

1. <span data-ttu-id="77b78-185">按一下 [儲存] 來建立輸入繫結。</span><span class="sxs-lookup"><span data-stu-id="77b78-185">Click **Save** to create the input binding.</span></span>

1. <span data-ttu-id="77b78-186">**C#**</span><span class="sxs-lookup"><span data-stu-id="77b78-186">**C#**</span></span>

    1. <span data-ttu-id="77b78-187">按一下函式的名稱以開啟程式碼視窗，然後將 **run.csx** 全部更換為 [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx) 中的內容。</span><span class="sxs-lookup"><span data-stu-id="77b78-187">Click the function's name to open the code window, and then replace all of **run.csx** with the content in [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx).</span></span>

1. <span data-ttu-id="77b78-188">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="77b78-188">**JavaScript**</span></span>

    1. <span data-ttu-id="77b78-189">按一下函式的名稱以開啟程式碼視窗，然後將 **index.js** 全部更換為 [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js) 中的內容。</span><span class="sxs-lookup"><span data-stu-id="77b78-189">Click the function's name to open the code window, and then replace all of **index.js** with the content in [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js).</span></span>

1. <span data-ttu-id="77b78-190">按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。</span><span class="sxs-lookup"><span data-stu-id="77b78-190">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="77b78-191">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="77b78-191">Click **Save**.</span></span> <span data-ttu-id="77b78-192">檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。</span><span class="sxs-lookup"><span data-stu-id="77b78-192">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-application"></a><span data-ttu-id="77b78-193">測試應用程式</span><span class="sxs-lookup"><span data-stu-id="77b78-193">Test the application</span></span>

1. <span data-ttu-id="77b78-194">在瀏覽器中開啟應用程式。</span><span class="sxs-lookup"><span data-stu-id="77b78-194">Open the application in a browser.</span></span> <span data-ttu-id="77b78-195">選取影像檔，並將它上傳。</span><span class="sxs-lookup"><span data-stu-id="77b78-195">Select an image file and upload it.</span></span>

1. <span data-ttu-id="77b78-196">幾秒後，新影像的縮圖便會出現在頁面上。</span><span class="sxs-lookup"><span data-stu-id="77b78-196">After a few seconds, the thumbnail of the new image appears on the page.</span></span>

1. <span data-ttu-id="77b78-197">在 Azure 入口網站中，使用 [搜尋] 方塊來依名稱搜尋 Cosmos DB 帳戶。</span><span class="sxs-lookup"><span data-stu-id="77b78-197">In the Azure portal, use the Search box to search for your Cosmos DB account by name.</span></span> <span data-ttu-id="77b78-198">按一下該帳戶加以開啟。</span><span class="sxs-lookup"><span data-stu-id="77b78-198">Click it to open it.</span></span>

1. <span data-ttu-id="77b78-199">按一下左側的 [資料總管] 來瀏覽集合和文件。</span><span class="sxs-lookup"><span data-stu-id="77b78-199">Click **Data Explorer** on the left to browse collections and documents.</span></span>

1. <span data-ttu-id="77b78-200">在 [imagesdb] 資料庫底下，選取 [images] 集合。</span><span class="sxs-lookup"><span data-stu-id="77b78-200">Under the **imagesdb** database, select the **images** collection.</span></span>

1. <span data-ttu-id="77b78-201">確認已針對上傳的影像建立文件。</span><span class="sxs-lookup"><span data-stu-id="77b78-201">Confirm that a document was created for the uploaded image.</span></span>

    ![資料總管，會顯示所上傳影像的文件](media/functions-first-serverless-web-app/4-data-explorer.png)



## <a name="summary"></a><span data-ttu-id="77b78-203">總結</span><span class="sxs-lookup"><span data-stu-id="77b78-203">Summary</span></span>

<span data-ttu-id="77b78-204">在本單元中，您已了解如何建立 Cosmos DB 帳戶、資料庫和集合。</span><span class="sxs-lookup"><span data-stu-id="77b78-204">In this module, you learned how to create a Cosmos DB account, database, and collection.</span></span> <span data-ttu-id="77b78-205">您也已經了解如何使用 Cosmos DB 繫結在 Cosmos DB 集合中儲存和擷取影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="77b78-205">You also learned how to use the Cosmos DB bindings to save and retrieve image metadata in the Cosmos DB collection.</span></span> <span data-ttu-id="77b78-206">接下來，您將會了解如何使用 Microsoft 認知服務為每個已上傳的影像自動產生標題。</span><span class="sxs-lookup"><span data-stu-id="77b78-206">Next, you learn how to automatically generate a caption for each uploaded image using Microsoft Cognitive Services.</span></span>
