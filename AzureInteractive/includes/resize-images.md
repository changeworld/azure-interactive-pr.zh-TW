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
ms.openlocfilehash: 88b0ac838dfa8e097a30cc6cef591377e4a95ad8
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079093"
---
<span data-ttu-id="b43d4-103">在上一個單元，您已看到無伺服器函式如何方便您從 Web 應用程式將影像安全地上傳到 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="b43d4-103">In the previous module, you saw how a serverless function can facilitate the secure uploading of images to Blob storage from a web application.</span></span> <span data-ttu-id="b43d4-104">在本單元中，您會建立另一個無伺服器函式，以監看所上傳的影像，並透過影像建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-104">In this module, you create another serverless function to watch for uploaded images and create thumbnails from them.</span></span>

## <a name="create-a-blob-storage-container-for-thumbnails"></a><span data-ttu-id="b43d4-105">建立供縮圖使用的 Blob 儲存體容器</span><span class="sxs-lookup"><span data-stu-id="b43d4-105">Create a blob storage container for thumbnails</span></span>

<span data-ttu-id="b43d4-106">完整大小的影像會儲存在名為 **images** 的容器中。</span><span class="sxs-lookup"><span data-stu-id="b43d4-106">The full size images are stored in a container named **images**.</span></span> <span data-ttu-id="b43d4-107">您需要另一個容器來儲存這些影像的縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-107">You need another container to store thumbnails of those images.</span></span>

1. <span data-ttu-id="b43d4-108">確定您仍登入 Cloud Shell (bash)。</span><span class="sxs-lookup"><span data-stu-id="b43d4-108">Ensure you are still signed in to the Cloud Shell (bash).</span></span>  <span data-ttu-id="b43d4-109">如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。</span><span class="sxs-lookup"><span data-stu-id="b43d4-109">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span> 

1. <span data-ttu-id="b43d4-110">在儲存體帳戶中，使用可存取所有 Blob 的公用權限建立名為 **thumbnails** 的新容器。</span><span class="sxs-lookup"><span data-stu-id="b43d4-110">Create a new container named **thumbnails** in your storage account with public access to all blobs.</span></span>

    ```azurecli
    az storage container create -n thumbnails --account-name <storage account name> --public-access blob
    ```


## <a name="create-a-blob-triggered-serverless-function"></a><span data-ttu-id="b43d4-111">建立由 Blob 觸發的無伺服器函式</span><span class="sxs-lookup"><span data-stu-id="b43d4-111">Create a blob-triggered serverless function</span></span>

<span data-ttu-id="b43d4-112">觸發程序會定義叫用函式的方式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-112">A trigger defines how a function is invoked.</span></span> <span data-ttu-id="b43d4-113">您接下來建立的函式會使用 Blob 觸發程序：有 Blob (影像檔) 上傳至 **images** 容器時，便會自動叫用函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-113">The function you create next uses a blob trigger: the function is automatically invoked when a blob (image file) is uploaded to the **images** container.</span></span> <span data-ttu-id="b43d4-114">函式必須有一個觸發程序。</span><span class="sxs-lookup"><span data-stu-id="b43d4-114">A function must have one trigger.</span></span> <span data-ttu-id="b43d4-115">觸發程序具有相關聯的資料，它通常是觸發函數的承載。</span><span class="sxs-lookup"><span data-stu-id="b43d4-115">Triggers have associated data, which is usually the payload that triggered the function.</span></span>

<span data-ttu-id="b43d4-116">繫結會定義函式在 Azure 或第三方服務中讀取或寫入資料的方式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-116">Bindings define how a function reads or writes data in Azure or third-party services.</span></span> <span data-ttu-id="b43d4-117">此函式會針對觸發函式的影像建立其縮圖版本，並將縮圖儲存在 *thumbnails* 容器中。</span><span class="sxs-lookup"><span data-stu-id="b43d4-117">This function creates a thumbnail version of the image that triggers it and saves the thumbnail in a *thumbnails* container.</span></span>

1. <span data-ttu-id="b43d4-118">在 Azure 入口網站中開啟函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-118">Open your function app in the Azure Portal.</span></span>

1. <span data-ttu-id="b43d4-119">在函式應用程式視窗的左側導覽中，將滑鼠停留在 [函式] 上，然後按一下 **+** 以開始建立新的無伺服器函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-119">In the function app window's left hand navigation, hover over **Functions** and click **+** to start creating a new serverless function.</span></span> <span data-ttu-id="b43d4-120">如果出現 [快速入門] 頁面，請按一下 [自訂函式] 以查看函式樣板清單。</span><span class="sxs-lookup"><span data-stu-id="b43d4-120">If a quickstart page appears, click **Custom function** to see a list of function templates.</span></span>

1. <span data-ttu-id="b43d4-121">尋找 **BlobTrigger** 樣板並加以選取。</span><span class="sxs-lookup"><span data-stu-id="b43d4-121">Find the **BlobTrigger** template and select it.</span></span>

1. <span data-ttu-id="b43d4-122">使用這些值來建立函式，以在影像上傳時建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-122">Use these values to create a function that creates thumbnails as images are uploaded.</span></span>

    | <span data-ttu-id="b43d4-123">設定</span><span class="sxs-lookup"><span data-stu-id="b43d4-123">Setting</span></span>      |  <span data-ttu-id="b43d4-124">建議的值</span><span class="sxs-lookup"><span data-stu-id="b43d4-124">Suggested value</span></span>   | <span data-ttu-id="b43d4-125">說明</span><span class="sxs-lookup"><span data-stu-id="b43d4-125">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="b43d4-126">**語言**</span><span class="sxs-lookup"><span data-stu-id="b43d4-126">**Language**</span></span> | <span data-ttu-id="b43d4-127">C# 或 JavaScript</span><span class="sxs-lookup"><span data-stu-id="b43d4-127">C# or JavaScript</span></span> | <span data-ttu-id="b43d4-128">選擇慣用語言。</span><span class="sxs-lookup"><span data-stu-id="b43d4-128">Choose your preferred language.</span></span> |
    | <span data-ttu-id="b43d4-129">**函式命名**</span><span class="sxs-lookup"><span data-stu-id="b43d4-129">**Name your function**</span></span> | <span data-ttu-id="b43d4-130">ResizeImage</span><span class="sxs-lookup"><span data-stu-id="b43d4-130">ResizeImage</span></span> | <span data-ttu-id="b43d4-131">輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-131">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="b43d4-132">**路徑**</span><span class="sxs-lookup"><span data-stu-id="b43d4-132">**Path**</span></span> | <span data-ttu-id="b43d4-133">images/{name}</span><span class="sxs-lookup"><span data-stu-id="b43d4-133">images/{name}</span></span> | <span data-ttu-id="b43d4-134">當 **images** 容器中出現檔案時，執行函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-134">Execute the function when a file appears in the **images** container.</span></span> |
    | <span data-ttu-id="b43d4-135">**儲存體帳戶資訊**</span><span class="sxs-lookup"><span data-stu-id="b43d4-135">**Storage account information**</span></span> | <span data-ttu-id="b43d4-136">AZURE_STORAGE_CONNECTION_STRING</span><span class="sxs-lookup"><span data-stu-id="b43d4-136">AZURE_STORAGE_CONNECTION_STRING</span></span> | <span data-ttu-id="b43d4-137">使用先前透過連接字串所建立的環境變數。</span><span class="sxs-lookup"><span data-stu-id="b43d4-137">Use the environment variable previously created with the connection string.</span></span> |

    ![輸入新函式的設定](media/functions-first-serverless-web-app/3-new-function.png)

1. <span data-ttu-id="b43d4-139">按一下 [建立] 以建立函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-139">Click **Create** to create the function.</span></span>

1. <span data-ttu-id="b43d4-140">函式建立好時，按一下 [整合] 以檢視其觸發程序、輸入和輸出繫結。</span><span class="sxs-lookup"><span data-stu-id="b43d4-140">When the function is created, click **Integrate** to view its trigger, input, and output bindings.</span></span>

1. <span data-ttu-id="b43d4-141">按一下 [新增輸出] 來建立新的輸出繫結。</span><span class="sxs-lookup"><span data-stu-id="b43d4-141">Click **New output** to create a new output binding.</span></span>

    ![選取 [整合] 索引標籤上的 [新增輸出]](media/functions-first-serverless-web-app/3-new-output.jpg)

1. <span data-ttu-id="b43d4-143">選取 [Azure Blob 儲存體]，然後按一下 [選取]。</span><span class="sxs-lookup"><span data-stu-id="b43d4-143">Select **Azure Blob Storage** and click **Select**.</span></span> <span data-ttu-id="b43d4-144">您可能必須向下捲動才能看到 [選取] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="b43d4-144">You may have to scroll down to see the **Select** button.</span></span>

    ![選取 Azure Blob 儲存體](media/functions-first-serverless-web-app/3-storage-output.jpg)

1. <span data-ttu-id="b43d4-146">輸入下列值。</span><span class="sxs-lookup"><span data-stu-id="b43d4-146">Enter the following values.</span></span>

    | <span data-ttu-id="b43d4-147">設定</span><span class="sxs-lookup"><span data-stu-id="b43d4-147">Setting</span></span>      |  <span data-ttu-id="b43d4-148">建議的值</span><span class="sxs-lookup"><span data-stu-id="b43d4-148">Suggested value</span></span>   | <span data-ttu-id="b43d4-149">說明</span><span class="sxs-lookup"><span data-stu-id="b43d4-149">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="b43d4-150">**Blob 參數名稱**</span><span class="sxs-lookup"><span data-stu-id="b43d4-150">**Blob parameter name**</span></span> | <span data-ttu-id="b43d4-151">thumbnail</span><span class="sxs-lookup"><span data-stu-id="b43d4-151">thumbnail</span></span> | <span data-ttu-id="b43d4-152">函式會搭配使用參數與此名稱來寫入縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-152">The function uses the parameter with this name to write the thumbnail.</span></span> |
    | <span data-ttu-id="b43d4-153">**使用函數傳回值**</span><span class="sxs-lookup"><span data-stu-id="b43d4-153">**Use function return value**</span></span> | <span data-ttu-id="b43d4-154">否</span><span class="sxs-lookup"><span data-stu-id="b43d4-154">No</span></span> |  |
    | <span data-ttu-id="b43d4-155">**路徑**</span><span class="sxs-lookup"><span data-stu-id="b43d4-155">**Path**</span></span> | <span data-ttu-id="b43d4-156">thumbnails/{name}</span><span class="sxs-lookup"><span data-stu-id="b43d4-156">thumbnails/{name}</span></span> | <span data-ttu-id="b43d4-157">縮圖會寫入到名為 **thumbnails** 的容器。</span><span class="sxs-lookup"><span data-stu-id="b43d4-157">The thumbnails are written to a container named **thumbnails**.</span></span> |
    | <span data-ttu-id="b43d4-158">**儲存體帳戶連線**</span><span class="sxs-lookup"><span data-stu-id="b43d4-158">**Storage account connection**</span></span> | <span data-ttu-id="b43d4-159">AZURE_STORAGE_CONNECTION_STRING</span><span class="sxs-lookup"><span data-stu-id="b43d4-159">AZURE_STORAGE_CONNECTION_STRING</span></span> | <span data-ttu-id="b43d4-160">使用先前透過連接字串所建立的環境變數。</span><span class="sxs-lookup"><span data-stu-id="b43d4-160">Use the environment variable previously created with the connection string.</span></span> |

    ![輸入 Blob 輸出繫結的設定](media/functions-first-serverless-web-app/3-blob-output.png)

1. <span data-ttu-id="b43d4-162">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="b43d4-162">**JavaScript**</span></span>

    1. <span data-ttu-id="b43d4-163">(JavaScript) 按一下視窗右上角的 [進階編輯器] 來顯示代表繫結的 JSON。</span><span class="sxs-lookup"><span data-stu-id="b43d4-163">(JavaScript) Click on **Advanced editor** in the top right corner of the window to reveal the JSON representing the bindings.</span></span>

    1. <span data-ttu-id="b43d4-164">(JavaScript) 在 `blobTrigger` 繫結中，新增名為 `dataType` 且值為 `binary` 的屬性。</span><span class="sxs-lookup"><span data-stu-id="b43d4-164">(JavaScript) In the `blobTrigger` binding, add a property named `dataType` with a value of `binary`.</span></span> <span data-ttu-id="b43d4-165">這會將繫結設定為以二進位資料的形式將 Blob 內容傳遞給函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-165">This configures the binding to pass the blob contents to the function as binary data.</span></span>

    ```json
    {
      "name": "myBlob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "images/{name}",
      "connection": "AZURE_STORAGE_CONNECTION_STRING",
      "dataType": "binary"
    }
    ```

1. <span data-ttu-id="b43d4-166">按一下 [儲存] 來建立新的繫結。</span><span class="sxs-lookup"><span data-stu-id="b43d4-166">Click **Save** to create the new binding.</span></span>

1. <span data-ttu-id="b43d4-167">**C#**</span><span class="sxs-lookup"><span data-stu-id="b43d4-167">**C#**</span></span>

    1. <span data-ttu-id="b43d4-168">(C#) 選取左側導覽上的 [ResizeImage] 函式名稱，以開啟函式的原始程式碼。</span><span class="sxs-lookup"><span data-stu-id="b43d4-168">(C#) Select the **ResizeImage** function name on the left navigation to open the function's source code.</span></span>

    1. <span data-ttu-id="b43d4-169">(C#) 函式需要名為 **ImageResizer** 的 NuGet 套件來產生縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-169">(C#) The function requires a NuGet package called **ImageResizer** to generate the thumbnails.</span></span> <span data-ttu-id="b43d4-170">NuGet 套件會使用 **project.json** 檔案新增至 C# 函式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-170">NuGet packages are added to C# functions using a **project.json** file.</span></span> <span data-ttu-id="b43d4-171">若要建立檔案，請按一下右邊的 [檢視檔案] 以顯示構成函式的檔案。</span><span class="sxs-lookup"><span data-stu-id="b43d4-171">To create the file, click **View Files** on the right to reveal the files that make up the function.</span></span>
    
    1. <span data-ttu-id="b43d4-172">(C#) 按一下 [新增] 以新增名為 **project.json** 的新檔案。</span><span class="sxs-lookup"><span data-stu-id="b43d4-172">(C#) Click **Add** to add a new file named **project.json**.</span></span>
    
    1. <span data-ttu-id="b43d4-173">(C#) 將 [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) 的內容複製到新建立的檔案。</span><span class="sxs-lookup"><span data-stu-id="b43d4-173">(C#) Copy the contents of [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) into the newly created file.</span></span> <span data-ttu-id="b43d4-174">儲存檔案。</span><span class="sxs-lookup"><span data-stu-id="b43d4-174">Save the file.</span></span> <span data-ttu-id="b43d4-175">檔案更新時，便會自動還原套件。</span><span class="sxs-lookup"><span data-stu-id="b43d4-175">Packages are automatically restored when the file is updated.</span></span>
    
        ![具有 ImageResizer 的 project.json 檔案](media/functions-first-serverless-web-app/3-project-json.png)
    
    1. <span data-ttu-id="b43d4-177">(C#) 選取 [檢視檔案] 下方的 [run.csx]，並將其內容更換為 [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx) 中的內容。</span><span class="sxs-lookup"><span data-stu-id="b43d4-177">(C#) Select **run.csx** under **View Files** and replace its content with the content in [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx).</span></span>

1. <span data-ttu-id="b43d4-178">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="b43d4-178">**JavaScript**</span></span> 

    1. <span data-ttu-id="b43d4-179">(JavaScript) 此函式需要 npm 的 `jimp` 套件才能調整影像大小。</span><span class="sxs-lookup"><span data-stu-id="b43d4-179">(JavaScript) This function requires the `jimp` package from npm to resize the photo.</span></span> <span data-ttu-id="b43d4-180">若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。</span><span class="sxs-lookup"><span data-stu-id="b43d4-180">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="b43d4-181">(JavaScript) 按一下 [主控台] 以顯示主控台視窗。</span><span class="sxs-lookup"><span data-stu-id="b43d4-181">(JavaScript) Click **Console** to reveal a console window.</span></span>

    1. <span data-ttu-id="b43d4-182">(JavaScript) 在主控台中執行 `npm install jimp` 命令。</span><span class="sxs-lookup"><span data-stu-id="b43d4-182">(JavaScript) Run the command `npm install jimp` in the console.</span></span> <span data-ttu-id="b43d4-183">此作業可能需要一兩分鐘的時間才能完成。</span><span class="sxs-lookup"><span data-stu-id="b43d4-183">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="b43d4-184">(JavaScript) 在左側導覽中按一下 [ResizeImage] 函式名稱來顯示函式，並將 **index.js** 全部更換為 [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js) 的內容。</span><span class="sxs-lookup"><span data-stu-id="b43d4-184">(JavaScript) Click on the **ResizeImage** function name in the left navigation to reveal the function, replace all of **index.js** with the content of [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js).</span></span>

1. <span data-ttu-id="b43d4-185">按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。</span><span class="sxs-lookup"><span data-stu-id="b43d4-185">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="b43d4-186">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="b43d4-186">Click **Save**.</span></span> <span data-ttu-id="b43d4-187">檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。</span><span class="sxs-lookup"><span data-stu-id="b43d4-187">Check the logs panel to ensure the function is successfully saved and there are no errors.</span></span>


## <a name="test-the-serverless-function"></a><span data-ttu-id="b43d4-188">測試無伺服器函式</span><span class="sxs-lookup"><span data-stu-id="b43d4-188">Test the serverless function</span></span>

1. <span data-ttu-id="b43d4-189">在瀏覽器中開啟應用程式。</span><span class="sxs-lookup"><span data-stu-id="b43d4-189">Open the application in a browser.</span></span> <span data-ttu-id="b43d4-190">選取影像檔，並將它上傳。</span><span class="sxs-lookup"><span data-stu-id="b43d4-190">Select an image file and upload it.</span></span> <span data-ttu-id="b43d4-191">上傳完成，但因為我們尚未新增顯示影像的功能，應用程式不會顯示已上傳的影像。</span><span class="sxs-lookup"><span data-stu-id="b43d4-191">The upload completes, but because we haven't added the ability to display images yet, the app doesn't show the uploaded photo.</span></span>

1. <span data-ttu-id="b43d4-192">在 Cloud Shell 中，確認影像已上傳到 [images] 容器。</span><span class="sxs-lookup"><span data-stu-id="b43d4-192">In the Cloud Shell, confirm the image was uploaded to the **images** container.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. <span data-ttu-id="b43d4-193">確認名為 **thumbnails** 的容器中已建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-193">Confirm the thumbnail was created in a container named **thumbnails**.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c thumbnails -o table
    ```

1. <span data-ttu-id="b43d4-194">取得縮圖的 URL。</span><span class="sxs-lookup"><span data-stu-id="b43d4-194">Obtain the thumbnail's URL.</span></span>

    ```azurecli
    az storage blob url --account-name <storage account name> -c thumbnails -n <filename> --output tsv
    ```

    <span data-ttu-id="b43d4-195">在瀏覽器中開啟 URL，並確認是否已正確建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-195">Open the URL in a browser and confirm the thumbnail was properly created.</span></span>

1. <span data-ttu-id="b43d4-196">在繼續進行下一個教學課程之前，請先刪除 [images] 和 [thumbnails] 容器中的所有檔案。</span><span class="sxs-lookup"><span data-stu-id="b43d4-196">Before moving on to the next tutorial, delete all files in the **images** and **thumbnails** containers.</span></span>

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```
    ```azurecli
    az storage blob delete-batch -s thumbnails --account-name <storage account name>
    ```


## <a name="summary"></a><span data-ttu-id="b43d4-197">總結</span><span class="sxs-lookup"><span data-stu-id="b43d4-197">Summary</span></span>

<span data-ttu-id="b43d4-198">在本單元中，您已建立無伺服器函式，以在每次有影像上傳至 Blob 儲存體容器時建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="b43d4-198">In this module, you created a serverless function to create a thumbnail whenever an image is uploaded to a Blob storage container.</span></span> <span data-ttu-id="b43d4-199">接下來，您將會了解如何使用 Azure Cosmos DB 來儲存和列出影像中繼資料。</span><span class="sxs-lookup"><span data-stu-id="b43d4-199">Next, you learn how to use Azure Cosmos DB to store and list image metadata.</span></span>
