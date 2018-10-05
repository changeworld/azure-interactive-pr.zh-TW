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
ms.openlocfilehash: 51c7d3e64424d499b473f3b138ce249a9cfd0182
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47460073"
---
您所要建置的應用程式是影像中心。 它會使用用戶端 JavaScript 來呼叫 API，以便上傳和顯示影像。 在本單元中，您會使用無伺服器函式建立 API，以產生有時間限制的 URL 來上傳影像。 Web 應用程式會使用所產生的 URL，透過 [Blob 儲存體 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) 將影像上傳至 Blob 儲存體。

## <a name="create-a-blob-storage-container-for-images"></a>建立供影像使用的 Blob 儲存體容器

應用程式需要不同的儲存體容器來上傳和裝載影像。

1. 確定您仍登入 Cloud Shell (bash)。 如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。

1.  在儲存體帳戶中，使用可存取所有 Blob 的公用權限建立名為 **images** 的新容器。

    ```azurecli
    az storage container create -n images --account-name <storage account name> --public-access blob
    ```

## <a name="create-an-azure-function-app"></a>建立 Azure 函數應用程式

Azure Functions 是用於執行無伺服器函式的服務。 無伺服器函式可由事件 (例如，HTTP 要求) 或在儲存體容器中建立 Blob 時觸發 (呼叫)。

Azure 函式應用程式是供一或多個無伺服器函式使用的容器。

在先前所建立名為 **first-serverless-app** 的資源群組中，使用唯一名稱建立新的 Azure 函式應用程式。 函式應用程式需要儲存體帳戶；在本教學課程中，您會使用現有儲存體帳戶。

```azurecli
az functionapp create -n <function app name> -g first-serverless-app -s <storage account name> -c westcentralus
```

## <a name="configure-the-function-app"></a>設定函式應用程式

本教學課程中的函式應用程式需要使用版本 1.x 的 Azure Functions 執行階段。 將 `FUNCTIONS_WORKER_RUNTIME` 應用程式設定設為 `~1`，以將函式應用程式釘選至最新的 1.x 版本。 使用 [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) 命令設定應用程式設定。

在下列 Azure CLI 命令中，`<app_name> 是函式應用程式的名稱。

```azurecli
az functionapp config appsettings set --name <function app name> --g first-serverless-app --settings FUNCTIONS_WORKER_RUNTIME=~1
```

## <a name="create-an-http-triggered-serverless-function"></a>建立由 HTTP 觸發的無伺服器函式

影像中心 Web 應用程式會對無伺服器函式發出 HTTP 要求，以產生有時間限制的 URL 將影像安全地上傳至 Blob 儲存體中。 此函式會由 HTTP 要求來觸發，並使用 Azure 儲存體 SDK 來產生及傳回安全的 URL。

1. 建立函式應用程式之後，請在 Azure 入口網站中使用 [搜尋] 方塊搜尋該應用程式，並按一下來加以開啟。

    ![開啟函式應用程式](media/functions-first-serverless-web-app/2-search-function-app.png)

1. 在函式應用程式視窗的左側導覽中，將滑鼠停留在 [函式] 上，然後按一下 **+** 以開始建立新的無伺服器函式。

    ![建立新函式](media/functions-first-serverless-web-app/2-new-function.png)

1. 按一下 [自訂函式] 以查看函式樣板清單。

1. 尋找 **HttpTrigger** 樣板，然後按一下要使用的語言 (C# 或 JavaScript)。

1. 使用這些值來建立函式以產生 Blob 上傳 URL。

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **語言** | C# 或 JavaScript | 選取您想要使用的語言。 |
    | **函式命名** | GetUploadUrl | 輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。 |
    | **授權層級** | 匿名 | 允許公開存取函式。 |

    ![針對由 HTTP 觸發的新函式輸入設定](media/functions-first-serverless-web-app/2-new-function-httptrigger.png)

1. 按一下 [建立] 以建立函式。

1. **C#** 

    1. 當函式的原始程式碼出現時，將 **run.csx** 全部更換為 [**csharp/GetUploadUrl/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetUploadUrl/run.csx) 的內容。

1. **JavaScript** 

    1. (JavaScript) 此函式需要 npm 的 `azure-storage` 套件，以產生所需的共用存取簽章 (SAS) 權杖來建立安全 URL。 若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。

    1. (JavaScript) 按一下 [主控台] 以顯示主控台視窗。

        ![開啟主控台視窗](media/functions-first-serverless-web-app/2-open-console.jpg)

    1. (JavaScript) 執行 `cd d:\home\site\wwwroot` 命令，確定目前的目錄為 **d:\home\site\wwwroot**。

    1. (JavaScript) 執行 `npm init -y` 命令以建立空白 **package.json** 檔案。

    1. (JavaScript) 在主控台執行 `npm install --save azure-storage` 命令以安裝套件，並將它儲存在 **package.json**。 此作業可能需要一兩分鐘的時間才能完成。

    1. (JavaScript) 在左側導覽中按一下函式名稱 (**GetUploadUrl**) 來顯示函式，並將 **index.js** 全部更換為 [**javascript/GetUploadUrl /index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetUploadUrl/index.js) 的內容。
    
        ![更新後的 index.js](media/functions-first-serverless-web-app/2-paste-js.jpg)

1. 按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。

1. 按一下 [檔案] 。 檢查 [記錄] 面板，以確定函式已成功完成編譯。

函式會產生所謂的共用存取簽章 (SAS) URL，以便用來將檔案上傳至 Blob 儲存體。 SAS URL 的有效時間很短，且只允許上傳一個檔案。 如需如何[使用共用存取簽章](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)的詳細資訊，請參閱 Blob 儲存體文。


## <a name="add-an-environment-variable-for-the-storage-connection-string"></a>針對儲存體連接字串新增環境變數

您剛才建立的函式需要儲存體帳戶的連接字串，才能產生 SAS URL。 您不一定要將連接字串硬式編碼到函式主體中，也可以將它儲存為應用程式設定。 應用程式設定可作為環境變數供函式應用程式中的所有函式存取。

1. 在 Cloud Shell 中查詢儲存體帳戶連接字串，並將它儲存到名為**STORAGE_CONNECTION_STRING** 的 Bash 變數。

    ```azurecli
    export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n <storage account name> -g first-serverless-app --query "connectionString" --output tsv)
    ```

    請確認您已成功設定好變數。

    ```azurecli
    echo $STORAGE_CONNECTION_STRING
    ```

1. 使用上一個步驟所儲存的值，建立名為 **AZURE_STORAGE_CONNECTION_STRING** 的新應用程式設定。

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING -o table
    ```

    請確認命令輸出中所包含的新應用程式設定有正確的值。


## <a name="test-the-serverless-function"></a>測試無伺服器函式

除了建立和編輯函式，Azure 入口網站也提供內建的函式測試工具。

1. 若要測試 HTTP 無伺服器函式，請按一下程式碼視窗右側的 [測試] 索引標籤，以展開 [測試] 面板。

1. 將 [HTTP 方法] 變更為 [GET]。

1. 在 [查詢] 之下，按一下 [新增參數] 並新增下列參數：

    | name      |  value   | 
    | --- | --- |
    | 檔名 | image1.jpg |

1. 按一下 [測試] 面板中的 [執行] 將 HTTP 要求傳送給函式。

1. 函式會在輸出中傳回上傳 URL。 [記錄] 面板中會出現函式執行情形。

    ![顯示函式已成功執行的記錄](media/functions-first-serverless-web-app/2-test-function.png)


## <a name="configure-cors-in-the-function-app"></a>在函式應用程式中設定 CORS

因為應用程式的前端裝載在 Blob 儲存體中，所以其網域名稱不同於 Azure 函式應用程式。 若要讓用戶端 JavaScript 能夠成功呼叫您剛才建立的函式，函式應用程式必須設定跨原始資源共用 (CORS)。

1. 在函式應用程式視窗的左側導覽列中，按一下函式應用程式的名稱。

1. 按一下 [平台功能] 以檢視進階功能清單。

1. 在 [API] 下，按一下 [CORS]。

    ![選取 CORS](media/functions-first-serverless-web-app/2-open-cors.jpg)

1. 針對上一個單元的應用程式 URL 新增允許來源，並省略尾端 **/** (例如：`https://firstserverlessweb.z4.web.core.windows.net`)。

    ![顯示已新增無伺服器 Web 應用程式 URL 的 CORS 設定](media/functions-first-serverless-web-app/2-add-cors.png)

1. 按一下 [檔案] 。

1. C#

   1. (C#) 瀏覽回到 `GetUploadUrl` 函式，然後選取 [整合] 索引標籤。

   1. (C#) 在 [選取的 HTTP 方法] 下，選取 [OPTIONS]。

      [GET]、[POST] 和 [OPTIONS] 全都應該選取。 CORS 會使用 OPTIONS 方法，若為 C# 函式，其預設並不會選取此方法。  

   1. (C#) 按一下 [儲存]。

1. 仍在入口網站中，瀏覽至函式應用程式，選取 [概觀] 索引標籤，然後按一下 [重新啟動] 以確保 CORS 的變更會生效。

## <a name="configure-cors-in-the-storage-account"></a>在儲存體帳戶中設定 CORS

因為應用程式也會對 Blob 儲存體發出用戶端 JavaScript 呼叫以上傳檔案，因此您也必須設定儲存體帳戶的 CORS。

1. 執行下列命令，以允許所有來源將檔案上傳至儲存體帳戶。

    ```azurecli
    az storage cors add --methods OPTIONS PUT --origins '*' --exposed-headers '*' --allowed-headers '*' --services b --account-name <storage account name>
    ```


## <a name="modify-the-web-app-to-upload-images"></a>修改 Web 應用程式以上傳影像

Web 應用程式會從名為 **settings.js** 的檔案中擷取設定。 在下列步驟中，您會使用 Cloud Shell 建立檔案，然後將 `window.apiBaseUrl` 設定為函式應用程式的 URL，以及將 `window.blobBaseUrl` 設定為 Azure Blob 儲存體端點的 URL。

1. 在 Cloud Shell 中，確定目前的目錄為 **www/dist** 資料夾。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. 查詢函式應用程式的 URL，並將它儲存在名為 **FUNCTION_APP_URL** 的 Bash 變數中。

    ```azurecli
    export FUNCTION_APP_URL="https://"$(az functionapp show -n <function app name> -g first-serverless-app --query "defaultHostName" --output tsv)
    ```

    請確認您已正確設定好變數。

    ```azurecli
    echo $FUNCTION_APP_URL
    ```

1. 若要將 API 呼叫的基底 URI 設定為函式應用程式，請建立 **settings.js** 並新增函式應用程式 URL，如下所示。

    `window.apiBaseUrl = 'https://fnapp@lab.GlobalLabInstanceId.azurewebsites.net'`

    您可以藉由執行下列命令或使用命令列編輯器 (例如 VIM) 來進行變更。

    ```azurecli
    echo "window.apiBaseUrl = '$FUNCTION_APP_URL'" > settings.js
    ```

    請確認您已成功寫入檔案。

    ```azurecli
    cat settings.js
    ```

1. 查詢 Blob 儲存體的基底 URL，並將它儲存在名為 **BLOB_BASE_URL** 的 Bash 變數中。

    ```azurecli
    export BLOB_BASE_URL=$(az storage account show -n <storage account name> -g first-serverless-app --query primaryEndpoints.blob -o tsv | sed 's/\/$//')
    ```

    請確認您已正確設定好變數。

    ```azurecli
    echo $BLOB_BASE_URL
    ```

1. 若要將 API 呼叫的基底 URI 設定為函式應用程式，請將儲存體 URL (例如下列程式碼) 附加到 **settings.js**。

    `window.blobBaseUrl = 'https://mystorage.blob.core.windows.net'`

    您可以藉由執行下列命令或使用命令列編輯器 (例如 VIM) 來進行變更。

    ```azurecli
    echo "window.blobBaseUrl = '$BLOB_BASE_URL'" >> settings.js
    ```

    請確認您已成功寫入檔案，且檔案內現在有 2 行。

    ```azurecli
    cat settings.js
    ```

1. 將檔案上傳至 Blob 儲存體。

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-web-application"></a>測試 Web 應用程式

此時，影像中心應用程式可將影像上傳至 Blob 儲存體，但還無法顯示影像。 它會嘗試呼叫 `GetImages` 函式，但因為後面的單元才會建立，所以此函式尚不存在。 該呼叫會失敗，網頁看起來會卡在「分析中...」，但您所選取的影像會成功上傳。

您可以在 Azure 入口網站中檢查 [images] 容器的內容，來確認影像是否已成功上傳。

1. 在瀏覽器視窗中，瀏覽至應用程式。 選取影像檔，並將它上傳。 上傳完成，但因為我們尚未新增顯示影像的功能，應用程式不會顯示已上傳的影像。 (網頁看起來會卡在「正在分析影像...」；您會在後面修正該問題)。

1. 在 Cloud Shell 中，確認影像已上傳到 [images] 容器。

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. 在繼續進行下一個教學課程之前，請先刪除 [images] 容器中的所有檔案。

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```


## <a name="summary"></a>總結

在本單元中，您已建立 Azure 函式應用程式，並了解如何使用無伺服器函式來允許 Web 應用程式將影像上傳至 Blob 儲存體。 接下來，您將會了解如何使用由 Blob 觸發的無伺服器函式，來為所上傳的影像建立縮圖。
