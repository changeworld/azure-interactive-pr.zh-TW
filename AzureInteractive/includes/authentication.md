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
Azure App Service 驗證可在 Azure 函式應用程式實現周全的驗證支援。 它可與 Facebook、Twitter、Microsoft 帳戶、Google 和 Azure Active Directory 無縫整合。 您會新增 App Service 驗證來保護 Web 應用程式的後端 API。

## <a name="enable-app-service-authentication"></a>啟用 App Service 驗證

1. 在 Azure 入口網站中開啟函式應用程式。

1. 在 [平台功能] 底下，選取 [驗證/授權]。

    ![選取 [驗證/授權]](media/functions-first-serverless-web-app/6-authorization.jpg)

1. 輸入下列值：
    
    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | ---|
    | **App Service 驗證** | 另一 | 啟用驗證。 |
    | **當要求未經驗證時所要採取的動作** | 使用 Azure Active Directory 登入 | 選取已設定的驗證方法 (下方)。 |
    | **驗證提供者** | 請參閱下方 | 請參閱下方 |
    | **權杖存放區** | 另一 | 可讓 App Service 儲存和管理權杖。 |
    | **允許的外部重新導向 URL** | 應用程式的 URL，例如：https://firstserverlessweb.z4.web.core.windows.net/ | 使用者通過驗證後，可作為 App Service 重新導向目標的 URL。 |

1. 選取 [Azure Active Directory] 以顯示 [Azure Active Directory 設定]。

    1. 選取 [快速] 作為 [管理模式]，並填入下列資訊。
    
        | 設定      |  建議的值   | 說明                                        |
        | --- | --- | ---|
        | **管理模式** | 快速、建立新的 AD 應用程式 | 自動設定服務主體與 Azure Active Directory 驗證。 |
        | **建立應用程式** | my-serverless-webapp | 輸入唯一的應用程式名稱。 |
    
    1. 按一下 [確定] 以儲存 Azure Active Directory 設定。

    ![[驗證/授權] 和 [Azure Active Directory 設定]](media/functions-first-serverless-web-app/6-create-aad.png)

1. 按一下 [檔案] 。


## <a name="modify-the-web-app-to-enable-authentication"></a>修改 Web 應用程式以啟用驗證

1. 在 Cloud Shell 中，確定目前的目錄為 **www/dist** 資料夾。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www/dist
    ```

1. 若要在函式應用程式中啟用驗證，請將下列程式碼附加到 **settings.js**。

    `window.authEnabled = true`

    藉由使用下列命令或使用命令列編輯器 (例如 VIM)，來進行變更和檢視結果。

    ```azurecli
    echo "window.authEnabled = true" >> settings.js
    ```

    確認已對檔案進行變更。

    ```azurecli
    cat settings.js
    ```

1. 將檔案上傳至 Blob 儲存體。

    ```azurecli
    az storage blob upload -c \$web --account-name <storage account name> -f settings.js -n settings.js
    ```


## <a name="test-the-application"></a>測試應用程式

1. 在瀏覽器中開啟應用程式。 按一下 [登入] 並進行登入。

1. 選取影像檔，並將它上傳。

    ![登入頁面](media/functions-first-serverless-web-app/6-aad-auth.png)
    

## <a name="summary"></a>總結

在本單元中，您已了解如何使用 Azure App Service 驗證，對應用程式新增驗證。
