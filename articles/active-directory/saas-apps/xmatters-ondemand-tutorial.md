---
title: 教學課程：Azure Active Directory 與 xMatters OnDemand 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 xMatters OnDemand 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/19/2020
ms.author: jeedes
ms.openlocfilehash: cbadf2e072cdd9bfdf64cb2b799355aada8ec4b0
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621149"
---
# <a name="tutorial-azure-active-directory-integration-with-xmatters-ondemand"></a>教學課程：Azure Active Directory 與 xMatters OnDemand 整合

在本教學課程中，您將了解如何整合 xMatters OnDemand 與 Azure Active Directory (Azure AD)。
xMatters OnDemand 與 Azure AD 整合提供下列優點：

* 您可以在 Azure AD 中控制可存取 xMatters OnDemand 的人員。
* 您可以讓使用者使用其 Azure AD 帳戶自動登入 xMatters OnDemand (單一登入)。
* 您可以在 Azure 入口網站中集中管理您的帳戶。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 xMatters OnDemand 的整合，您需要下列項目：

* Azure AD 訂用帳戶。 如果您沒有 Azure AD 環境，您可以申請[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 xMatters OnDemand 單一登入的訂用帳戶

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* xMatters OnDemand 支援由 **IDP** 起始的 SSO

## <a name="adding-xmatters-ondemand-from-the-gallery"></a>從資源庫新增 xMatters OnDemand

若要設定將 xMatters OnDemand 整合到 Azure AD 中，您需要從資源庫將 xMatters OnDemand 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 Azure 入口網站。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務。
1. 巡覽至 [企業應用程式]，然後選取 [所有應用程式]。
1. 若要新增應用程式，請選取 [新增應用程式]  。
1. 在 [從資源庫新增] 區段的搜尋方塊中輸入 **xMatters OnDemand**。
1. 從結果面板選取 [xMatters OnDemand]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。


## <a name="configure-and-test-azure-ad-sso-for-xmatters-ondemand"></a>針對 xMatters OnDemand 設定及測試 Azure AD SSO

以名為 **B.Simon** 的測試使用者，設定及測試與 xMatters OnDemand 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 xMatters OnDemand 中相關使用者之間的連結關聯性。

若要設定及測試與 xMatters OnDemand 搭配運作的 Azure AD SSO，請執行下列步驟：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
    2. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
2. **[設定 xMatters OnDemand SSO](#configure-xmatters-ondemand-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 xMatters OnDemand 測試使用者](#create-xmatters-ondemand-test-user)** - 在 xMatters OnDemand 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
3. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [xMatters OnDemand] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入]。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，按一下 [基本 SAML 設定]  的編輯/畫筆圖示，以編輯設定。

   ![編輯基本 SAML 組態](common/edit-urls.png)

1. 在 [基本 SAML 組態]  區段上，輸入下列欄位的值：

    a. 在 [識別碼] 文字方塊中，使用下列其中一種模式來輸入 URL：

    | 識別碼 |
    | ---------- |
    | `https://<companyname>.au1.xmatters.com.au/` |
    | `https://<companyname>.cs1.xmatters.com/` |
    | `https://<companyname>.xmatters.com/` |
    | `https://www.xmatters.com` |
    | `https://<companyname>.xmatters.com.au/` |

    b. 在 [回覆 URL] 文字方塊中，以下列其中一個模式輸入 URL：

    | 回覆 URL |
    | ---------- |
    |  `https://<companyname>.au1.xmatters.com.au` |
    | `https://<companyname>.xmatters.com/sp/<instancename>` |
    | `https://<companyname>.cs1.xmatters.com/sp/<instancename>` |
    | `https://<companyname>.au1.xmatters.com.au/<instancename>` |

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [Matters OnDemand 用戶端支援小組](https://www.xmatters.com/company/contact-us/)以取得這些值。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

5. 在 [以 SAML 設定單一登入]  頁面的 [SAML 簽署憑證]  區段中，按一下 [下載]  ，以依據您的需求從指定選項下載 [憑證 (Base64)]  ，並儲存在您的電腦上。

    ![憑證下載連結](common/certificatebase64.png)

    > [!IMPORTANT]
    > 您需要將憑證轉送給 [xMatters OnDemand 支援小組](https://www.xmatters.com/company/contact-us/)。 xMatters 支援小組需要先上傳憑證，您才能完成單一登入組態。

6. 在 [設定 xMatters OnDemand] 區段上，依據您的需求複製適當的 URL。

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

在本節中，您會將 xMatters OnDemand 的存取權授與 B.Simon，使其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 在應用程式清單中，選取 **xMatters OnDemand**。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  ，然後在 [新增指派]  對話方塊中選取 [使用者和群組]  。
1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon]，然後按一下畫面底部的 [選取] 按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 下拉式清單中選取。 如果未設定此應用程式的角色，您會看到已選取 [預設存取] 角色。
1. 在 [新增指派]  對話方塊中，按一下 [指派]  按鈕。


## <a name="configure-xmatters-ondemand-sso"></a>設定 xMatters OnDemand SSO

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 xMatters OnDemand 公司網站。

2. 按一下 [系統管理]，然後按一下 [公司詳細資料]。

    ![系統管理頁面](./media/xmatters-ondemand-tutorial/admin.png "Admin")

3. 在 [SAML 組態]  頁面上，執行下列步驟：

    ![SAML 設定區段](./media/xmatters-ondemand-tutorial/saml-configuration.png "SAML 設定")

    a. 選取 [啟用 SAML]  。

    b. 在 [識別提供者識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 [Azure AD 識別碼] 值。

    c. 在 [單一登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登入 URL] 值。

    d. 在 [登出 URL 重新導向] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    e. 按一下 [選擇檔案] 以上傳您從 Azure 入口網站下載的 **憑證 (Base64)** 。 

    f. 在 [公司詳細資料] 頁面的頂端，按一下 [儲存變更]。

    ![公司詳細資料](./media/xmatters-ondemand-tutorial/save-button.png "公司詳細資料")

### <a name="create-xmatters-ondemand-test-user"></a>建立 xMatters OnDemand 測試使用者

1. 登入 **XMatters OnDemand** 租用戶。

2. 移至 **[使用者] 圖示** > [使用者]，然後按一下 [新增使用者]。

    ![使用者](./media/xmatters-ondemand-tutorial/add-user.png "使用者")

3. 在 [新增使用者] 區段中填寫必要的欄位，然後按一下 [新增使用者] 按鈕。

    ![加入使用者](./media/xmatters-ondemand-tutorial/add-user-2.png "加入使用者")



### <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。

* 在 Azure 入口網站中按一下 [測試此應用程式]，您應該會自動登入您已設定 SSO 的 xMatters OnDemand

* 您可以使用 Microsoft 的「我的應用程式」。 當您在「我的應用程式」中按一下 [xMatters OnDemand] 圖格時，應該會自動登入您已設定 SSO 的 xMatters OnDemand。 如需「我的應用程式」的詳細資訊，請參閱[我的應用程式簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="next-steps"></a>後續步驟

設定 xMatters OnDemand 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)。
