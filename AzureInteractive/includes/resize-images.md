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
在上一個單元，您已看到無伺服器函式如何方便您從 Web 應用程式將影像安全地上傳到 Blob 儲存體。 在本單元中，您會建立另一個無伺服器函式，以監看所上傳的影像，並透過影像建立縮圖。

## <a name="create-a-blob-storage-container-for-thumbnails"></a>建立供縮圖使用的 Blob 儲存體容器

完整大小的影像會儲存在名為 **images** 的容器中。 您需要另一個容器來儲存這些影像的縮圖。

1. 確定您仍登入 Cloud Shell (bash)。  如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。 

1. 在儲存體帳戶中，使用可存取所有 Blob 的公用權限建立名為 **thumbnails** 的新容器。

    ```azurecli
    az storage container create -n thumbnails --account-name <storage account name> --public-access blob
    ```


## <a name="create-a-blob-triggered-serverless-function"></a>建立由 Blob 觸發的無伺服器函式

觸發程序會定義叫用函式的方式。 您接下來建立的函式會使用 Blob 觸發程序：有 Blob (影像檔) 上傳至 **images** 容器時，便會自動叫用函式。 函式必須有一個觸發程序。 觸發程序具有相關聯的資料，它通常是觸發函數的承載。

繫結會定義函式在 Azure 或第三方服務中讀取或寫入資料的方式。 此函式會針對觸發函式的影像建立其縮圖版本，並將縮圖儲存在 *thumbnails* 容器中。

1. 在 Azure 入口網站中開啟函式應用程式。

1. 在函式應用程式視窗的左側導覽中，將滑鼠停留在 [函式] 上，然後按一下 **+** 以開始建立新的無伺服器函式。 如果出現 [快速入門] 頁面，請按一下 [自訂函式] 以查看函式樣板清單。

1. 尋找 **BlobTrigger** 樣板並加以選取。

1. 使用這些值來建立函式，以在影像上傳時建立縮圖。

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **語言** | C# 或 JavaScript | 選擇慣用語言。 |
    | **函式命名** | ResizeImage | 輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。 |
    | **路徑** | images/{name} | 當 **images** 容器中出現檔案時，執行函式。 |
    | **儲存體帳戶資訊** | AZURE_STORAGE_CONNECTION_STRING | 使用先前透過連接字串所建立的環境變數。 |

    ![輸入新函式的設定](media/functions-first-serverless-web-app/3-new-function.png)

1. 按一下 [建立] 以建立函式。

1. 函式建立好時，按一下 [整合] 以檢視其觸發程序、輸入和輸出繫結。

1. 按一下 [新增輸出] 來建立新的輸出繫結。

    ![選取 [整合] 索引標籤上的 [新增輸出]](media/functions-first-serverless-web-app/3-new-output.jpg)

1. 選取 [Azure Blob 儲存體]，然後按一下 [選取]。 您可能必須向下捲動才能看到 [選取] 按鈕。

    ![選取 Azure Blob 儲存體](media/functions-first-serverless-web-app/3-storage-output.jpg)

1. 輸入下列值。

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **Blob 參數名稱** | thumbnail | 函式會搭配使用參數與此名稱來寫入縮圖。 |
    | **使用函數傳回值** | 否 |  |
    | **路徑** | thumbnails/{name} | 縮圖會寫入到名為 **thumbnails** 的容器。 |
    | **儲存體帳戶連線** | AZURE_STORAGE_CONNECTION_STRING | 使用先前透過連接字串所建立的環境變數。 |

    ![輸入 Blob 輸出繫結的設定](media/functions-first-serverless-web-app/3-blob-output.png)

1. **JavaScript**

    1. (JavaScript) 按一下視窗右上角的 [進階編輯器] 來顯示代表繫結的 JSON。

    1. (JavaScript) 在 `blobTrigger` 繫結中，新增名為 `dataType` 且值為 `binary` 的屬性。 這會將繫結設定為以二進位資料的形式將 Blob 內容傳遞給函式。

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

1. 按一下 [儲存] 來建立新的繫結。

1. **C#**

    1. (C#) 選取左側導覽上的 [ResizeImage] 函式名稱，以開啟函式的原始程式碼。

    1. (C#) 函式需要名為 **ImageResizer** 的 NuGet 套件來產生縮圖。 NuGet 套件會使用 **project.json** 檔案新增至 C# 函式。 若要建立檔案，請按一下右邊的 [檢視檔案] 以顯示構成函式的檔案。
    
    1. (C#) 按一下 [新增] 以新增名為 **project.json** 的新檔案。
    
    1. (C#) 將 [**/csharp/ResizeImage/project.json**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/project.json) 的內容複製到新建立的檔案。 儲存檔案。 檔案更新時，便會自動還原套件。
    
        ![具有 ImageResizer 的 project.json 檔案](media/functions-first-serverless-web-app/3-project-json.png)
    
    1. (C#) 選取 [檢視檔案] 下方的 [run.csx]，並將其內容更換為 [**/csharp/ResizeImage/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/ResizeImage/run.csx) 中的內容。

1. **JavaScript** 

    1. (JavaScript) 此函式需要 npm 的 `jimp` 套件才能調整影像大小。 若要安裝 npm 套件，請在左側導覽中按一下函式應用程式的名稱，然後按一下 [平台功能]。

    1. (JavaScript) 按一下 [主控台] 以顯示主控台視窗。

    1. (JavaScript) 在主控台中執行 `npm install jimp` 命令。 此作業可能需要一兩分鐘的時間才能完成。

    1. (JavaScript) 在左側導覽中按一下 [ResizeImage] 函式名稱來顯示函式，並將 **index.js** 全部更換為 [**/javascript/ResizeImage/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/ResizeImage/index.js) 的內容。

1. 按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。

1. 按一下 [檔案] 。 檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。


## <a name="test-the-serverless-function"></a>測試無伺服器函式

1. 在瀏覽器中開啟應用程式。 選取影像檔，並將它上傳。 上傳完成，但因為我們尚未新增顯示影像的功能，應用程式不會顯示已上傳的影像。

1. 在 Cloud Shell 中，確認影像已上傳到 [images] 容器。

    ```azurecli
    az storage blob list --account-name <storage account name> -c images -o table
    ```

1. 確認名為 **thumbnails** 的容器中已建立縮圖。

    ```azurecli
    az storage blob list --account-name <storage account name> -c thumbnails -o table
    ```

1. 取得縮圖的 URL。

    ```azurecli
    az storage blob url --account-name <storage account name> -c thumbnails -n <filename> --output tsv
    ```

    在瀏覽器中開啟 URL，並確認是否已正確建立縮圖。

1. 在繼續進行下一個教學課程之前，請先刪除 [images] 和 [thumbnails] 容器中的所有檔案。

    ```azurecli
    az storage blob delete-batch -s images --account-name <storage account name>
    ```
    ```azurecli
    az storage blob delete-batch -s thumbnails --account-name <storage account name>
    ```


## <a name="summary"></a>總結

在本單元中，您已建立無伺服器函式，以在每次有影像上傳至 Blob 儲存體容器時建立縮圖。 接下來，您將會了解如何使用 Azure Cosmos DB 來儲存和列出影像中繼資料。
