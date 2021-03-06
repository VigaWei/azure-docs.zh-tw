---
title: 教學課程：Azure Active Directory 與 Keeper Password Manager & Digital Vault 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Keeper Password Manager & Digital Vault 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/13/2020
ms.author: jeedes
ms.openlocfilehash: b70c50e7c2900f884dd4d91c6650205bc626326e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/26/2020
ms.locfileid: "96177998"
---
# <a name="tutorial-azure-active-directory-integration-with-keeper-password-manager--digital-vault"></a>教學課程：Azure Active Directory 與 Keeper Password Manager & Digital Vault 整合

在本教學課程中，您將了解如何整合 Keeper Password Manager & Digital Vault 與 Azure Active Directory (Azure AD)。
此整合可提供下列優點：

* 您可以在 Azure AD 中控制可存取 Keeper Password Manager & Digital Vault 的人員。
* 您可以讓使用者使用其 Azure AD 帳戶自動登入 Keeper Password Manager & Digital Vault (單一登入)。
* 您可以集中管理您的帳戶：Azure 入口網站。


## <a name="prerequisites"></a>Prerequisites

若要設定與 Keeper Password Manager & Digital Vault 的 Azure AD 整合，您需要：

* Azure AD 訂用帳戶。 如果您沒有 Azure AD 環境，您可以取得[一個月的試用帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* 已啟用單一登入 (SSO) 的 Keeper Password Manager & Digital Vault 訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD 單一登入。

* Keeper Password Manager & Digital Vault 支援由 SP 起始的 SSO。

* Keeper Password Manager & Digital Vault 支援 Just In Time 使用者佈建。

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>從資源庫新增 Keeper Password Manager & Digital Vault

若要進行將 Keeper Password Manager & Digital Vault 整合到 Azure AD 中的設定，您必須從資源庫將應用程式新增至受控的軟體即服務 (SaaS) 應用程式清單。

1. 使用公司或學校帳戶或個人 Microsoft 帳戶登入 Azure 入口網站。
1. 在左窗格中選取 [Azure Active Directory]  服務。
1. 移至 [企業應用程式]  ，然後選取 [所有應用程式]  。
1. 若要新增新的應用程式，請選取 [新增應用程式]。
1. 在 [從資源庫新增] 的搜尋方塊中，輸入 **Keeper Password Manager & Digital Vault**。
1. 從結果面板中選取 [Keeper Password Manager & Digital Vault]，然後新增應用程式。 當應用程式新增至您的租用戶時，請等候幾秒鐘。

## <a name="configure-and-test-azure-ad-sso-for-keeper-password-manager--digital-vault"></a>設定和測試適用於 Keeper Password Manager & Digital Vault 的 Azure AD SSO

使用名為 **B.Simon** 的測試使用者，設定及測試與 Keeper Password Manager & Digital Vault 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須在 Azure AD 使用者和 Keeper Password Manager & Digital Vault 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Keeper Password Manager & Digital Vault 搭配運作的 Azure AD SSO：

1. [設定 Azure AD SSO](#configure-azure-ad-sso)，讓您的使用者能夠使用此功能。

    * [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
    * [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。

1. [設定 Keeper Password Manager & Digital Vault SSO](#configure-keeper-password-manager--digital-vault-sso)，在應用程式端設定 SSO 設定。
    * [建立 Keeper Password Manager & Digital Vault 測試使用者](#create-a-keeper-password-manager--digital-vault-test-user)，使 Keeper Password Manager & Digital Vault 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. [測試 SSO](#test-sso)，以驗證組態是否能運作。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 Azure 入口網站的 [Keeper Password Manager & Digital Vault] 應用程式整合頁面上，尋找 [管理] 區段。 選取 [單一登入]  。
1. 在 [**選取單一登入方法**] 頁面上，選取 [**SAML**]。
1. 在 [以 SAML 設定單一登入]  頁面上，選取 [基本 SAML 組態]  的鉛筆圖示，以編輯設定。

   ![[以 SAML 設定單一登入] 的螢幕擷取畫面，其中已醒目提示鉛筆圖示。](common/edit-urls.png)

4. 在 [基本 SAML 組態]  區段中，執行下列步驟：

    a. 針對 [登入 URL]，輸入採用下列模式的 URL：
    * 針對 Cloud SSO：`https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * 針對內部部署 SSO：`https://<KEEPER_FQDN>/sso-connect/saml/login`

    b. 針對 [識別碼 (實體識別碼)]，輸入採用下列模式的 URL：
    * 針對 Cloud SSO：`https://keepersecurity.com/api/rest/sso/saml/<CLOUD_INSTANCE_ID>`
    * 針對內部部署 SSO：`https://<KEEPER_FQDN>/sso-connect`

    c. 針對 [回覆 URL]，輸入採用下列模式的 URL：
    * 針對 Cloud SSO：`https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * 針對內部部署 SSO：`https://<KEEPER_FQDN>/sso-connect/saml/sso`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的登入 URL、識別碼及回覆 URL 來更新這些值。 若要取得這些值，請連絡 [Keeper Password Manager & Digital Vault 用戶端支援小組](https://keepersecurity.com/contact.html)。 您也可以參考 Azure 入口網站中 **基本 SAML 組態** 區段所示的模式。

1. Keeper Password Manager & Digital Vault 應用程式需要特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示預設屬性清單。

    ![[使用者屬性與宣告] 的螢幕擷取畫面。](common/default-attributes.png)

1. 此外，Keeper Password Manager & Digital Vault 應用程式還需要在 SAML 回應中多傳回幾個屬性。 如下表所示。 這些屬性也會預先填入，但您可以根據您的需求來檢閱這些屬性。

    | 名稱 | 來源屬性|
    | ------------| --------- |
    | First | user.givenname |
    | 最後一個 | user.surname |
    | 電子郵件 | user.mail |

5. 在 [以 SAML 設定單一登入] 頁面的 [SAML 簽署憑證] 區段中，選取 [下載]。 這會根據您的需求從選項下載 [同盟中繼資料 XML]，並將其儲存在您的電腦上。

    ![[SAML 簽署憑證] 的螢幕擷取畫面，其中已醒目提示 [下載]。](common/metadataxml.png)

6. 在 [設定 Keeper Password Manager & Digital Vault] 上，依據您的需求複製適當的 URL。

    ![[設定 Keeper Password Manager & Digital Vault] 的螢幕擷取畫面，其中已醒目提示 URL。](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者 

在本節中，您會在 Azure 入口網站中建立名為 `B.Simon` 的測試使用者。

1. 在 Azure 入口網站的左窗格中，選取 [Azure Active Directory]   > [使用者]   > [所有使用者]  。
1. 在畫面頂端選取 [新增使用者]。
1. 在 [使用者] 屬性中，執行下列步驟：
   1. 針對 [名稱] 輸入 `B.Simon`。  
   1. 針對 [使用者名稱]，輸入 `username@companydomain.extension`。 例如： `B.Simon@contoso.com` 。
   1. 選取 [顯示密碼]，並記下顯示的值。
   1. 選取 [建立]。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Keeper Password Manager & Digital Vault 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式] > [所有應用程式]。
1. 在應用程式清單中，選取 **Keeper Password Manager & Digital Vault**。
1. 在應用程式的概觀頁面中尋找 [管理]  區段，然後選取 [使用者和群組]  。
1. 選取 [新增使用者]  。 在 [新增指派] 中，選取 [使用者和群組]。
1. 在 [使用者和群組] 的使用者清單中，選取 [B.Simon]。 然後，選擇畫面底部的 [選取]  按鈕。
1. 如果您需要將角色指派給使用者，您可以從 [選取角色] 清單中選取。 如果未設定此應用程式的角色，則系統會選取 **預設存取** 角色。
1. 在 [新增指派] 中，選取 [指派]。


## <a name="configure-keeper-password-manager--digital-vault-sso"></a>設定 Keeper Password Manager & Digital Vault SSO

若要為應用程式設定 SSO，請參閱 [Keeper 支援指南](https://docs.keeper.io/sso-connect-guide/)中的指導方針。

### <a name="create-a-keeper-password-manager--digital-vault-test-user"></a>建立 Keeper Password Manager & Digital Vault 測試使用者

若要讓 Azure AD 使用者登入 Keeper Password Manager & Digital Vault，您必須佈建這些使用者。 應用程式支援 Just-In-Time 使用者佈建，而在驗證之後，就會自動在應用程式中建立使用者。 如果您想要手動設定使用者，請連絡 [Keeper 支援](https://keepersecurity.com/contact.html)。

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用下列選項來測試您的 Azure AD 單一登入組態。 

* 在 Azure 入口網站中，選取 [測試此應用程式]。 這會重新導向至 Keeper Password Manager & Digital Vault 的登入 URL，您可以在其中起始登入。 

* 您可以直接移至應用程式的登入 URL，然後從該處起始登入。

* 您可以使用 Microsoft 存取面板。 當您選取存取面板中的 [Keeper Password Manager & Digital Vault] 圖格時，應該會重新導向至應用程式的登入 URL。 如需存取面板的詳細資訊，請參閱[登入我的應用程式入口網站並啟動應用程式](../user-help/my-apps-portal-end-user-access.md)。


## <a name="next-steps"></a>後續步驟

設定 Keeper Password Manager & Digital Vault 後，您可以強制執行工作階段控制項。 這可即時保護貴組織的敏感性資料免於外洩和遭到滲透。 工作階段控制項會從條件式存取延伸。 如需詳細資訊，請參閱[了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](/cloud-app-security/proxy-deployment-aad)。