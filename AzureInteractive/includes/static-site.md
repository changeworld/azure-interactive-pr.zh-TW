<span data-ttu-id="ed094-101">Azure Blob 儲存體是低成本且可大幅調整的服務，可用來裝載靜態檔案。</span><span class="sxs-lookup"><span data-stu-id="ed094-101">Azure Blob storage is a low-cost and massively scalable service that can be used to host static files.</span></span> <span data-ttu-id="ed094-102">在本教學課程中，您會使用它來為您建置的 Web 應用程式提供靜態內容 (例如，HTML、JavaScript、CSS)。</span><span class="sxs-lookup"><span data-stu-id="ed094-102">For this tutorial, you use it to serve static content (for example, HTML, JavaScript, CSS) for the web app that you build.</span></span>

## <a name="create-a-storage-account"></a><span data-ttu-id="ed094-103">建立儲存體帳戶</span><span class="sxs-lookup"><span data-stu-id="ed094-103">Create a Storage account</span></span>

<span data-ttu-id="ed094-104">儲存體帳戶是一種 Azure 資源，可讓您儲存資料表、佇列、檔案、Blob (物件) 與虛擬機器磁碟。</span><span class="sxs-lookup"><span data-stu-id="ed094-104">A Storage account is an Azure resource that allows you to store tables, queues, files, blobs (objects), and virtual machine disks.</span></span>

1. <span data-ttu-id="ed094-105">選取 [進入焦點模式] 按鈕，以登入 Cloud Shell (Bash)。</span><span class="sxs-lookup"><span data-stu-id="ed094-105">Log in to the Cloud Shell (Bash), by selecting the **Enter focus mode** button.</span></span> <span data-ttu-id="ed094-106">視瀏覽器視窗的寬度而定，此按鈕會位於頁面右上方或底部。</span><span class="sxs-lookup"><span data-stu-id="ed094-106">This button is at the top right or the bottom of the page, depending on how wide your browser window is.</span></span> <span data-ttu-id="ed094-107">焦點模式會將 Cloud Shell 視窗停駐在瀏覽器視窗右邊，讓您可以輕鬆地執行本教學課程所示的命令。</span><span class="sxs-lookup"><span data-stu-id="ed094-107">Focus mode docks a Cloud Shell window on the right side of your browser window, so you can easily execute commands that are shown in the tutorial.</span></span>

1. <span data-ttu-id="ed094-108">在 Azure 中，資源群組是一種容器，可保存相關 Azure 資源以方便您管理。</span><span class="sxs-lookup"><span data-stu-id="ed094-108">In Azure, a Resource Group is a container that holds related Azure resources for ease of management.</span></span> <span data-ttu-id="ed094-109">建立名為 **first-serverless-app** 的新資源群組。</span><span class="sxs-lookup"><span data-stu-id="ed094-109">Create a new resource group named **first-serverless-app**.</span></span>

    ```azurecli
    az group create -n first-serverless-app -l westcentralus
    ```

1. <span data-ttu-id="ed094-110">本教學課程的靜態內容 (HTML、CSS 和 JavaScript 檔案) 會裝載在 Blob 儲存體中。</span><span class="sxs-lookup"><span data-stu-id="ed094-110">The static content (HTML, CSS, and JavaScript files) for this tutorial is hosted in Blob Storage.</span></span> <span data-ttu-id="ed094-111">Blob 儲存體需要儲存體帳戶。</span><span class="sxs-lookup"><span data-stu-id="ed094-111">Blob Storage requires a Storage account.</span></span> <span data-ttu-id="ed094-112">請在資源群組中建立儲存體帳戶 (一般用途 V2)。</span><span class="sxs-lookup"><span data-stu-id="ed094-112">Create a Storage account (general purpose V2) in the resource group.</span></span> <span data-ttu-id="ed094-113">將 `<storage account name>` 更換為唯一名稱。</span><span class="sxs-lookup"><span data-stu-id="ed094-113">Replace `<storage account name>` with a unique name.</span></span>

    ```azurecli
    az storage account create -n <storage account name> -g first-serverless-app --kind StorageV2 -l westcentralus --https-only true --sku Standard_LRS
    ```

1. <span data-ttu-id="ed094-114">使用 [Azure 入口網站](https://portal.azure.com)頂端的 [搜尋] 列，尋找您剛才建立的儲存體帳戶並加以開啟。</span><span class="sxs-lookup"><span data-stu-id="ed094-114">Using the Search bar at the top of the [Azure portal](https://portal.azure.com), find the storage account you just created and open it.</span></span>

1. <span data-ttu-id="ed094-115">在左側導覽中，選取 [靜態網站 (預覽)] 來設定靜態網站託管的容器。</span><span class="sxs-lookup"><span data-stu-id="ed094-115">On the left navigation, select **Static website (preview)** to configure a container for static website hosting.</span></span>
    - <span data-ttu-id="ed094-116">選取 [已啟用] 來啟用靜態網站。</span><span class="sxs-lookup"><span data-stu-id="ed094-116">Select **Enabled** to enable static website.</span></span>
    - <span data-ttu-id="ed094-117">輸入「index.html」作為索引文件名稱。</span><span class="sxs-lookup"><span data-stu-id="ed094-117">Enter *index.html* as the index document name.</span></span> <span data-ttu-id="ed094-118">欄位中已經有灰色字型的「index.html」，但這只是文字範例；您還是必須在欄位中輸入該值。</span><span class="sxs-lookup"><span data-stu-id="ed094-118">The field already has *index.html* in a gray font but this is only example text; you still have to enter that value in the field.</span></span>
    - <span data-ttu-id="ed094-119">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="ed094-119">Click **Save**.</span></span>
    
    ![輸入靜態網站設定](media/functions-first-serverless-web-app/1-storage-static-website.png)

1. <span data-ttu-id="ed094-121">將 [主要端點] 儲存到某個可方便您在進行教學課程期間複製此值的地方。</span><span class="sxs-lookup"><span data-stu-id="ed094-121">Save the **Primary Endpoint** somewhere you can conveniently copy it from while working through the tutorial.</span></span> <span data-ttu-id="ed094-122">這是 Web 應用程式的 URL。</span><span class="sxs-lookup"><span data-stu-id="ed094-122">This is the URL of your web application.</span></span>

## <a name="upload-the-web-application"></a><span data-ttu-id="ed094-123">上傳 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="ed094-123">Upload the web application</span></span>

1. <span data-ttu-id="ed094-124">您在本教學課程中所建置的應用程式原始程式檔位於 [GitHub 存放庫](https://github.com/Azure-Samples/functions-first-serverless-web-application)。</span><span class="sxs-lookup"><span data-stu-id="ed094-124">The source files for the application that you build in this tutorial are located in a [GitHub repository](https://github.com/Azure-Samples/functions-first-serverless-web-application).</span></span> <span data-ttu-id="ed094-125">請確定您位於 Cloud Shell 的主目錄，並複製這個存放庫。</span><span class="sxs-lookup"><span data-stu-id="ed094-125">Make sure that you are in your home directory in Cloud Shell and clone this repository.</span></span>

    ```azurecli
    cd ~
    git clone https://github.com/Azure-Samples/functions-first-serverless-web-application
    ```

    <span data-ttu-id="ed094-126">此存放庫會複製到 `/home/<username>/functions-first-serverless-web-application`。</span><span class="sxs-lookup"><span data-stu-id="ed094-126">The repository is cloned to `/home/<username>/functions-first-serverless-web-application`.</span></span>

1. <span data-ttu-id="ed094-127">用戶端 Web 應用程式位於 **www** 資料夾，並且會使用 Vue.js JavaScript 架構來建置。</span><span class="sxs-lookup"><span data-stu-id="ed094-127">The client-side web application is located in the **www** folder and is built using the Vue.js JavaScript framework.</span></span> <span data-ttu-id="ed094-128">變更至該資料夾，然後執行 npm 命令來安裝應用程式的相依性，並建置應用程式。</span><span class="sxs-lookup"><span data-stu-id="ed094-128">Change into the folder and run npm commands to install the application's dependencies and build the application.</span></span> <span data-ttu-id="ed094-129">這些命令的最後一個可能需要幾分鐘的時間才會完成。</span><span class="sxs-lookup"><span data-stu-id="ed094-129">The last of these commands might take several minutes to complete.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www
    npm install
    npm run generate
    ```

    <span data-ttu-id="ed094-130">應用程式會在 **dist** 資料夾中產生。</span><span class="sxs-lookup"><span data-stu-id="ed094-130">The application is generated in the **dist** folder.</span></span>

1. <span data-ttu-id="ed094-131">將目前的目錄變更至 **dist**，然後將應用程式上傳至 **$web** Blob 容器。</span><span class="sxs-lookup"><span data-stu-id="ed094-131">Change the current directory to **dist** and upload the application to the **$web** Blob container.</span></span>

    ```azurecli
    cd dist
    az storage blob upload-batch -s . -d \$web --account-name <storage account name>
    ```

1. <span data-ttu-id="ed094-132">在網頁瀏覽器中開啟儲存體靜態網站的主要端點 URL，以檢視應用程式。</span><span class="sxs-lookup"><span data-stu-id="ed094-132">Open the Storage static websites primary endpoint URL in a web browser to view the application.</span></span>

    ![第一個無伺服器 Web 應用程式首頁](media/functions-first-serverless-web-app/1-app-screenshot-new.png)


## <a name="summary"></a><span data-ttu-id="ed094-134">總結</span><span class="sxs-lookup"><span data-stu-id="ed094-134">Summary</span></span>

<span data-ttu-id="ed094-135">在本單元中，您已建立名為 **first-serverless-app** 且包含儲存體帳戶的資源群組。</span><span class="sxs-lookup"><span data-stu-id="ed094-135">In this module, you created a resource group named **first-serverless-app** containing a Storage account.</span></span> <span data-ttu-id="ed094-136">儲存體帳戶中名為 **$web** 的 Blob 容器，會儲存 Web 應用程式的靜態內容，並公開提供內容。</span><span class="sxs-lookup"><span data-stu-id="ed094-136">A blob container named **$web** in the Storage account stores the static content for your web application and makes the content available publicly.</span></span> <span data-ttu-id="ed094-137">接下來，您將會了解如何使用無伺服器函式，從這個 Web 應用程式將影像上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="ed094-137">Next, you learn how to use a serverless function to upload images to Blob storage from this web application.</span></span>
