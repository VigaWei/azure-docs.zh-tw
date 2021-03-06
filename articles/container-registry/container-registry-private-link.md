---
title: 設定私人連結
description: 在容器登錄上設定私人端點，並透過本機虛擬網路中的私人連結來啟用存取。 Private link 存取是 Premium 服務層級的功能。
ms.topic: article
ms.date: 10/01/2020
ms.openlocfilehash: 3193c65a2021d29f03bd9ae6cbc00fd6c349d9bf
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342295"
---
# <a name="connect-privately-to-an-azure-container-registry-using-azure-private-link"></a>使用 Azure Private Link 私下連接到 Azure container registry


將虛擬網路私人 IP 位址指派給登錄端點並使用 [Azure Private Link](../private-link/private-link-overview.md)，以限制登錄的存取權。 虛擬網路上用戶端與登錄私人端點之間的網路流量會在 Microsoft 骨幹網路上進行虛擬網路和私人連結，以消除公用網際網路的洩漏。 Private Link 也可透過 [Azure ExpressRoute](../expressroute/expressroute-introduction.MD) 私人對等互連或 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)，從內部部署啟用私人登錄存取。

您可以設定登錄私人端點的 [DNS 設定](../private-link/private-endpoint-overview.md#dns-configuration) ，讓設定解析為登錄的已配置私人 IP 位址。 透過 DNS 組態，網路中的用戶端和服務可繼續以登錄的完整網域名稱 (例如 myregistry.azurecr.io) 來存取登錄。 

**進階** 容器登錄服務層級中提供這項功能。 目前，最多可以為登錄設定10個私人端點。 如需登錄服務層級和限制的相關資訊，請參閱 [Azure 容器登錄層級](container-registry-skus.md)。

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="prerequisites"></a>Prerequisites

* 若要使用本文中的 Azure CLI 步驟，建議使用 Azure CLI 2.6.0 版或更新版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli]。 或在 [Azure Cloud Shell](../cloud-shell/quickstart.md) 中執行。
* 如果您還沒有容器登錄，請建立一個 (進階層) ，然後[import](container-registry-import-images.md) `mcr.microsoft.com/hello-world` 從 Microsoft container registry 匯入範例公用映射。 例如，使用 [Azure 入口網站][quickstart-portal]或 [Azure CLI][quickstart-cli] 來建立登錄。
* 若要使用不同 Azure 訂用帳戶中的私人連結來設定登錄存取，您必須在該訂用帳戶中註冊 Azure Container Registry 的資源提供者。 例如：

  ```azurecli
  az account set --subscription <Name or ID of subscription of private link>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

本文中的 Azure CLI 範例使用下列環境變數。 取代適用於您環境的值。 所有範例已處理為 Bash 殼層格式：

```bash
REGISTRY_NAME=<container-registry-name>
REGISTRY_LOCATION=<container-registry-location> # Azure region such as westeurope where registry created
RESOURCE_GROUP=<resource-group-name>
VM_NAME=<virtual-machine-name>
```

[!INCLUDE [Set up Docker-enabled VM](../../includes/container-registry-docker-vm-setup.md)]

## <a name="set-up-private-link---cli"></a>設定私人連結 - CLI

### <a name="get-network-and-subnet-names"></a>取得網路和子網路名稱

如果您還沒有這些項目，將需要虛擬網路和子網路的名稱，才能設定私人連結。 在此範例中，您會對 VM 和登錄的私人端點使用相同的子網路。 不過，在許多情況下，您會在不同的子網路中設定端點。 

當您建立 VM 時，Azure 預設會在相同的資源群組中建立虛擬網路。 虛擬網路的名稱是以虛擬機器的名稱為基礎。 例如，如果您將虛擬機器命名為 *myDockerVM* ，預設的虛擬網路名稱會是 *myDockerVMVNET* ，子網路名為 *myDockerVMSubnet* 。 藉由執行 [az network vnet list][az-network-vnet-list] 命令，在環境變數中設定這些值：

```azurecli
NETWORK_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Name: name}' --output tsv)

SUBNET_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Subnet: subnets[0].name}' --output tsv)

echo NETWORK_NAME=$NETWORK_NAME
echo SUBNET_NAME=$SUBNET_NAME
```

### <a name="disable-network-policies-in-subnet"></a>停用子網路中的網路原則

[停用網路原則](../private-link/disable-private-endpoint-network-policy.md)，例如私人端點子網路中的網路安全性群組。 使用 [az network vnet subnet update][az-network-vnet-subnet-update] 來更新子網路設定：

```azurecli
az network vnet subnet update \
 --name $SUBNET_NAME \
 --vnet-name $NETWORK_NAME \
 --resource-group $RESOURCE_GROUP \
 --disable-private-endpoint-network-policies
```

### <a name="configure-the-private-dns-zone"></a>設定私人 DNS 區域

建立私人 Azure container registry 網域的 [私人 DNS 區域](../dns/private-dns-privatednszone.md) 。 在後續步驟中，您會在此 DNS 區域中建立登錄網域的 DNS 記錄。

若要使用私人區域來覆寫 Azure Container Registry 的預設 DNS 解析，區域必須命名為 **privatelink.azurecr.io** 。 執行下列 [az network private-dns zone create][az-network-private-dns-zone-create] 命令以建立私人區域：

```azurecli
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name "privatelink.azurecr.io"
```

### <a name="create-an-association-link"></a>建立關聯連結

執行 [az network private-dns link vnet create][az-network-private-dns-link-vnet-create]，將私人區域與虛擬網路建立關聯。 這個範例會建立名為 myDNSLink 的連結。

```azurecli
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name "privatelink.azurecr.io" \
  --name MyDNSLink \
  --virtual-network $NETWORK_NAME \
  --registration-enabled false
```

### <a name="create-a-private-registry-endpoint"></a>建立私人登錄端點

在本節中，請在虛擬網路中建立登錄的私人端點。 首先，取得登錄的資源識別碼：

```azurecli
REGISTRY_ID=$(az acr show --name $REGISTRY_NAME \
  --query 'id' --output tsv)
```

執行 [az network private-endpoint create][az-network-private-endpoint-create] 命令，以建立登錄的私人端點。

下列範例會建立 myPrivateEndpoint 端點和 myConnection 服務連線。 若要指定端點的容器登錄資源，請傳遞 `--group-ids registry`：

```azurecli
az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $NETWORK_NAME \
    --subnet $SUBNET_NAME \
    --private-connection-resource-id $REGISTRY_ID \
    --group-ids registry \
    --connection-name myConnection
```

### <a name="get-private-ip-addresses"></a>取得私人 IP 位址

執行 [az network private-endpoint show][az-network-private-endpoint-show] 來查詢網路介面識別碼的端點：

```azurecli
NETWORK_INTERFACE_ID=$(az network private-endpoint show \
  --name myPrivateEndpoint \
  --resource-group $RESOURCE_GROUP \
  --query 'networkInterfaces[0].id' \
  --output tsv)
```

在此範例中，與網路介面相關聯的是容器登錄的兩個私人 IP 位址：一個用於登錄本身，另一個用於登錄的資料端點。 下列 [az resource show][az-resource-show] 命令會取得容器登錄和登錄資料端點的私人 IP 位址：

```azurecli
PRIVATE_IP=$(az resource show \
  --ids $NETWORK_INTERFACE_ID \
  --api-version 2019-04-01 \
  --query 'properties.ipConfigurations[1].properties.privateIPAddress' \
  --output tsv)

DATA_ENDPOINT_PRIVATE_IP=$(az resource show \
  --ids $NETWORK_INTERFACE_ID \
  --api-version 2019-04-01 \
  --query 'properties.ipConfigurations[0].properties.privateIPAddress' \
  --output tsv)
```

> [!NOTE]
> 如果您的登錄是[異地複寫](container-registry-geo-replication.md)，請查詢每個登錄複本的其他資料端點。

### <a name="create-dns-records-in-the-private-zone"></a>在私人區域中建立 DNS 記錄

下列命令會在私人區域中建立登錄端點及其資料端點的 DNS 記錄。 例如，如果您在 westeurope 區域中有名為 myregistry 的登錄，則端點名稱為 `myregistry.azurecr.io` 和 `myregistry.westeurope.data.azurecr.io`。 

> [!NOTE]
> 如果您的登錄是[異地複寫](container-registry-geo-replication.md)，請為每個複本的資料端點 IP 建立其他 DNS 記錄。

請先執行 [az network private-dns record-set a create][az-network-private-dns-record-set-a-create]，為登錄端點和資料端點建立空的 A 記錄集：

```azurecli
az network private-dns record-set a create \
  --name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP

# Specify registry region in data endpoint name
az network private-dns record-set a create \
  --name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP
```

執行 [az network private-dns record-set a add-record][az-network-private-dns-record-set-a-add-record] 命令，以建立登錄端點和資料端點的 A 記錄：

```azurecli
az network private-dns record-set a add-record \
  --record-set-name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $PRIVATE_IP

# Specify registry region in data endpoint name
az network private-dns record-set a add-record \
  --record-set-name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $DATA_ENDPOINT_PRIVATE_IP
```

私人連結現在已設定並可供使用。

## <a name="set-up-private-link---portal"></a>設定私人連結 - 入口網站

當您建立登錄時設定私人連結，或將私人連結新增至現有的登錄。 下列步驟假設您已具有虛擬網路和透過 VM 設定的子網路以進行測試。 您也可以[建立新的虛擬網路和子網路](../virtual-network/quick-create-portal.md)。

### <a name="create-a-private-endpoint---new-registry"></a>建立私人端點 - 新的登錄

1. 在入口網站中建立登錄時，請在 [基本] 索引標籤的 [SKU] 中，選取 [Premium]。
1. 選取 [網路] 索引標籤。
1. 在 [網路連線] 中，選取 [私人端點] > [+ 新增]。
1. 輸入或選取下列資訊：

    | 設定 | 值 |
    | ------- | ----- |
    | 訂用帳戶 | 選取您的訂用帳戶。 |
    | 資源群組 | 輸入現有群組的名稱，或建立一個新的群組。|
    | 名稱 | 輸入唯一名稱。 |
    | 子資源 |選取 [登錄]|
    | **網路功能** | |
    | 虛擬網路| 選取您在其中部署虛擬機器的虛擬網路，例如 myDockerVMVNET。 |
    | 子網路 | 選取子網路，例如您在其中部署虛擬機器的 myDockerVMSubnet。 |
    |**私人 DNS 整合**||
    |與私人 DNS 區域整合 |選取 [是]。 |
    |私人 DNS 區域 |選取 (新增) privatelink.azurecr.io |
    |||
1. 設定其餘的登錄設定，然後選取 [檢閱 + 建立]。

  ![建立具有私人端點的登錄](./media/container-registry-private-link/private-link-create-portal.png)

### <a name="create-a-private-endpoint---existing-registry"></a>建立私人端點 - 現有登錄

1. 在入口網站中，瀏覽到您的容器登錄。
1. 在 [設定] 底下，選取 [網路]。
1. 在 [私人端點] 索引標籤上，選取 [+ 私人端點]。
1. 在 [基本] 索引標籤中，輸入或選取以下資訊：

    | 設定 | 值 |
    | ------- | ----- |
    | **專案詳細資料** | |
    | 訂用帳戶 | 選取您的訂用帳戶。 |
    | 資源群組 | 輸入現有群組的名稱，或建立一個新的群組。|
    | **執行個體詳細資料** |  |
    | 名稱 | 輸入名稱。 |
    |區域|選取區域。|
    |||
5. 完成時，選取 [下一步:資源]。
6. 輸入或選取下列資訊：

    | 設定 | 值 |
    | ------- | ----- |
    |連線方法  | 選取 [連線到我目錄中的 Azure 資源]。|
    | 訂用帳戶| 選取您的訂用帳戶。 |
    | 資源類型 | 選取 **Microsoft.ContainerRegistry/registries** 。 |
    | 資源 |選取您登錄的名稱|
    |目標子資源 |選取 [登錄]|
    |||
7. 完成時，選取 [下一步:組態]。
8. 輸入或選取資訊：

    | 設定 | 值 |
    | ------- | ----- |
    |**網路功能**| |
    | 虛擬網路| 選取您在其中部署虛擬機器的虛擬網路，例如 myDockerVMVNET。 |
    | 子網路 | 選取子網路，例如您在其中部署虛擬機器的 myDockerVMSubnet。 |
    |**私人 DNS 整合**||
    |與私人 DNS 區域整合 |選取 [是]。 |
    |私人 DNS 區域 |選取 (新增) privatelink.azurecr.io |
    |||

1. 選取 [檢閱 + 建立]。 您會移至 [檢閱 + 建立] 頁面，其中 Azure 會驗證您的設定。 
2. 當您看到 [驗證成功] 訊息時，請選取 [建立]。

建立私人端點之後，私人區域中的 DNS 設定會出現在入口網站的 [私人端點] 頁面上：

1. 在入口網站中，導覽至您的容器登錄，然後選取 [設定 > 網路]。
1. 在 [私人端點] 索引標籤上，選取您建立的私人端點。
1. 在 [概觀] 頁面上，檢閱連結設定和自訂 DNS 設定。

  ![端點 DNS 設定](./media/container-registry-private-link/private-endpoint-overview.png)

您的私人連結現在已設定並可供使用。

## <a name="disable-public-access"></a>停用公用存取

在許多情況下，停用公用網路的登錄存取。 此設定可防止虛擬網路外部的用戶端連線到登錄端點。 

### <a name="disable-public-access---cli"></a>停用公用存取 - CLI

若要使用 Azure CLI 停用公用存取，請執行 [az acr update][az-acr-update]，並將 `--public-network-enabled` 設定為 `false`。 

> [!NOTE]
> `public-network-enabled` 引數需要 Azure CLI 2.6.0 或更新版本。 

```azurecli
az acr update --name $REGISTRY_NAME --public-network-enabled false
```


### <a name="disable-public-access---portal"></a>停用公用存取 - 入口網站

1. 在入口網站中，瀏覽至您的容器登錄，然後選取 [設定] > [網路]。
1. 在 [公用存取] 索引標籤的 [允許公用網路存取] 中，選取 [停用]。 然後選取 [儲存]。

## <a name="validate-private-link-connection"></a>驗證私人連結連線

您應該驗證私人端點子網路內的資源是否透過私人 IP 位址連線到您的登錄，且具有正確的私人 DNS 區域整合。

若要驗證私人連結連線，請透過 SSH 連線到您在虛擬網路中設定的虛擬機器。

執行或之類的公用 `nslookup` 程式 `dig` ，透過私人連結查閱您登錄的 IP 位址。 例如：

```bash
dig $REGISTRY_NAME.azurecr.io
```

範例輸出會在子網路的位址空間中顯示登錄的 IP 位址：

```console
[...]
; <<>> DiG 9.11.3-1ubuntu1.13-Ubuntu <<>> myregistry.azurecr.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52155
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;myregistry.azurecr.io.         IN      A

;; ANSWER SECTION:
myregistry.azurecr.io.  1783    IN      CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 10 IN A      10.0.0.7

[...]
```

將此結果與 `dig` 輸出中的公用 IP 位址進行比較，以透過公用端點取得相同的登錄：

```console
[...]
;; ANSWER SECTION:
myregistry.azurecr.io.  2881    IN  CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 2881 IN CNAME xxxx.xx.azcr.io.
xxxx.xx.azcr.io.    300 IN  CNAME   xxxx-xxx-reg.trafficmanager.net.
xxxx-xxx-reg.trafficmanager.net. 300 IN CNAME   xxxx.westeurope.cloudapp.azure.com.
xxxx.westeurope.cloudapp.azure.com. 10  IN A 20.45.122.144

[...]
```

### <a name="registry-operations-over-private-link"></a>透過私人連結的登錄作業

也請確認您可以從子網路中的虛擬機器執行登錄作業。 建立虛擬機器的 SSH 連線，並執行 [az acr login][az-acr-login] 以登入您的登錄。 視您的 VM 設定而定，您可能需要在下列命令前面加上 `sudo`。

```bash
az acr login --name $REGISTRY_NAME
```

執行登錄作業 (例如 `docker pull`)，以從登錄中提取範例映像。 以適用於您登錄的映像和標籤取代 `hello-world:v1`，並在前面加上登錄登入伺服器名稱 (全部為小寫)：

```bash
docker pull myregistry.azurecr.io/hello-world:v1
``` 

Docker 已成功將映像提取到 VM。

## <a name="manage-private-endpoint-connections"></a>管理私人端點連線

使用 Azure 入口網站或使用 [az acr private-endpoint-connection][az-acr-private-endpoint-connection] 命令群組中的命令，以管理登錄的私人端點連線。 作業包括核准、刪除、列出、拒絕或顯示登錄私人端點連線的詳細資料。

例如，若要列出登錄的私人端點連線，請執行 [az acr private-endpoint-connection list][az-acr-private-endpoint-connection-list] 命令。 例如：

```azurecli
az acr private-endpoint-connection list \
  --registry-name $REGISTRY_NAME 
```

當您使用本文中的步驟設定私人端點連線時，登錄會自動接受來自用戶端的連線，以及在登錄上具有 Azure RBAC 許可權的服務。 您可以設定端點以要求手動核准連線。 如需有關如何核准及拒絕私人端點連線的詳細資訊，請參閱[管理私人端點連線](../private-link/manage-private-endpoint.md)。

## <a name="add-zone-records-for-replicas"></a>新增複本的區域記錄

如本文所示，當您新增登錄的私人端點連線時，您會在登錄的區域中建立 DNS 記錄，並在複寫登錄的 `privatelink.azurecr.io` 區域中建立其資料端點。 [replicated](container-registry-geo-replication.md) 

如果您稍後新增新的複本，則必須手動為該區域中的資料端點新增新的區域記錄。 例如，如果您在 northeurope 位置中建立 myregistry 的複本，請新增 `myregistry.northeurope.data.azurecr.io` 的區域記錄。 如需相關步驟，請參閱本文中的[在私人區域中建立 DNS 記錄](#create-dns-records-in-the-private-zone)。

## <a name="dns-configuration-options"></a>DNS 組態選項

此範例中的私人端點會與基本虛擬網路相關聯的私人 DNS 區域整合。 此安裝程式會直接使用 Azure 提供的 DNS 服務，將登錄的公用 FQDN 解析為虛擬網路中的私人 IP 位址。 

Private link 支援使用私人區域的其他 DNS 設定案例，包括自訂 DNS 解決方案。 例如，您可能會在虛擬網路中部署自訂 DNS 解決方案，或在網路中使用 VPN 閘道或 Azure ExpressRoute 連線到虛擬網路的內部部署。 

若要將登錄的公用 FQDN 解析為這些案例中的私人 IP 位址，您必須將伺服器層級轉寄站設定為 Azure DNS 服務 (168.63.129.16) 。 確切的設定選項和步驟取決於您現有的網路和 DNS。 如需範例，請參閱 [Azure 私人端點 DNS](../private-link/private-endpoint-dns.md)設定。

> [!IMPORTANT]
> 如果您在數個區域中建立了私人端點，則建議您在每個區域中使用個別的資源群組，並將虛擬網路和相關聯的私人 DNS 區域放在其中。 這項設定也可防止因共用相同私人 DNS 區域而造成無法預期的 DNS 解析。

## <a name="clean-up-resources"></a>清除資源

如果您已在相同的資源群組中建立所有 Azure 資源，但不再需要這些資源，您可以使用單一 [az group delete](/cli/azure/group) 命令，選擇性地刪除資源：

```azurecli
az group delete --name $RESOURCE_GROUP
```

若要在入口網站中清除資源，請導覽至您的資源群組。 載入資源群組後，按一下 [刪除資源群組] 以移除資源群組及儲存於該處的資源。

## <a name="next-steps"></a>後續步驟

* 若要深入了解 Private Link，請參閱 [Azure Private Link](../private-link/private-link-overview.md) 文件。
* 如果您需要設定用戶端防火牆後方的登錄存取規則，請參閱[設定可以從防火牆後方存取 Azure Container Registry 的規則](container-registry-firewall-access-rules.md)。

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-private-endpoint-connection]: /cli/azure/acr/private-endpoint-connection
[az-acr-private-endpoint-connection-list]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-list
[az-acr-private-endpoint-connection-approve]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-approve
[az-acr-update]: /cli/azure/acr#az-acr-update
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-update
[az-network-vnet-list]: /cli/azure/network/vnet/#az-network-vnet-list
[az-network-private-endpoint-create]: /cli/azure/network/private-endpoint#az-network-private-endpoint-create
[az-network-private-endpoint-show]: /cli/azure/network/private-endpoint#az-network-private-endpoint-show
[az-network-private-dns-zone-create]: /cli/azure/network/private-dns/zone#az-network-private-dns-zone-create
[az-network-private-dns-link-vnet-create]: /cli/azure/network/private-dns/link/vnet#az-network-private-dns-link-vnet-create
[az-network-private-dns-record-set-a-create]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-create
[az-network-private-dns-record-set-a-add-record]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-add-record
[az-resource-show]: /cli/azure/resource#az-resource-show
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
