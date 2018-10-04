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
ms.openlocfilehash: f51b864cab14273c1e88dd85d22400e0e76ef770
ms.sourcegitcommit: 81587470a181e314242c7a97cd0f91c82d4fe232
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47459988"
---
到目前為止，此應用程式已是一個可運作的影像中心，可讓您上傳和檢視影像。 在本單元中，您會了解如何使用 Microsoft 認知服務的電腦視覺 API，為所上傳的影像產生標題，並使用影像中繼資料將標題儲存在 Cosmos DB 中。

## <a name="create-a-computer-vision-account"></a>建立電腦視覺帳戶

Microsoft 認知服務是一組可供開發人員使用的服務，可讓其應用程式更有智慧。 電腦視覺 API 是一種無伺服器服務，會使用進階演算法處理影像，並傳回每個影像的相關資訊。 它有免費層可提供每月最多 5000 個 API 呼叫。

1. 確定您仍登入 Cloud Shell。 如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。 

1. 使用唯一名稱在資源群組中建立 **ComputerVision** 種類的新認知服務帳戶。 在免費層中，請使用 **F0** 作為 SKU。 如果您已有既有的電腦視覺帳戶，則可能需要建立標準帳戶 (S1)；不過，這麼做可能會產生一些成本。

    ```azurecli
    az cognitiveservices account create -g first-serverless-app -n <computer vision account name> --kind ComputerVision --sku F0 -l westcentralus
    ```


## <a name="create-function-app-settings-for-computer-vision-url-and-key"></a>建立函式應用程式設定以取得電腦視覺 URL 和金鑰

若要呼叫電腦視覺 API，您需要有 URL 和金鑰。

1. 請取得電腦視覺 API 金鑰和 URL，並將其儲存在 Bash 變數中。

    ```azurecli
    export COMP_VISION_KEY=$(az cognitiveservices account keys list -g first-serverless-app -n <computer vision account name> --query key1 --output tsv)
    ```
    ```azurecli
    export COMP_VISION_URL=$(az cognitiveservices account show -g first-serverless-app -n <computer vision account name> --query endpoint --output tsv)
    ```

1. 在函式應用程式中分別建立名為 **COMP_VISION_KEY** 和 **COMP_VISION_URL** 的應用程式設定。

    ```azurecli
    az functionapp config appsettings set -n <function app name> -g first-serverless-app --settings COMP_VISION_KEY=$COMP_VISION_KEY COMP_VISION_URL=$COMP_VISION_URL -o table
    ```


## <a name="call-computer-vision-api-from-resizeimage-function"></a>從 ResizeImage 函式呼叫電腦視覺 API

在下列步驟中，您會修改 **ResizeImage** 函式來呼叫電腦視覺 API，以描述每個所上傳的影像，並將描述儲存在 Cosmos DB 中。

1. 在 Azure 入口網站中開啟函式應用程式。

1. **C#**

    1. (C#) 使用左側導覽找出 **ResizeImage** 函式，並開啟其程式碼視窗。

    1. (C#) 將程式碼更換為 [**/csharp/ResizeImage/run-module5.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run-module5.csx) 的內容。 此程式碼使用 `HttpClient` 來呼叫電腦視覺 API，並會將其結果儲存在 Cosmos DB 中。

1. **JavaScript**

    1. (JavaScript) 此函式需要 npm 的 `axios` 套件，才能對電腦視覺 API 發出 HTTP 呼叫。 若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。

    1. (JavaScript) 按一下 [主控台] 以顯示主控台視窗。

    1. (JavaScript) 在主控台中執行 `npm install axios` 命令。 此作業可能需要一兩分鐘的時間才能完成。

    1. (JavaScript) 在左側導覽中按一下函式名稱 (**ResizeImage**) 來顯示函式，然後將 **index.js** 全部更換為 [**/javascript/ResizeImage/index-module5.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index-module5.js) 的內容。

1. 按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。

1. 按一下 [檔案] 。 檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。


## <a name="test-the-application"></a>測試應用程式

1. 在瀏覽器中開啟應用程式。 選取影像檔，並將它上傳。

1. 幾秒後，新影像的縮圖應該就會出現在頁面上。 將滑鼠停留在影像上，以查看電腦願景所產生的描述。


## <a name="summary"></a>總結

在本單元中，您已了解如何使用 Microsoft 認知服務的電腦視覺 API，為每個已上傳的影像自動產生標題。 接下來，您將會了解如何使用 Azure App Service 驗證，對應用程式新增驗證。
