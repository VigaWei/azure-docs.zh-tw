---
title: 教學課程：Azure Active Directory 與 Zscaler One 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Zscaler One 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2020
ms.author: jeedes
ms.openlocfilehash: 4f519ed60dcec85048907be0660e1fd01871ccc3
ms.sourcegitcommit: 2aa52d30e7b733616d6d92633436e499fbe8b069
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/06/2021
ms.locfileid: "97937071"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-one"></a>教學課程：Azure Active Directory 與 Zscaler One 整合

在本教學課程中，您將了解如何整合 Zscaler One 與 Azure Active Directory (Azure AD)。
將 Zscaler One 與 Azure AD 整合可提供下列優點：

* 您可以在 Azure AD 中控制可存取 Zscaler One 的人員。
* 您可以讓使用者使用其 Azure AD 帳戶自動登入 Zscaler One (單一登入)。
* 您可以在 Azure 入口網站中集中管理您的帳戶。


## <a name="prerequisites"></a>Prerequisites

若要設定 Azure AD 與 Zscaler One 的整合，您需要下列項目：

* Azure AD 訂用帳戶。 如果您沒有 Azure AD 環境，您可以申請[免費帳戶](https://azure.microsoft.com/free/)
* 已啟用 Zscaler One 單一登入的訂用帳戶

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* Zscaler One 支援 **SP** 起始的 SSO

* Zscaler One支援 **Just In Time** 使用者佈建

## <a name="adding-zscaler-one-from-the-gallery"></a>從資源庫新增 Zscaler One

若要設定將 Zscaler One 整合到 Azure AD 中，您需要從資源庫將 Zscaler One 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 **Zscaler One**。
1. 從結果面板選取 [Zscaler One]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-zscaler-one"></a>設定和測試 Zscaler One 的 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 Zscaler One 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Zscaler One 中相關使用者之間的連結關聯性。

若要設定及測試與 Zscaler One 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
2. **[設定 Zscaler One SSO](#configure-zscaler-one-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Zscaler One 測試使用者](#create-zscaler-one-test-user)** - 使 Zscaler One 中 Britta Simon 的對應使用者連結到該使用者在 Azure AD 中的代表身分。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 **Zscaler One** 應用程式整合頁面上，尋找 **管理** 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] 的鉛筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

4. 在 [基本 SAML 組態]  區段上，執行下列步驟：

    在 [登入 URL]  文字方塊中，鍵入使用者用來登入您 Zscaler One 的 URL。

    > [!NOTE]
    > 您會使用實際的登入 URL 來更新此值。 請連絡 [Zscaler One 用戶端支援小組](https://www.zscaler.com/company/contact)以取得此值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

5. Zscaler One 應用程式會預期特定格式的 SAML 判斷提示，這會需要您將自訂屬性對應新增至您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示預設屬性清單。 按一下 **編輯** 圖示以開啟 [使用者屬性] 對話方塊。 

    ![顯示使用者屬性的螢幕擷取畫面，其中已選取 [編輯] 圖示。](common/edit-attribute.png)

6. 除了以上屬性外，Zscaler One 應用程式還會預期 SAML 回應傳回更多屬性。 在 [使用者屬性] 對話方塊的 [使用者宣告] 區段中，執行下列步驟以設定 SAML 權杖屬性，如下表所示：
    
    | 名稱 | 來源屬性 |
    | ---------| ------------ |
    | memberOf | user.assignedroles |

    a. 按一下 [新增宣告]  以開啟 [管理使用者宣告]  對話方塊。

    b. 在 [名稱]  文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 讓 [命名空間]  保持空白。

    d. 選取 [來源] 作為 [屬性]  。

    e. 在 [來源屬性]  清單中，輸入該資料列所顯示的屬性值。
    
    f. 按一下 [檔案]  。

    > [!NOTE]
    > 請按一下[這裡](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#app-roles-ui)，以了解如何在 Azure AD 中設定角色。

7. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，按一下 [下載]  ，以依據您的需求從指定選項下載 [憑證 (Base64)]  ，並儲存在您的電腦上。

    ![憑證下載連結](common/certificatebase64.png)

8. 在 [設定 Zscaler One]  區段上，依據需求複製適當的 URL。

    ![複製組態 URL](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者 

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory]  、[使用者]  和 [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]  。
1. 在 [使用者]  屬性中，執行下列步驟：
   1. 在 [名稱]  欄位中，輸入 `B.Simon`。  
   1. 在 [使用者名稱]  欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]  核取方塊，然後記下 [密碼]  方塊中顯示的值。
   1. 按一下 [建立]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Zscaler One 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 [Zscaler One]  。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您已如上述所述設定角色，可以從 **選取角色** 下拉式清單中選取。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。

### <a name="configure-zscaler-one-sso"></a>設定 Zscaler One SSO

1. 若要自動執行 Zscaler One 內的設定，您必須按一下 [安裝擴充功能] 來安裝「我的應用程式安全登入瀏覽器擴充功能」。

    ![我的應用程式擴充功能](common/install-myappssecure-extension.png)

2. 將擴充功能新增至瀏覽器之後，按一下 [設定 Zscaler One]  便會將您導向到 Zscaler One 應用程式。 請從該處提供用以登入 Zscaler One 的管理員認證。 瀏覽器擴充功能會自動為您設定應用程式，並自動執行步驟 3 到 6。

    ![設定 SSO](common/setup-sso.png)

3. 如果您想要手動設定 Zscaler One，請開啟新的網頁瀏覽器視窗，並以系統管理員身分登入 Zscaler One 公司網站，然後執行下列步驟：

4. 移至 管理 > 驗證 > 驗證設定  並執行下列步驟：
   
    ![螢幕擷取畫面：顯示 Zscaler One 網站，並有所述的步驟。](./media/zscaler-one-tutorial/ic800206.png "系統管理")

    a. 在 [驗證類型] 下選擇 [SAML]  。

    b. 按一下 [設定 SAML]  。

5. 在 [編輯 SAML]  視窗上執行下列步驟，然後按一下 [儲存]。  
            
    ![管理使用者和驗證](./media/zscaler-one-tutorial/ic800208.png "管理使用者和驗證")
    
    a. 在 [SAML 入口網站 URL]  文字方塊中，貼上您從 Azure 入口網站複製的 [登入 URL]  。

    b. 在 [登入名稱屬性]  文字方塊中，輸入 **NameID**。

    c. 按一下 [上傳]  ，以上傳您從 Azure 入口網站的 [公開 SSL 憑證]  下載的 Azure SAML 簽署憑證。

    d. 切換 [啟用 SAML 自動佈建]  。

    e. 如果您想要啟用 displayName 屬性的 SAML 自動佈建，請在 [使用者顯示名稱屬性]  文字方塊中，輸入 **displayName**。

    f. 如果您想要啟用 memberOf 屬性的 SAML 自動佈建，請在 [群組名稱屬性]  文字方塊中，輸入 **memberOf**。

    g. 如果您想要啟用 department 屬性的 SAML 自動佈建，請在 [部門名稱屬性]  文字方塊中，輸入 **department**。

    h. 按一下 [檔案]  。

6. 在 [設定使用者驗證]  對話方塊頁面上執行下列步驟：

    ![螢幕擷取畫面：顯示已選取 [啟用] 的 [設定使用者驗證] 對話方塊。](./media/zscaler-one-tutorial/ic800207.png)

    a. 將滑鼠停留在靠近左下方的 [啟用]  功能表上。

    b. 按一下 [啟用]  。

## <a name="configuring-proxy-settings"></a>進行 Proxy 設定
### <a name="to-configure-the-proxy-settings-in-internet-explorer"></a>在 Internet Explorer 中進行 Proxy 設定

1. 啟動 **Internet Explorer**。

2. 從 [工具] 功能表選取 [網際網路選項] 可開啟 [網際網路選項] 對話方塊。   

    ![網際網路選項](./media/zscaler-one-tutorial/ic769492.png "，")

3. 按一下 [連線]  索引標籤。   

    ![連線](./media/zscaler-one-tutorial/ic769493.png "連接")

4. 按一下 [區域網路設定]  可開啟 [區域網路設定]  對話方塊。

5. 在 [Proxy 伺服器] 區段中，執行下列步驟：   
   
    ![Proxy 伺服器](./media/zscaler-one-tutorial/ic769494.png "Proxy 伺服器")

    a. 選取 [在您的區域網路使用 Proxy 伺服器]  。

    b. 在 [位址] 文字方塊中輸入 **gateway.Zscaler One.net**。

    c. 在 [連接埠] 文字方塊中輸入 **80**。

    d. 選取 [近端網址不使用 Proxy 伺服器]  。

    e. 按一下 [確定]  關閉 [區域網路 (LAN) 設定]  對話方塊。

7. 按一下 [確定]  關閉 [網際網路選項]  對話方塊。

### <a name="create-zscaler-one-test-user"></a>建立 Zscaler One 測試使用者

本節會在 Zscaler One 中建立名為 Britta Simon 的使用者。 Zscaler One 支援依預設啟用的 Just-In-Time 使用者佈建。 在這一節沒有您需要進行的動作項目。 如果 Zscaler One 中還沒有任何使用者存在，在驗證之後就會建立新的使用者。

>[!Note]
>如果您需要手動建立使用者，就需要連絡 [ 支援小組](https://www.zscaler.com/company/contact)。

### <a name="test-sso"></a>測試 SSO 

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

* 在 Azure 入口網站中按一下 [測試此應用程式]。 這會重新導向您可以在其中起始登入流程的 Zscaler One 登入 URL。 

* 直接移至 Zscaler One 登入 URL，然後從該處起始登入流程。

* 您可以使用 Microsoft 的「我的應用程式」。 當您按一下「我的應用程式」中的 Zscaler One 圖格時，將會重新導向至 Zscaler One 登入 URL。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。


## <a name="next-steps"></a>後續步驟

設定 Zscaler One 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-any-app)。