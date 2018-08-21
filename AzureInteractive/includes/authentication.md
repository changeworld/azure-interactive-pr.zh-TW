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
ms.openlocfilehash: d1f9a07ce3d3b096b498e48b5c4f68c3454b2b37
ms.sourcegitcommit: e721422a57e6deb95245135fd9f4f5677c344d93
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "40079133"
---
<span data-ttu-id="0d986-103">Azure App Service 驗證可在 Azure 函式應用程式實現周全的驗證支援。</span><span class="sxs-lookup"><span data-stu-id="0d986-103">Azure App Service Authentication enables turn-key authentication support in an Azure Function app.</span></span> <span data-ttu-id="0d986-104">它可與 Facebook、Twitter、Microsoft 帳戶、Google 和 Azure Active Directory 無縫整合。</span><span class="sxs-lookup"><span data-stu-id="0d986-104">It integrates seamlessly with Facebook, Twitter, Microsoft Account, Google, and Azure Active Directory.</span></span> <span data-ttu-id="0d986-105">您會新增 App Service 驗證來保護 Web 應用程式的後端 API。</span><span class="sxs-lookup"><span data-stu-id="0d986-105">You'll add App Service Authentication to protect the backend APIs of your web app.</span></span>

## <a name="enable-app-service-authentication"></a><span data-ttu-id="0d986-106">啟用 App Service 驗證</span><span class="sxs-lookup"><span data-stu-id="0d986-106">Enable App Service Authentication</span></span>

1. <span data-ttu-id="0d986-107">在 Azure 入口網站中開啟函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="0d986-107">Open the function app in the Azure Portal.</span></span>

1. <span data-ttu-id="0d986-108">在 [平台功能] 底下，選取 [驗證/授權]。</span><span class="sxs-lookup"><span data-stu-id="0d986-108">Under **Platform features**, select **Authentication/Authorization**.</span></span>

    ![選取 [驗證/授權]](media/functions-first-serverless-web-app/6-authorization.jpg)

1. <span data-ttu-id="0d986-110">輸入下列值：</span><span class="sxs-lookup"><span data-stu-id="0d986-110">Select the following values:</span></span>
    
    | <span data-ttu-id="0d986-111">設定</span><span class="sxs-lookup"><span data-stu-id="0d986-111">Setting</span></span>      |  <span data-ttu-id="0d986-112">建議的值</span><span class="sxs-lookup"><span data-stu-id="0d986-112">Suggested value</span></span>   | <span data-ttu-id="0d986-113">說明</span><span class="sxs-lookup"><span data-stu-id="0d986-113">Description</span></span>                                        |
    | --- | --- | ---|
    | <span data-ttu-id="0d986-114">**App Service 驗證**</span><span class="sxs-lookup"><span data-stu-id="0d986-114">**App Service Authentication**</span></span> | <span data-ttu-id="0d986-115">另一</span><span class="sxs-lookup"><span data-stu-id="0d986-115">On</span></span> | <span data-ttu-id="0d986-116">啟用驗證。</span><span class="sxs-lookup"><span data-stu-id="0d986-116">Enable authentication.</span></span> |
    | <span data-ttu-id="0d986-117">**當要求未經驗證時所要採取的動作**</span><span class="sxs-lookup"><span data-stu-id="0d986-117">**Action when request is not authenticated**</span></span> | <span data-ttu-id="0d986-118">使用 Azure Active Directory 登入</span><span class="sxs-lookup"><span data-stu-id="0d986-118">Log in with Azure Active Directory</span></span> | <span data-ttu-id="0d986-119">選取已設定的驗證方法 (下方)。</span><span class="sxs-lookup"><span data-stu-id="0d986-119">Select a configured authentication method (below).</span></span> |
    | <span data-ttu-id="0d986-120">**驗證提供者**</span><span class="sxs-lookup"><span data-stu-id="0d986-120">**Authentication Providers**</span></span> | <span data-ttu-id="0d986-121">請參閱下方</span><span class="sxs-lookup"><span data-stu-id="0d986-121">See below</span></span> | <span data-ttu-id="0d986-122">請參閱下方</span><span class="sxs-lookup"><span data-stu-id="0d986-122">See below</span></span> |
    | <span data-ttu-id="0d986-123">**權杖存放區**</span><span class="sxs-lookup"><span data-stu-id="0d986-123">**Token store**</span></span> | <span data-ttu-id="0d986-124">另一</span><span class="sxs-lookup"><span data-stu-id="0d986-124">On</span></span> | <span data-ttu-id="0d986-125">可讓 App Service 儲存和管理權杖。</span><span class="sxs-lookup"><span data-stu-id="0d986-125">Allow App Service to store and manage tokens.</span></span> |
    | <span data-ttu-id="0d986-126">**允許的外部重新導向 URL**</span><span class="sxs-lookup"><span data-stu-id="0d986-126">**Allowed external redirect URLs**</span></span> | <span data-ttu-id="0d986-127">應用程式的 URL，例如：https://firstserverlessweb.z4.web.core.windows.net/</span><span class="sxs-lookup"><span data-stu-id="0d986-127">The URL of your application, for example: https://firstserverlessweb.z4.web.core.windows.net/</span></span> | <span data-ttu-id="0d986-128">使用者通過驗證後，可作為 App Service 重新導向目標的 URL。</span><span class="sxs-lookup"><span data-stu-id="0d986-128">URL(s) that App Service is allowed to redirect to after a user is authenticated.</span></span> |

1. <span data-ttu-id="0d986-129">選取 [Azure Active Directory] 以顯示 [Azure Active Directory 設定]。</span><span class="sxs-lookup"><span data-stu-id="0d986-129">Select **Azure Active Directory** to reveal **Azure Active Directory Settings**.</span></span>

    1. <span data-ttu-id="0d986-130">選取 [快速] 作為 [管理模式]，並填入下列資訊。</span><span class="sxs-lookup"><span data-stu-id="0d986-130">Select **Express** as the **Management Mode** and fill in the following information.</span></span>
    
        | <span data-ttu-id="0d986-131">設定</span><span class="sxs-lookup"><span data-stu-id="0d986-131">Setting</span></span>      |  <span data-ttu-id="0d986-132">建議的值</span><span class="sxs-lookup"><span data-stu-id="0d986-132">Suggested value</span></span>   | <span data-ttu-id="0d986-133">說明</span><span class="sxs-lookup"><span data-stu-id="0d986-133">Description</span></span>                                        |
        | --- | --- | ---|
        | <span data-ttu-id="0d986-134">**管理模式**</span><span class="sxs-lookup"><span data-stu-id="0d986-134">**Management mode**</span></span> | <span data-ttu-id="0d986-135">快速、建立新的 AD 應用程式</span><span class="sxs-lookup"><span data-stu-id="0d986-135">Express, Create new AD app</span></span> | <span data-ttu-id="0d986-136">自動設定服務主體與 Azure Active Directory 驗證。</span><span class="sxs-lookup"><span data-stu-id="0d986-136">Automatically set up a service principal and Azure Active Directory authentication.</span></span> |
        | <span data-ttu-id="0d986-137">**建立應用程式**</span><span class="sxs-lookup"><span data-stu-id="0d986-137">**Create app**</span></span> | <span data-ttu-id="0d986-138">my-serverless-webapp</span><span class="sxs-lookup"><span data-stu-id="0d986-138">my-serverless-webapp</span></span> | <span data-ttu-id="0d986-139">輸入唯一的應用程式名稱。</span><span class="sxs-lookup"><span data-stu-id="0d986-139">Enter a unique application name.</span></span> |
    
    1. <span data-ttu-id="0d986-140">按一下 [確定] 以儲存 Azure Active Directory 設定。</span><span class="sxs-lookup"><span data-stu-id="0d986-140">Click **OK** to save the Azure Active Directory settings.</span></span>

    ![[驗證/授權] 和 [Azure Active Directory 設定]](media/functions-first-serverless-web-app/6-create-aad.png)

1. <span data-ttu-id="0d986-142">按一下 [檔案] 。</span><span class="sxs-lookup"><span data-stu-id="0d986-142">Click **Save**.</span></span>


## <a name="modify-the-web-app-to-enable-authentication"></a><span data-ttu-id="0d986-143">修改 Web 應用程式以啟用驗證</span><span class="sxs-lookup"><span data-stu-id="0d986-143">Modify the web app to enable authentication</span></span>

1. <span data-ttu-id="0d986-144">在 Cloud Shell 中，確定目前的目錄為 **www/dist** 資料夾。</span><span class="sxs-lookup"><span data-stu-id="0d986-144">In the Cloud Shell, ensure that the current directory is the **www/dist** folder.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. <span data-ttu-id="0d986-145">若要在函式應用程式中啟用驗證，請將下列程式碼附加到 **settings.js**。</span><span class="sxs-lookup"><span data-stu-id="0d986-145">To enable authentication in your function app, append the following line of code to **settings.js**.</span></span>

    `window.authEnabled = true`

    <span data-ttu-id="0d986-146">藉由使用下列命令或使用命令列編輯器 (例如 VIM)，來進行變更和檢視結果。</span><span class="sxs-lookup"><span data-stu-id="0d986-146">Make the change and view the result by using the following commands or by using a command-line editor like VIM.</span></span>

    ```azurecli
    echo "window.authEnabled = true" >> settings.js
    ```

    <span data-ttu-id="0d986-147">確認已對檔案進行變更。</span><span class="sxs-lookup"><span data-stu-id="0d986-147">Confirm the change was made to the file.</span></span>

    ```azurecli
    cat settings.js
    ```

1. <span data-ttu-id="0d986-148">將檔案上傳至 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="0d986-148">Upload the file to Blob storage.</span></span>

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-application"></a><span data-ttu-id="0d986-149">測試應用程式</span><span class="sxs-lookup"><span data-stu-id="0d986-149">Test the application</span></span>

1. <span data-ttu-id="0d986-150">在瀏覽器中開啟應用程式。</span><span class="sxs-lookup"><span data-stu-id="0d986-150">Open the application in a browser.</span></span> <span data-ttu-id="0d986-151">按一下 [登入] 並進行登入。</span><span class="sxs-lookup"><span data-stu-id="0d986-151">Click **Log in** and log in.</span></span>

1. <span data-ttu-id="0d986-152">選取影像檔，並將它上傳。</span><span class="sxs-lookup"><span data-stu-id="0d986-152">Select an image file and upload it.</span></span>

    ![登入頁面](media/functions-first-serverless-web-app/6-aad-auth.png)
    

## <a name="summary"></a><span data-ttu-id="0d986-154">總結</span><span class="sxs-lookup"><span data-stu-id="0d986-154">Summary</span></span>

<span data-ttu-id="0d986-155">在本單元中，您已了解如何使用 Azure App Service 驗證，對應用程式新增驗證。</span><span class="sxs-lookup"><span data-stu-id="0d986-155">In this module, you learned how to add authentication to the applcation using Azure App Service Authentication.</span></span>
