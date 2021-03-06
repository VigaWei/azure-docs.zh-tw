---
title: Azure IoT 中樞作業監視 (已淘汰) | Microsoft Docs
description: 如何使用 Azure IoT 中樞作業監視來即時監視 IoT 中樞上的作業狀態。
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: robinsh
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: 045d5693c4388c6285bc6983ac2a385ceac9f6d0
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/10/2020
ms.locfileid: "94408119"
---
# <a name="iot-hub-operations-monitoring-deprecated"></a>IoT 中樞作業監視 (已淘汰)

IoT 中樞的作業監視可讓您即時監視其 IoT 中樞上的作業狀態。 IoT 中樞可追蹤橫跨數個作業類別的事件。 您可以選擇將一或多個類別的事件傳送至 IoT 中樞的端點進行處理。 您可以監視資料中是否有錯誤，或根據資料模式設定更複雜的處理行為。

>[!NOTE]
>IoT 中樞 **作業監視已淘汰，並已於 2019 年 3 月 10 日自 IoT 中樞移除** 。 若要監視 IoT 中樞的作業和健全狀況，請參閱 [監視 Iot 中樞](monitor-iot-hub.md)。 如需有關淘汰時間表的詳細資訊，請參閱[使用 Azure 監視器和 Azure 資源健康狀態來監視您的 Azure IoT 解決方案](https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health)。

IoT 中樞會監視六個類別的事件：

* 裝置身分識別作業
* 裝置遙測
* 雲端到裝置的訊息
* 連接
* 檔案上傳
* 訊息路由

> [!IMPORTANT]
> IoT 中樞作業監視並不保證事件傳遞的可靠性或順序。 根據 IoT 中樞基礎結構，某些事件可能會遺失或不依順序傳遞。 以錯誤訊號 (例如連線嘗試失敗) 或針對特定裝置的高頻率中斷連線作為基礎，使用作業監視來產生警示。 您不應依賴作業監視事件來建立一致的裝置狀態存放區，例如追蹤裝置連線或中斷連線狀態的存放區。 

## <a name="how-to-enable-operations-monitoring"></a>如何啟用作業監視

1. 建立 IoT 中樞。 您可在[開始使用](quickstart-send-telemetry-dotnet.md)指南中找到如何建立 IoT 中樞的指示。

2. 開啟 IoT 中樞的刀鋒視窗。 按一下其中的 [作業監視] 。

    ![在入口網站中存取作業監視組態](./media/iot-hub-operations-monitoring/enable-OM-1.png)

3. 選取您要監視的監視類別，然後按一下 [儲存]。 您可以從 [監視設定] 中所列出的事件中樞相容端點讀取事件。 IoT 中樞端點稱為 `messages/operationsmonitoringevents`。

    ![在 IoT 中樞上設定作業監視](./media/iot-hub-operations-monitoring/enable-OM-2.png)

> [!NOTE]
> [連線] 類別若選取 [Verbose] 監視，會導致 IoT 中樞產生額外的診斷訊息。 對於所有其他類別，[Verbose] 設定會變更 IoT 中樞在每個錯誤訊息中包含的資訊量。

## <a name="event-categories-and-how-to-use-them"></a>事件類別和其使用方式

每個作業監視類別會各自追蹤與 IoT 中樞所進行的不同類型的互動，而每一個監視類別都有結構描述來定義該類別的事件要如何構成。

### <a name="device-identity-operations"></a>裝置身分識別作業

裝置身分識別作業類別會追蹤您嘗試在其 IoT 中樞的身分識別登錄中建立、更新或刪除項目時所發生的錯誤。 佈建案例就很適合追蹤此類別。

```json
{
    "time": "UTC timestamp",
    "operationName": "create",
    "category": "DeviceIdentityOperations",
    "level": "Error",
    "statusCode": 4XX,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "durationMs": 1234,
    "userAgent": "userAgent",
    "sharedAccessPolicy": "accessPolicy"
}
```

### <a name="device-telemetry"></a>裝置遙測

裝置遙測類別會追蹤在 IoT 中樞發生，且與遙測管線相關的錯誤。 此類別包括在傳送遙測事件 (例如節流) 和接收遙測事件 (例如未經授權的讀取器) 時所發生的錯誤。 這個類別無法捕捉裝置本身所執行之程式碼所造成的錯誤。

```json
{
    "messageSizeInBytes": 1234,
    "batching": 0,
    "protocol": "Amqp",
    "authType": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "DeviceTelemetry",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="cloud-to-device-commands"></a>雲端到裝置的命令

雲端到裝置的命令類別會追蹤在 IoT 中樞發生，且與雲端到裝置訊息管線相關的錯誤。 這個類別包括在傳送雲端到裝置訊息 (例如未經授權的寄件者)、接收雲端到裝置訊息 (例如超過傳遞計數) 和接收雲端到裝置訊息意見反應 (例如意見反應已過期) 時所發生的錯誤。 如果已成功傳遞雲端到裝置訊息，但裝置未適當處理雲端到裝置訊息，此類別不會捕捉來自此裝置的錯誤。

```json
{
    "messageSizeInBytes": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "deliveryAcknowledgement": 0,
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "C2DCommands",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="connections"></a>連接

連線類別會追蹤當裝置與 IoT 中樞連線或中斷連線時發生的錯誤。 若想識別未經授權的連線嘗試，以及在連線品質不佳之區域內的裝置中斷連線時進行追蹤，就很適合追蹤此類別。

```json
{
    "durationMs": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "deviceConnect",
    "category": "Connections",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID"
}
```

### <a name="file-uploads"></a>檔案上傳

檔案上傳類別會追蹤在 IoT 中樞發生且與檔案上傳功能相關的錯誤。 此類別包括︰

* SAS URI 所發生的錯誤，例如當它在裝置通知中樞已完成上傳之前就到期時。

* 裝置所報告的失敗上傳。

* IoT 中樞通知訊息建立期間在儲存體中找不到檔案時所發生的錯誤。

此類別無法捕捉直接發生在裝置將檔案上傳到儲存體時的錯誤。

```json
{
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "HTTP",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "fileUpload",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "blobUri": "http//bloburi.com",
    "durationMs": 1234
}
```

### <a name="message-routing"></a>訊息路由

訊息路由類別會在訊息路由評估期間追蹤發生的錯誤以及 IoT 中樞所認知的端點健全狀況。 此類別包含的事件像是：當規則評估為「未定義」、當 IoT 中樞將端點標示為無效、及其他任何從端點收到錯誤。 此類別不包含有關訊息本身的特定錯誤 (例如裝置節流錯誤)，這些是在「裝置遙測」類別下報告。

```json
{
    "messageSizeInBytes": 1234,
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "routes",
    "level": "Error",
    "deviceId": "device-ID",
    "messageId": "ID of message",
    "routeName": "myroute",
    "endpointName": "myendpoint",
    "details": "ExternalEndpointDisabled"
}
```

## <a name="connect-to-the-monitoring-endpoint"></a>連線至監視端點

IoT 中樞上的監視端點是相容於事件中樞的端點。 您可以使用可搭配「事件中樞」使用的任何機制從此端點讀取監視訊息。 下列範例會建立的基本讀取器不適合用於高輸送量部署。 如需有關如何處理來自「事件中樞」之訊息的詳細資訊，請參閱 [開始使用事件中樞](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) 教學課程。

若要連線至監視端點，您需要連接字串和端點名稱。 下列步驟顯示如何在入口網站中尋找需要的值：

1. 在入口網站中，瀏覽至您的 IoT 中樞資源刀鋒視窗。

2. 選擇 [作業監視]，並記下 [事件中樞相容名稱]和 [事件中樞相容端點]值：

    ![事件中樞相容端點值](./media/iot-hub-operations-monitoring/monitoring-endpoint.png)

3. 選擇 [共用存取原則]，然後選擇 [服務]。 記下 **主要金鑰** 值：

    ![服務共用存取原則主要金鑰](./media/iot-hub-operations-monitoring/service-key.png)

下列 C# 程式碼範例取自 Visual Studio **Windows 傳統桌面** C# 主控台應用程式。 專案已安裝 **WindowsAzure.ServiceBus** NuGet 套件。

* 以您先前所記下，使用 **事件中樞相容端點** 和服務 **主要金鑰** 值的連接字串來取代連接字串預留位置，如下列範例所示：

    ```csharp
    "Endpoint={your Event Hub-compatible endpoint};SharedAccessKeyName=service;SharedAccessKey={your service primary key value}"
    ```

* 以您先前記下的 **事件中樞相容名稱** 值取代監視端點名稱預留位置。

```csharp
class Program
{
    static string connectionString = "{your monitoring endpoint connection string}";
    static string monitoringEndpointName = "{your monitoring endpoint name}";
    static EventHubClient eventHubClient;

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();

        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }

        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }

            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));

            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

若要進一步探索如何使用 Azure 監視器來監視 IoT 中樞，請參閱：

* [監視 IoT 中樞](monitor-iot-hub.md)

* [從 IoT 中樞作業監視遷移至 Azure 監視器](iot-hub-migrate-to-diagnostics-settings.md)