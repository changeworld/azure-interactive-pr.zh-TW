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
Azure Cosmos DB 是 Microsoft 全球發行的多模型無伺服器資料庫。 在本單元中，您會了解如何使用 Azure Functions，在 Cosmos DB 中儲存和擷取 JSON 文件形式的影像中繼資料。

## <a name="create-a-cosmos-db-account-database-and-collection"></a>建立 Cosmos DB 帳戶、資料庫和集合

Cosmos DB 帳戶是 Azure 資源，內含 Cosmos DB 資料庫。

1. 確定您仍登入 Cloud Shell。  如果沒有，請選取 [進入焦點模式] 以開啟 Cloud Shell 視窗。 

1. 在與本教學課程中其他資源相同的資源群組內，使用唯一名稱建立 Cosmos DB 帳戶。

    ```azurecli
    az cosmosdb create -g first-serverless-app -n <cosmos db account name>
    ```

1. 建立 Cosmos DB 帳戶之後，於該帳戶內建立名為 **imagesdb** 的新資料庫。

    ```azurecli
    az cosmosdb database create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb
    ```

1. 建立資料庫之後，於資料庫內建立名為 **images**，且輸送量為 400 個要求單位 (RU) 的新集合。

    ```azurecli
    az cosmosdb collection create -g first-serverless-app -n <cosmos db account name> --db-name imagesdb --collection-name images --throughput 400
    ```


## <a name="save-a-document-to-cosmos-db-when-a-thumbnail-is-created"></a>在建立縮圖時將文件儲存至 Cosmos DB

Cosmos DB 輸出繫結可讓您從 Azure Functions 在 Cosmos DB 集合中建立文件。 在下列步驟中，您會在 [ResizeImage] 函式中設定 Cosmos DB 輸出繫結，並修改函式以傳回要儲存的文件 (物件)。

1. 在 Azure 入口網站中開啟函式應用程式。

1. 在左側導覽中，展開 [ResizeImage] 函式，然後選取 [整合]。

1. 在 [輸出] 下，按一下 [新增輸出]。

1. 尋找 [Azure Cosmos DB] 項目，然後加以選取。 然後按一下 [選取] 。

    ![選取 [新增輸出]](media/functions-first-serverless-web-app/4-new-output.jpg)

1. 使用下列值填寫 [Azure Cosmos DB 輸出] 下方的欄位。

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **文件參數名稱** | 選取 [使用函式傳回值] | 文字方塊的值會自動設定為 [$return]。 |
    | **資料庫名稱** | imagesdb | 使用所建立資料庫的名稱。 |
    | **集合名稱** | images | 使用所建立集合的名稱。 |

1. 按一下 [Azure Cosmos DB 帳戶連線] 旁的 [新增]。 選取您先前建立的 Cosmos DB 帳戶。

    ![輸入 Azure Cosmos DB 輸出繫結的設定](media/functions-first-serverless-web-app/4-cosmos-db-output.png)

1. 按一下 [儲存] 以建立 Cosmos DB 輸出繫結。

1. 按一下左側的 [ResizeImage] 函式名稱來開啟該函式。

1. **C#**

    1. (C#) 將函式的傳回型別從 **void** 變更為 **object**。

    1. (C#) 在函式結尾新增下列程式碼區塊，以傳回要儲存的文件：
    
        ```csharp
        return new {
            id = name,
            imgPath = "/images/" + name,
            thumbnailPath = "/thumbnails/" + name
        };
        ```
    
        ![ResizeImages 函式在修改後的 run.csx](media/functions-first-serverless-web-app/4-update-function.png)

1. **JavaScript**

    1. (JavaScript) 變更 `else` 子句中的 `context.done()` 陳述式，以傳回要儲存到 Cosmos DB 的文件。

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

1. 按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。

1. 按一下 [檔案] 。 檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。


## <a name="create-a-function-to-list-images-from-cosmos-db"></a>建立函式以列出 Cosmos DB 中的影像

Web 應用程式需要 API，以擷取 Cosmos DB 中的影像中繼資料。 在下列步驟中。 您會建立由 HTTP 觸發的函式，以使用 Cosmos DB 輸入繫結來查詢資料庫集合。

1. 在函式應用程式中，將滑鼠停留在左邊的 [函式]，然後按一下 **+** 來建立新函式。

1. 尋找 **HttpTrigger** 樣板並加以選取。

1. 使用這些值來建立函式以產生取得影像 URL。

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **函式命名** | GetImages | 輸入這個與所示名稱完全相同的名稱，讓應用程式可以探索到此函式。 |
    | **授權層級** | 匿名 | 允許公開存取函式。 |

1. 按一下頁面底部的 [新增] 。

1. 在新函式建立時，按一下左側導覽上函式名稱底下的 [整合]。

1. 按一下 [新增輸入]，然後選取 [Azure Cosmos DB]。 

    ![選取 [新增輸入]](media/functions-first-serverless-web-app/4-new-input.jpg)

1. 按一下 [選取] 。

1. 填寫下列值：

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **文件參數名稱** | documents | 符合函式中的參數名稱。 |
    | **資料庫名稱** | imagesdb |  |
    | **集合名稱** | images |  |
    | **SQL query** | select * from c order by c._ts desc | 取得文件，最新的文件優先取得。 |
    | **Azure Cosmos DB 帳戶連線** | 選取現有連接字串 |  |

1. 按一下 [儲存] 來建立輸入繫結。

1. **C#**

    1. 按一下函式的名稱以開啟程式碼視窗，然後將 **run.csx** 全部更換為 [**/csharp/GetImages/run.csx**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/csharp/GetImages/run.csx) 中的內容。

1. **JavaScript**

    1. 按一下函式的名稱以開啟程式碼視窗，然後將 **index.js** 全部更換為 [**/javascript/GetImages/index.js**](https://raw.githubusercontent.com/Azure-Samples/functions-first-serverless-web-application/master/javascript/GetImages/index.js) 中的內容。

1. 按一下程式碼視窗下方的 [記錄]，以展開 [記錄] 面板。

1. 按一下 [檔案] 。 檢查 [記錄] 面板，以確定函式已儲存成功，且沒有任何錯誤。


## <a name="test-the-application"></a>測試應用程式

1. 在瀏覽器中開啟應用程式。 選取影像檔，並將它上傳。

1. 幾秒後，新影像的縮圖便會出現在頁面上。

1. 在 Azure 入口網站中，使用 [搜尋] 方塊來依名稱搜尋 Cosmos DB 帳戶。 按一下該帳戶加以開啟。

1. 按一下左側的 [資料總管] 來瀏覽集合和文件。

1. 在 [imagesdb] 資料庫底下，選取 [images] 集合。

1. 確認已針對上傳的影像建立文件。

    ![資料總管，會顯示所上傳影像的文件](media/functions-first-serverless-web-app/4-data-explorer.png)



## <a name="summary"></a>總結

在本單元中，您已了解如何建立 Cosmos DB 帳戶、資料庫和集合。 您也已經了解如何使用 Cosmos DB 繫結在 Cosmos DB 集合中儲存和擷取影像中繼資料。 接下來，您將會了解如何使用 Microsoft 認知服務為每個已上傳的影像自動產生標題。
