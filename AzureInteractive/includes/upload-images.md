---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
manager: jeconnoc
ms.service: multiple
ms.topic: include
ms.date: 10/12/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 3779c2e130afa7ee8d5879f30a924e258b7a41e9
ms.sourcegitcommit: fdb43556b8dcf67cb39c18e532b5fab7ac53eaee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49315971"
---
<span data-ttu-id="2e1de-103">您所要建置的應用程式是影像中心。</span><span class="sxs-lookup"><span data-stu-id="2e1de-103">The application that you're building is a photo gallery.</span></span> <span data-ttu-id="2e1de-104">它會使用用戶端 JavaScript 來呼叫 API，以便上傳和顯示影像。</span><span class="sxs-lookup"><span data-stu-id="2e1de-104">It uses client-side JavaScript to call APIs to upload and display images.</span></span> <span data-ttu-id="2e1de-105">在本單元中，您會使用無伺服器函式建立 API，以產生有時間限制的 URL 來上傳影像。</span><span class="sxs-lookup"><span data-stu-id="2e1de-105">In this module, you create an API using a serverless function that generates a time-limited URL to upload an image.</span></span> <span data-ttu-id="2e1de-106">Web 應用程式會使用所產生的 URL，透過 [Blob 儲存體 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 將影像上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="2e1de-106">The web application uses the generated URL to upload an image to Blob storage using the [Blob storage REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api).</span></span>

## <a name="create-a-blob-storage-container-for-images"></a><span data-ttu-id="2e1de-107">建立供影像使用的 Blob 儲存體容器</span><span class="sxs-lookup"><span data-stu-id="2e1de-107">Create a blob storage container for images</span></span>

<span data-ttu-id="2e1de-108">應用程式需要不同的儲存體容器來上傳和裝載影像。</span><span class="sxs-lookup"><span data-stu-id="2e1de-108">The application requires a separate storage container to upload and host images.</span></span>

1. <span data-ttu-id="2e1de-109">確定您仍登入 Cloud Shell (bash)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-109">Ensure you are still signed in to the Cloud Shell (bash).</span></span> <span data-ttu-id="2e1de-110">如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。</span><span class="sxs-lookup"><span data-stu-id="2e1de-110">If not, select **Enter focus mode** to open a Cloud Shell window.</span></span>

1.  <span data-ttu-id="2e1de-111">在儲存體帳戶中，使用可存取所有 Blob 的公用權限建立名為 **images** 的新容器。</span><span class="sxs-lookup"><span data-stu-id="2e1de-111">Create a new container named **images** in your storage account with public access to all blobs.</span></span>

    ```azurecli
    az storage container create -n images --account-name <storage account name> --public-access blob
    ```

## <a name="create-an-azure-function-app"></a><span data-ttu-id="2e1de-112">建立 Azure 函數應用程式</span><span class="sxs-lookup"><span data-stu-id="2e1de-112">Create an Azure Function app</span></span>

<span data-ttu-id="2e1de-113">Azure Functions 是用於執行無伺服器函式的服務。</span><span class="sxs-lookup"><span data-stu-id="2e1de-113">Azure Functions is a service for running serverless functions.</span></span> <span data-ttu-id="2e1de-114">無伺服器函式可由事件 (例如，HTTP 要求) 或在儲存體容器中建立 Blob 時觸發 (呼叫)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-114">A serverless function can be triggered (called) by events such as an HTTP request or when a blob is created in a storage container.</span></span>

<span data-ttu-id="2e1de-115">Azure 函式應用程式是供一或多個無伺服器函式使用的容器。</span><span class="sxs-lookup"><span data-stu-id="2e1de-115">An Azure Function app is a container for one or more serverless functions.</span></span>

<span data-ttu-id="2e1de-116">在先前所建立名為 **first-serverless-app** 的資源群組中，使用唯一名稱建立新的 Azure 函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-116">Create a new Azure Function app with a unique name in the resource group you created earlier named **first-serverless-app**.</span></span> <span data-ttu-id="2e1de-117">函式應用程式需要儲存體帳戶；在本教學課程中，您會使用現有儲存體帳戶。</span><span class="sxs-lookup"><span data-stu-id="2e1de-117">Function apps require a Storage account; in this tutorial, you use the existing storage account.</span></span>

```azurecli
az functionapp create -n <function app name> -g first-serverless-app -s <storage account name> -c westcentralus
```

## <a name="configure-the-function-app"></a><span data-ttu-id="2e1de-118">設定函式應用程式</span><span class="sxs-lookup"><span data-stu-id="2e1de-118">Configure the function app</span></span>

<span data-ttu-id="2e1de-119">本教學課程中的函式應用程式需要使用版本 1.x 的 Azure Functions 執行階段。</span><span class="sxs-lookup"><span data-stu-id="2e1de-119">The function app in this tutorial requires version 1.x of the Functions runtime.</span></span> <span data-ttu-id="2e1de-120">將 `FUNCTIONS_EXTENSION_VERSION` 應用程式設定設為 `~1`，以將函式應用程式釘選至最新的 1.x 版本。</span><span class="sxs-lookup"><span data-stu-id="2e1de-120">Setting the `FUNCTIONS_EXTENSION_VERSION` application setting to `~1` pins the function app to the latest 1.x version.</span></span> <span data-ttu-id="2e1de-121">使用 [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) 命令設定應用程式設定。</span><span class="sxs-lookup"><span data-stu-id="2e1de-121">Set application settings with the [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) command.</span></span>

<span data-ttu-id="2e1de-122">在下列 Azure CLI 命令中，\`<app_name> 是您函數應用程式的名稱。</span><span class="sxs-lookup"><span data-stu-id="2e1de-122">In the following Azure CLI command, \`<app_name> is the name of your function app.</span></span>

```azurecli
az functionapp config appsettings set --name <function app name> --g first-serverless-app --settings FUNCTIONS_EXTENSION_VERSION=~1
```

## <a name="create-an-http-triggered-serverless-function"></a><span data-ttu-id="2e1de-123">建立由 HTTP 觸發的無伺服器函式</span><span class="sxs-lookup"><span data-stu-id="2e1de-123">Create an HTTP-triggered serverless function</span></span>

<span data-ttu-id="2e1de-124">影像中心 Web 應用程式會對無伺服器函式發出 HTTP 要求，以產生有時間限制的 URL 將影像安全地上傳至 Blob 儲存體中。</span><span class="sxs-lookup"><span data-stu-id="2e1de-124">The photo gallery web application makes an HTTP request to serverless function to generate a time-limited URL to securely upload an image to Blob storage.</span></span> <span data-ttu-id="2e1de-125">此函式會由 HTTP 要求來觸發，並使用 Azure 儲存體 SDK 來產生及傳回安全的 URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-125">The function is triggered by an HTTP request and uses the Azure Storage SDK to generate and return the secure URL.</span></span>

1. <span data-ttu-id="2e1de-126">建立函式應用程式之後，請在 Azure 入口網站中使用 [搜尋] 方塊搜尋該應用程式，並按一下來加以開啟。</span><span class="sxs-lookup"><span data-stu-id="2e1de-126">After the Function app is created, search for it in the Azure Portal using the Search box and click to open it.</span></span>

    ![開啟函式應用程式](media/functions-first-serverless-web-app/2-search-function-app.png)

1. <span data-ttu-id="2e1de-128">在函式應用程式視窗的左側導覽中，將滑鼠停留在 [函式] 上，然後按一下 **+** 以開始建立新的無伺服器函式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-128">In the function app window's left hand navigation, hover over **Functions** and click **+** to start creating a new serverless function.</span></span>

    ![建立新函式](media/functions-first-serverless-web-app/2-new-function.png)

1. <span data-ttu-id="2e1de-130">按一下 [自訂函式] 以查看函式樣板清單。</span><span class="sxs-lookup"><span data-stu-id="2e1de-130">Click **Custom function** to see a list of function templates.</span></span>

1. <span data-ttu-id="2e1de-131">尋找 **HttpTrigger** 樣板，然後按一下要使用的語言 (C# 或 JavaScript)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-131">Find the **HttpTrigger** template and click the language to use (C# or JavaScript).</span></span>

1. <span data-ttu-id="2e1de-132">使用這些值來建立函式以產生 Blob 上傳 URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-132">Use these values to create a function that generates a blob upload URL.</span></span>

    | <span data-ttu-id="2e1de-133">設定</span><span class="sxs-lookup"><span data-stu-id="2e1de-133">Setting</span></span>      |  <span data-ttu-id="2e1de-134">建議的值</span><span class="sxs-lookup"><span data-stu-id="2e1de-134">Suggested value</span></span>   | <span data-ttu-id="2e1de-135">說明</span><span class="sxs-lookup"><span data-stu-id="2e1de-135">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="2e1de-136">**語言**</span><span class="sxs-lookup"><span data-stu-id="2e1de-136">**Language**</span></span> | <span data-ttu-id="2e1de-137">C# 或 JavaScript</span><span class="sxs-lookup"><span data-stu-id="2e1de-137">C# or JavaScript</span></span> | <span data-ttu-id="2e1de-138">選取您想要使用的語言。</span><span class="sxs-lookup"><span data-stu-id="2e1de-138">Select the language you want to use.</span></span> |
    | <span data-ttu-id="2e1de-139">**函式命名**</span><span class="sxs-lookup"><span data-stu-id="2e1de-139">**Name your function**</span></span> | <span data-ttu-id="2e1de-140">GetUploadUrl</span><span class="sxs-lookup"><span data-stu-id="2e1de-140">GetUploadUrl</span></span> | <span data-ttu-id="2e1de-141">輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-141">Type this name exactly as shown so the application can discover the function.</span></span> |
    | <span data-ttu-id="2e1de-142">**授權層級**</span><span class="sxs-lookup"><span data-stu-id="2e1de-142">**Authorization level**</span></span> | <span data-ttu-id="2e1de-143">匿名</span><span class="sxs-lookup"><span data-stu-id="2e1de-143">Anonymous</span></span> | <span data-ttu-id="2e1de-144">允許公開存取函式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-144">Allow the function to be accessed publicly.</span></span> |

    ![針對由 HTTP 觸發的新函式輸入設定](media/functions-first-serverless-web-app/2-new-function-httptrigger.png)

1. <span data-ttu-id="2e1de-146">按一下 [建立] 以建立函式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-146">Click **Create** to create the function.</span></span>

1. <span data-ttu-id="2e1de-147">**C#**</span><span class="sxs-lookup"><span data-stu-id="2e1de-147">**C#**</span></span> 

    1. <span data-ttu-id="2e1de-148">當函式的原始程式碼出現時，將 **run.csx** 全部更換為 [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx) 的內容。</span><span class="sxs-lookup"><span data-stu-id="2e1de-148">When the function's source code appears, replace all of **run.csx** with the content of [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx).</span></span>

1. <span data-ttu-id="2e1de-149">**JavaScript**</span><span class="sxs-lookup"><span data-stu-id="2e1de-149">**JavaScript**</span></span> 

    1. <span data-ttu-id="2e1de-150">(JavaScript) 此函式需要 npm 的 `azure-storage` 套件，以產生所需的共用存取簽章 (SAS) 權杖來建立安全 URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-150">(JavaScript) This function requires the `azure-storage` package from npm to generate the shared access signature (SAS) token required to build the secure URL.</span></span> <span data-ttu-id="2e1de-151">若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。</span><span class="sxs-lookup"><span data-stu-id="2e1de-151">To install the npm package, click on the Function App's name on the left navigation and click **Platform features**.</span></span>

    1. <span data-ttu-id="2e1de-152">(JavaScript) 按一下 [主控台] 以顯示主控台視窗。</span><span class="sxs-lookup"><span data-stu-id="2e1de-152">(JavaScript) Click **Console** to reveal a console window.</span></span>

        ![開啟主控台視窗](media/functions-first-serverless-web-app/2-open-console.jpg)

    1. <span data-ttu-id="2e1de-154">(JavaScript) 執行 `cd d:\home\site\wwwroot` 命令，確定目前的目錄為 **d:\home\site\wwwroot**。</span><span class="sxs-lookup"><span data-stu-id="2e1de-154">(JavaScript) Ensure the current directory is **d:\home\site\wwwroot** by running the command `cd d:\home\site\wwwroot`.</span></span>

    1. <span data-ttu-id="2e1de-155">(JavaScript) 執行 `npm init -y` 命令以建立空白 **package.json** 檔案。</span><span class="sxs-lookup"><span data-stu-id="2e1de-155">(JavaScript) Run the command `npm init -y` to create an empty **package.json** file.</span></span>

    1. <span data-ttu-id="2e1de-156">(JavaScript) 在主控台執行 `npm install --save azure-storage` 命令以安裝套件，並將它儲存在 **package.json**。</span><span class="sxs-lookup"><span data-stu-id="2e1de-156">(JavaScript) Run the command `npm install --save azure-storage` in the console to install the package and save it in **package.json**.</span></span> <span data-ttu-id="2e1de-157">此作業可能需要一兩分鐘的時間才能完成。</span><span class="sxs-lookup"><span data-stu-id="2e1de-157">It may take a minute or two to complete the operation.</span></span>

    1. <span data-ttu-id="2e1de-158">(JavaScript) 在左側導覽中按一下函式名稱 (**GetUploadUrl**) 來顯示函式，並將 **index.js** 全部更換為 [**javascript/GetUploadUrl /index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js) 的內容。</span><span class="sxs-lookup"><span data-stu-id="2e1de-158">(JavaScript) Click on the function name (**GetUploadUrl**) in the left navigation to reveal the function, replace all of **index.js** with the content of [**javascript/GetUploadUrl/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js).</span></span>
    
        ![更新後的 index.js](media/functions-first-serverless-web-app/2-paste-js.jpg)

1. <span data-ttu-id="2e1de-160">按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。</span><span class="sxs-lookup"><span data-stu-id="2e1de-160">Click **Logs** below the code window to expand the logs panel.</span></span>

1. <span data-ttu-id="2e1de-161">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="2e1de-161">Click **Save**.</span></span> <span data-ttu-id="2e1de-162">檢查 [記錄] 面板，以確定函式已成功完成編譯。</span><span class="sxs-lookup"><span data-stu-id="2e1de-162">Check the logs panel to ensure the function is successfully compiled.</span></span>

<span data-ttu-id="2e1de-163">函式會產生所謂的共用存取簽章 (SAS) URL，以便用來將檔案上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="2e1de-163">The function generates what is called a shared access signature (SAS) URL that is used to upload a file to Blob storage.</span></span> <span data-ttu-id="2e1de-164">SAS URL 的有效時間很短，且只允許上傳一個檔案。</span><span class="sxs-lookup"><span data-stu-id="2e1de-164">The SAS URL is valid for a short period of time and only allows a single file to be uploaded.</span></span> <span data-ttu-id="2e1de-165">如需如何[使用共用存取簽章](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)的詳細資訊，請參閱 Blob 儲存體文。</span><span class="sxs-lookup"><span data-stu-id="2e1de-165">Consult the Blob storage documentation for more information on [using shared access signatures](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1).</span></span>


## <a name="add-an-environment-variable-for-the-storage-connection-string"></a><span data-ttu-id="2e1de-166">針對儲存體連接字串新增環境變數</span><span class="sxs-lookup"><span data-stu-id="2e1de-166">Add an environment variable for the storage connection string</span></span>

<span data-ttu-id="2e1de-167">您剛才建立的函式需要儲存體帳戶的連接字串，才能產生 SAS URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-167">The function you just created requires a connection string for the Storage account so that it can generate the SAS URL.</span></span> <span data-ttu-id="2e1de-168">您不一定要將連接字串硬式編碼到函式主體中，也可以將它儲存為應用程式設定。</span><span class="sxs-lookup"><span data-stu-id="2e1de-168">Instead of hardcoding the connection string in the function's body, it can be stored as an application setting.</span></span> <span data-ttu-id="2e1de-169">應用程式設定可作為環境變數供函式應用程式中的所有函式存取。</span><span class="sxs-lookup"><span data-stu-id="2e1de-169">Application settings are accessible as environment variables by all functions in the Function app.</span></span>

1. <span data-ttu-id="2e1de-170">在 Cloud Shell 中查詢儲存體帳戶連接字串，並將它儲存到名為**STORAGE_CONNECTION_STRING** 的 Bash 變數。</span><span class="sxs-lookup"><span data-stu-id="2e1de-170">In the Cloud Shell, query the Storage account connection string and save it to a bash variable named **STORAGE_CONNECTION_STRING**.</span></span>

    ```azurecli
    export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n <storage account name> -g first-serverless-app --query "connectionString" --output tsv)
    ```

    <span data-ttu-id="2e1de-171">請確認您已成功設定好變數。</span><span class="sxs-lookup"><span data-stu-id="2e1de-171">Confirm the variable is set successfully.</span></span>

    ```azurecli
    echo $STORAGE_CONNECTION_STRING
    ```

1. <span data-ttu-id="2e1de-172">使用上一個步驟所儲存的值，建立名為 **AZURE_STORAGE_CONNECTION_STRING** 的新應用程式設定。</span><span class="sxs-lookup"><span data-stu-id="2e1de-172">Create a new application setting named **AZURE_STORAGE_CONNECTION_STRING** using the value saved from the previous step.</span></span>

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING -o table
    ```

    <span data-ttu-id="2e1de-173">請確認命令輸出中所包含的新應用程式設定有正確的值。</span><span class="sxs-lookup"><span data-stu-id="2e1de-173">Confirm that the command's output contains the new application setting with the correct value.</span></span>


## <a name="test-the-serverless-function"></a><span data-ttu-id="2e1de-174">測試無伺服器函式</span><span class="sxs-lookup"><span data-stu-id="2e1de-174">Test the serverless function</span></span>

<span data-ttu-id="2e1de-175">除了建立和編輯函式，Azure 入口網站也提供內建的函式測試工具。</span><span class="sxs-lookup"><span data-stu-id="2e1de-175">In addition to creating and editing functions, the Azure portal also provides an built-in tool for testing functions.</span></span>

1. <span data-ttu-id="2e1de-176">若要測試 HTTP 無伺服器函式，請按一下程式碼視窗右側的 [測試] 索引標籤，以展開 [測試] 面板。</span><span class="sxs-lookup"><span data-stu-id="2e1de-176">To test the HTTP serverless function, click on the **Test** tab on the right of the code window to expand the test panel.</span></span>

1. <span data-ttu-id="2e1de-177">將 [HTTP 方法] 變更為 [GET]。</span><span class="sxs-lookup"><span data-stu-id="2e1de-177">Change the **Http method** to **GET**.</span></span>

1. <span data-ttu-id="2e1de-178">在 [查詢] 之下，按一下 [新增參數]\* 並新增下列參數：</span><span class="sxs-lookup"><span data-stu-id="2e1de-178">Under **Query**, click *Add parameter*\* and add the following parameter:</span></span>

    | <span data-ttu-id="2e1de-179">name</span><span class="sxs-lookup"><span data-stu-id="2e1de-179">name</span></span>      |  <span data-ttu-id="2e1de-180">value</span><span class="sxs-lookup"><span data-stu-id="2e1de-180">value</span></span>   | 
    | --- | --- |
    | <span data-ttu-id="2e1de-181">檔名</span><span class="sxs-lookup"><span data-stu-id="2e1de-181">filename</span></span> | <span data-ttu-id="2e1de-182">image1.jpg</span><span class="sxs-lookup"><span data-stu-id="2e1de-182">image1.jpg</span></span> |

1. <span data-ttu-id="2e1de-183">按一下 [測試] 面板中的 [執行] 將 HTTP 要求傳送給函式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-183">Click **Run** in the test panel to send an HTTP request to the function.</span></span>

1. <span data-ttu-id="2e1de-184">函式會在輸出中傳回上傳 URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-184">The function returns an upload URL in the output.</span></span> <span data-ttu-id="2e1de-185">[記錄] 面板中會出現函式執行情形。</span><span class="sxs-lookup"><span data-stu-id="2e1de-185">The function execution appears in the logs panel.</span></span>

    ![顯示函式已成功執行的記錄](media/functions-first-serverless-web-app/2-test-function.png)


## <a name="configure-cors-in-the-function-app"></a><span data-ttu-id="2e1de-187">在函式應用程式中設定 CORS</span><span class="sxs-lookup"><span data-stu-id="2e1de-187">Configure CORS in the function app</span></span>

<span data-ttu-id="2e1de-188">因為應用程式的前端裝載在 Blob 儲存體中，所以其網域名稱不同於 Azure 函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-188">Because the app's frontend is hosted in Blob storage, it has a different domain name than the Azure Function app.</span></span> <span data-ttu-id="2e1de-189">若要讓用戶端 JavaScript 能夠成功呼叫您剛才建立的函式，函式應用程式必須設定跨原始資源共用 (CORS)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-189">For the client-side JavaScript to successfully call the function you just created, the function app needs to be configured for cross-origin resource sharing (CORS).</span></span>

1. <span data-ttu-id="2e1de-190">在函式應用程式視窗的左側導覽列中，按一下函式應用程式的名稱。</span><span class="sxs-lookup"><span data-stu-id="2e1de-190">In the left navigation bar of the function app window, click on the name of your function app.</span></span>

1. <span data-ttu-id="2e1de-191">按一下 [平台功能] 以檢視進階功能清單。</span><span class="sxs-lookup"><span data-stu-id="2e1de-191">Click on **Platform features** to view a list of advanced features.</span></span>

1. <span data-ttu-id="2e1de-192">在 [API] 下，按一下 [CORS]。</span><span class="sxs-lookup"><span data-stu-id="2e1de-192">Under **API**, click **CORS**.</span></span>

    ![選取 CORS](media/functions-first-serverless-web-app/2-open-cors.jpg)

1. <span data-ttu-id="2e1de-194">針對上一個單元的應用程式 URL 新增允許來源，並省略尾端 **/** (例如：`https://firstserverlessweb.z4.web.core.windows.net`)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-194">Add an allow origin for the application URL from the previous module, omitting the trailing **/** (for example: `https://firstserverlessweb.z4.web.core.windows.net`).</span></span>

    ![顯示已新增無伺服器 Web 應用程式 URL 的 CORS 設定](media/functions-first-serverless-web-app/2-add-cors.png)

1. <span data-ttu-id="2e1de-196">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="2e1de-196">Click **Save**.</span></span>

1. <span data-ttu-id="2e1de-197">C#</span><span class="sxs-lookup"><span data-stu-id="2e1de-197">C#</span></span>

   1. <span data-ttu-id="2e1de-198">(C#) 瀏覽回到 `GetUploadUrl` 函式，然後選取 [整合] 索引標籤。</span><span class="sxs-lookup"><span data-stu-id="2e1de-198">(C#) Navigate back to the `GetUploadUrl` function, and then select the **Integrate** tab.</span></span>

   1. <span data-ttu-id="2e1de-199">(C#) 在 [選取的 HTTP 方法] 下，選取 [OPTIONS]。</span><span class="sxs-lookup"><span data-stu-id="2e1de-199">(C#) Under Selected HTTP methods, select **OPTIONS**.</span></span>

      <span data-ttu-id="2e1de-200">[GET]、[POST] 和 [OPTIONS] 全都應該選取。</span><span class="sxs-lookup"><span data-stu-id="2e1de-200">**GET**, **POST**, and **OPTIONS** should all be selected.</span></span> <span data-ttu-id="2e1de-201">CORS 會使用 OPTIONS 方法，若為 C# 函式，其預設並不會選取此方法。</span><span class="sxs-lookup"><span data-stu-id="2e1de-201">CORS uses the OPTIONS method, which is not selected by default for C# functions.</span></span>  

   1. <span data-ttu-id="2e1de-202">(C#) 按一下 [儲存]。</span><span class="sxs-lookup"><span data-stu-id="2e1de-202">(C#) Click **Save**.</span></span>

1. <span data-ttu-id="2e1de-203">仍在入口網站中，瀏覽至函式應用程式，選取 [概觀] 索引標籤，然後按一下 [重新啟動] 以確保 CORS 的變更會生效。</span><span class="sxs-lookup"><span data-stu-id="2e1de-203">Still in the portal, navigate to the function app, select the **Overview** tab, and then click **Restart** to make sure that the changes for CORS take effect.</span></span>

## <a name="configure-cors-in-the-storage-account"></a><span data-ttu-id="2e1de-204">在儲存體帳戶中設定 CORS</span><span class="sxs-lookup"><span data-stu-id="2e1de-204">Configure CORS in the Storage account</span></span>

<span data-ttu-id="2e1de-205">因為應用程式也會對 Blob 儲存體發出用戶端 JavaScript 呼叫以上傳檔案，因此您也必須設定儲存體帳戶的 CORS。</span><span class="sxs-lookup"><span data-stu-id="2e1de-205">Because the app also makes client-side JavaScript calls to Blob Storage to upload files, you also have to configure the Storage account for CORS.</span></span>

1. <span data-ttu-id="2e1de-206">執行下列命令，以允許所有來源將檔案上傳至儲存體帳戶。</span><span class="sxs-lookup"><span data-stu-id="2e1de-206">Run the following command to allow all origins to upload files to the Storage account.</span></span>

    ```azurecli
    az storage cors add --methods OPTIONS PUT --origins '*' --exposed-headers '*' --allowed-headers '*' --services b --account-name <storage account name>
    ```


## <a name="modify-the-web-app-to-upload-images"></a><span data-ttu-id="2e1de-207">修改 Web 應用程式以上傳影像</span><span class="sxs-lookup"><span data-stu-id="2e1de-207">Modify the web app to upload images</span></span>

<span data-ttu-id="2e1de-208">Web 應用程式會從名為 **settings.js** 的檔案中擷取設定。</span><span class="sxs-lookup"><span data-stu-id="2e1de-208">The web app retrieves settings from a file named **settings.js**.</span></span> <span data-ttu-id="2e1de-209">在下列步驟中，您會使用 Cloud Shell 建立檔案，然後將 `window.apiBaseUrl` 設定為函式應用程式的 URL，以及將 `window.blobBaseUrl` 設定為 Azure Blob 儲存體端點的 URL。</span><span class="sxs-lookup"><span data-stu-id="2e1de-209">In the following steps, you create the file using Cloud Shell, then set `window.apiBaseUrl` to the URL of the Function app and `window.blobBaseUrl` to the URL of the Azure Blob Storage endpoint.</span></span>

1. <span data-ttu-id="2e1de-210">在 Cloud Shell 中，確定目前的目錄為 **www/dist** 資料夾。</span><span class="sxs-lookup"><span data-stu-id="2e1de-210">In the Cloud Shell, ensure that the current directory is the **www/dist** folder.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. <span data-ttu-id="2e1de-211">查詢函式應用程式的 URL，並將它儲存在名為 **FUNCTION_APP_URL** 的 Bash 變數中。</span><span class="sxs-lookup"><span data-stu-id="2e1de-211">Query the function app's URL and store it in a bash variable named **FUNCTION_APP_URL**.</span></span>

    ```azurecli
    export FUNCTION_APP_URL="https://"$(az functionapp show -n <function app name> -g first-serverless-app --query "defaultHostName" --output tsv)
    ```

    <span data-ttu-id="2e1de-212">請確認您已正確設定好變數。</span><span class="sxs-lookup"><span data-stu-id="2e1de-212">Confirm the variable is correctly set.</span></span>

    ```azurecli
    echo $FUNCTION_APP_URL
    ```

1. <span data-ttu-id="2e1de-213">若要將 API 呼叫的基底 URI 設定為函式應用程式，請建立 **settings.js** 並新增函式應用程式 URL，如下所示。</span><span class="sxs-lookup"><span data-stu-id="2e1de-213">To set the base URI of API calls to your function app, create **settings.js** and add the function app URL like the following.</span></span>

    `window.apiBaseUrl = 'https://fnapp@lab.GlobalLabInstanceId.azurewebsites.net'`

    <span data-ttu-id="2e1de-214">您可以藉由執行下列命令或使用命令列編輯器 (例如 VIM) 來進行變更。</span><span class="sxs-lookup"><span data-stu-id="2e1de-214">You can make the change by running the following command or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.apiBaseUrl = '$FUNCTION_APP_URL'" > settings.js
    ```

    <span data-ttu-id="2e1de-215">請確認您已成功寫入檔案。</span><span class="sxs-lookup"><span data-stu-id="2e1de-215">Confirm the file was successfully written.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="2e1de-216">查詢 Blob 儲存體的基底 URL，並將它儲存在名為 **BLOB_BASE_URL** 的 Bash 變數中。</span><span class="sxs-lookup"><span data-stu-id="2e1de-216">Query the Blob Storage base URL and store it in a bash variable named **BLOB_BASE_URL**.</span></span>

    ```azurecli
    export BLOB_BASE_URL=$(az storage account show -n <storage account name> -g first-serverless-app --query primaryEndpoints.blob -o tsv | sed 's/\/$//')
    ```

    <span data-ttu-id="2e1de-217">請確認您已正確設定好變數。</span><span class="sxs-lookup"><span data-stu-id="2e1de-217">Confirm the variable is correctly set.</span></span>

    ```azurecli
    echo $BLOB_BASE_URL
    ```

1. <span data-ttu-id="2e1de-218">若要將 API 呼叫的基底 URI 設定為函式應用程式，請將儲存體 URL (例如下列程式碼) 附加到 **settings.js**。</span><span class="sxs-lookup"><span data-stu-id="2e1de-218">To set the base URI of API calls to your function app, append the storage URL like the following line of code to **settings.js**.</span></span>

    `window.blobBaseUrl = 'https://mystorage.blob.core.windows.net'`

    <span data-ttu-id="2e1de-219">您可以藉由執行下列命令或使用命令列編輯器 (例如 VIM) 來進行變更。</span><span class="sxs-lookup"><span data-stu-id="2e1de-219">You can make the change by running the following command or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.blobBaseUrl = '$BLOB_BASE_URL'" >> settings.js
    ```

    <span data-ttu-id="2e1de-220">請確認您已成功寫入檔案，且檔案內現在有 2 行。</span><span class="sxs-lookup"><span data-stu-id="2e1de-220">Confirm the file was successfully written and it now contains 2 lines.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="2e1de-221">將檔案上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="2e1de-221">Upload the file to Blob storage.</span></span>

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-web-application"></a><span data-ttu-id="2e1de-222">測試 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="2e1de-222">Test the web application</span></span>

<span data-ttu-id="2e1de-223">此時，影像中心應用程式可將影像上傳至 Blob 儲存體，但還無法顯示影像。</span><span class="sxs-lookup"><span data-stu-id="2e1de-223">At this point, the gallery application is able to upload an image to Blob storage, but it can't display images yet.</span></span> <span data-ttu-id="2e1de-224">它會嘗試呼叫 `GetImages` 函式，但因為後面的單元才會建立，所以此函式尚不存在。</span><span class="sxs-lookup"><span data-stu-id="2e1de-224">It will try to call a `GetImages` function that doesn't exist yet because you create it in a later module.</span></span> <span data-ttu-id="2e1de-225">該呼叫會失敗，網頁看起來會卡在「分析中...」，但您所選取的影像會成功上傳。</span><span class="sxs-lookup"><span data-stu-id="2e1de-225">That call will fail, and the web page will appear to be stuck on "Analyzing...", but the image you select will be successfully uploaded.</span></span>

<span data-ttu-id="2e1de-226">您可以在 Azure 入口網站中檢查 [images] 容器的內容，來確認影像是否已成功上傳。</span><span class="sxs-lookup"><span data-stu-id="2e1de-226">You can verify an image is successfully uploaded by checking the contents of the **images** container in the Azure portal.</span></span>

1. <span data-ttu-id="2e1de-227">在瀏覽器視窗中，瀏覽至應用程式。</span><span class="sxs-lookup"><span data-stu-id="2e1de-227">In a browser window, browse to the application.</span></span> <span data-ttu-id="2e1de-228">選取影像檔，並將它上傳。</span><span class="sxs-lookup"><span data-stu-id="2e1de-228">Select an image file and upload it.</span></span> <span data-ttu-id="2e1de-229">上傳完成，但因為我們尚未新增顯示影像的功能，應用程式不會顯示已上傳的影像。</span><span class="sxs-lookup"><span data-stu-id="2e1de-229">The upload completes, but because we haven't added the ability to display images yet, the app doesn't show the uploaded photo.</span></span> <span data-ttu-id="2e1de-230">(網頁看起來會卡在「正在分析影像...」；您會在後面修正該問題)。</span><span class="sxs-lookup"><span data-stu-id="2e1de-230">(The web page appears to be stuck on "Analyzing image..."; you'll fix that later.)</span></span>

1. <span data-ttu-id="2e1de-231">在 Cloud Shell 中，確認影像已上傳到 [images] 容器。</span><span class="sxs-lookup"><span data-stu-id="2e1de-231">In the Cloud Shell, confirm the image was uploaded to the **images** container.</span></span>

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. <span data-ttu-id="2e1de-232">在繼續進行下一個教學課程之前，請先刪除 [images] 容器中的所有檔案。</span><span class="sxs-lookup"><span data-stu-id="2e1de-232">Before moving on to the next tutorial, delete all files in the **images** container.</span></span>

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```


## <a name="summary"></a><span data-ttu-id="2e1de-233">總結</span><span class="sxs-lookup"><span data-stu-id="2e1de-233">Summary</span></span>

<span data-ttu-id="2e1de-234">在本單元中，您已建立 Azure 函式應用程式，並了解如何使用無伺服器函式來允許 Web 應用程式將影像上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="2e1de-234">In this module, you created an Azure Function app and learned how to use a serverless function to allow a web application to upload images to Blob storage.</span></span> <span data-ttu-id="2e1de-235">接下來，您將會了解如何使用由 Blob 觸發的無伺服器函式，來為所上傳的影像建立縮圖。</span><span class="sxs-lookup"><span data-stu-id="2e1de-235">Next, you learn how to create thumbnails for the uploaded images using a Blob triggered serverless function.</span></span>
