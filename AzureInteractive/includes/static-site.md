Azure Blob 儲存體是低成本且可大幅調整的服務，可用來裝載靜態檔案。 在本教學課程中，您會使用它來為您建置的 Web 應用程式提供靜態內容 (例如，HTML、JavaScript、CSS)。

## <a name="create-a-storage-account"></a>建立儲存體帳戶

儲存體帳戶是一種 Azure 資源，可讓您儲存資料表、佇列、檔案、Blob (物件) 與虛擬機器磁碟。

1. 選取 [進入焦點模式] 按鈕，以登入 Cloud Shell (Bash)。 視瀏覽器視窗的寬度而定，此按鈕會位於頁面右上方或底部。 焦點模式會將 Cloud Shell 視窗停駐在瀏覽器視窗右邊，讓您可以輕鬆地執行本教學課程所示的命令。

1. 在 Azure 中，資源群組是一種容器，可保存相關 Azure 資源以方便您管理。 建立名為 **first-serverless-app** 的新資源群組。

    ```azurecli
    az group create -n first-serverless-app -l westcentralus
    ```

1. 本教學課程的靜態內容 (HTML、CSS 和 JavaScript 檔案) 會裝載在 Blob 儲存體中。 Blob 儲存體需要儲存體帳戶。 請在資源群組中建立儲存體帳戶 (一般用途 V2)。 將 `<storage account name>` 更換為唯一名稱。

    ```azurecli
    az storage account create -n <storage account name> -g first-serverless-app --kind StorageV2 -l westcentralus --https-only true --sku Standard_LRS
    ```

1. 使用 [Azure 入口網站](https://portal.azure.com)頂端的 [搜尋] 列，尋找您剛才建立的儲存體帳戶並加以開啟。

1. 在左側導覽中，選取 [靜態網站 (預覽)] 來設定靜態網站託管的容器。
    - 選取 [已啟用] 來啟用靜態網站。
    - 輸入「index.html」作為索引文件名稱。 欄位中已經有灰色字型的「index.html」，但這只是文字範例；您還是必須在欄位中輸入該值。
    - 按一下 [檔案] 。
    
    ![輸入靜態網站設定](media/functions-first-serverless-web-app/1-storage-static-website.png)

1. 將 [主要端點] 儲存到某個可方便您在進行教學課程期間複製此值的地方。 這是 Web 應用程式的 URL。

## <a name="upload-the-web-application"></a>上傳 Web 應用程式

1. 您在本教學課程中所建置的應用程式原始程式檔位於 [GitHub 存放庫](https://github.com/Azure-Samples/functions-first-serverless-web-application)。 請確定您位於 Cloud Shell 的主目錄，並複製這個存放庫。

    ```azurecli
    cd ~
    git clone https://github.com/Azure-Samples/functions-first-serverless-web-application
    ```

    此存放庫會複製到 `/home/<username>/functions-first-serverless-web-application`。

1. 用戶端 Web 應用程式位於 **www** 資料夾，並且會使用 Vue.js JavaScript 架構來建置。 變更至該資料夾，然後執行 npm 命令來安裝應用程式的相依性，並建置應用程式。 這些命令的最後一個可能需要幾分鐘的時間才會完成。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www
    npm install
    npm run generate
    ```

    應用程式會在 **dist** 資料夾中產生。

1. 將目前的目錄變更至 **dist**，然後將應用程式上傳至 **$web** Blob 容器。

    ```azurecli
    cd dist
    az storage blob upload-batch -s . -d \$web --account-name <storage account name>
    ```

1. 在網頁瀏覽器中開啟儲存體靜態網站的主要端點 URL，以檢視應用程式。

    ![第一個無伺服器 Web 應用程式首頁](media/functions-first-serverless-web-app/1-app-screenshot-new.png)


## <a name="summary"></a>總結

在本單元中，您已建立名為 **first-serverless-app** 且包含儲存體帳戶的資源群組。 儲存體帳戶中名為 **$web** 的 Blob 容器，會儲存 Web 應用程式的靜態內容，並公開提供內容。 接下來，您將會了解如何使用無伺服器函式，從這個 Web 應用程式將影像上傳至 Blob 儲存體。
