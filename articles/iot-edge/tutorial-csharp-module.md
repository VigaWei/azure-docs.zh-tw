---
title: 教學課程 - 使用 Azure IoT Edge 開發 Linux 的 C# 模組
description: 本教學課程說明如何使用 C# 程式碼建立 IoT Edge 模組，並將其部署到 Linux IoT Edge 裝置。
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 71bfc84eb50521aef72f78b482bddda112c00c6c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964349"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-linux-devices"></a>教學課程：開發適用於 Linux 裝置的 C# IoT Edge 模組

使用 Visual Studio Code 開發 C# 程式碼，並將它部署到執行 Azure IoT Edge 的 Linux 裝置。

您可以使用 Azure IoT Edge 模組來部署程式碼，直接在 IoT Edge 裝置上實作您的商務邏輯。 本教學課程會逐步引導您建立並部署能篩選感應器資料的 IoT Edge 模組。 您將會使用模擬的 IoT Edge 裝置，其建立方法詳述於在 [Windows](quickstart.md) 或 [Linux](quickstart-linux.md) 中的模擬裝置上部署 Azure IoT Edge 的快速入門中。 在本教學課程中，您會了解如何：

> [!div class="checklist"]
>
> * 使用 Visual Studio Code 建立以 .NET Core 2.1 SDK 為基礎的 IoT Edge 模組。
> * 使用 Visual Studio Code 和 Docker 建立 Docker 映像，並將其發佈至您的登錄中。
> * 將模組部署到您的 IoT Edge 裝置。
> * 檢視產生的資料。

您於此教學課程中建立的 IoT Edge 模組，能夠篩選由您裝置所產生的溫度資料。 它只有在溫度超過指定的閾值時，才會將訊息往上游傳送。 這類於邊緣所進行的分析，對於減少針對雲端所傳輸及儲存的資料量相當有幫助。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

本教學課程示範如何使用 **Visual Studio Code** 以 **C#** 開發模組，以及如何將其部署到 **Linux 裝置**。 如果您要開發適用於 Windows 裝置的模組，請改為移至[開發適用於 Windows 裝置的 C# IoT Edge 模組](tutorial-csharp-module-windows.md)。

請使用下表了解開發 C# 模組並部署至 Linux 的選項：

| C# | Visual Studio Code | Visual Studio |
| -- | ------------------ | ------------- |
| **Linux AMD64** | ![VS Code 中適用於 LinuxAMD64 的 C# 模組](./media/tutorial-c-module/green-check.png) | ![Visual Studio 中適用於 LinuxAMD64 的 C# 模組](./media/tutorial-c-module/green-check.png) |
| **Linux ARM32** | ![VS Code 中適用於 LinuxARM32 的 C# 模組](./media/tutorial-c-module/green-check.png) | ![Visual Studio 中適用於 LinuxARM64 的 C# 模組](./media/tutorial-c-module/green-check.png) |

>[!NOTE]
>[公開預覽](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)提供 Linux ARM64 裝置的支援。 如需詳細資訊，請參閱[在 Visual Studio Code (預覽)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview) 中開發和偵錯 ARM64 IoT Edge 模組。

在開始本教學課程之前，您應該已經完成先前的教學課程 ([開發適用於 Linux 裝置的 IoT Edge 模組](tutorial-develop-for-linux.md)) 來設定您的開發環境。 完成該教學課程之後，您已經具備下列必要條件：

* Azure 中的免費或標準層 [IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)。
* [執行 Azure IoT Edge 的 Linux 裝置](quickstart-linux.md)。
* 容器登錄，像是 [Azure Container Registry](../container-registry/index.yml)。
* 已設定 [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) 的 [Visual Studio Code](https://code.visualstudio.com/)。
* [Docker CE](https://docs.docker.com/install/) 設定為執行 Linux 容器。

若要完成下列教學課程，請在您的開發機器上備妥下列其他必要條件：

* [C# for Visual Studio Code (採用 OmniSharp 技術) 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)。
* [.NET Core 2.1 SDK](https://www.microsoft.com/net/download)。

## <a name="create-a-module-project"></a>建立模組專案

下列步驟會使用 Visual Studio Code 和 Azure IoT Tools 擴充功能來建立適用於 C# 的 IoT Edge 模組專案。 建立專案範本後，新增程式碼，以便模組根據其報告的屬性篩選出訊息。

### <a name="create-a-new-project"></a>建立新專案

建立您可以使用自己的程式碼自訂的 C# 解決方案範本。

1. 在 Visual Studio Code 中，選取 [檢視]   > [命令選擇區]  ，以開啟 VS Code 命令選擇區。

2. 在 [命令選擇區] 中，輸入並執行命令 **Azure:Sign in**，然後遵循指示來登入您的 Azure 帳戶。 如果您已登入，則可以略過此步驟。

3. 在命令選擇區中，輸入並執行命令 Azure IoT Edge：**新增 IoT Edge 解決方案**。 依照命令選擇區中的提示建立解決方案。

   | 欄位 | 值 |
   | ----- | ----- |
   | 選取資料夾 | 選擇開發機器上可供 VS Code 建立解決方案檔案的位置。 |
   | 提供解決方案名稱 | 輸入解決方案的描述性名稱或接受預設值 **EdgeSolution**。 |
   | 選取模組範本 | 選擇 [C# 模組]  。 |
   | 提供模組名稱 | 將模組命名為 **CSharpModule**。 |
   | 提供模組的 Docker 映像存放庫 | 映像存放庫包含容器登錄名稱和容器映像名稱。 您的容器映像會從您在上一個步驟中提供的名稱預先填入。 將 **localhost:5000** 取代為 Azure Container Registry 的 **登入伺服器** 值。 您可以在 Azure 入口網站中，從容器登錄的概觀頁面擷取登入伺服器。 <br><br>最終的映像存放庫看起來類似於：\<registry name\>.azurecr.io/csharpmodule。 |

   ![提供 Docker 映像存放庫](./media/tutorial-csharp-module/repository.png)

### <a name="add-your-registry-credentials"></a>新增登錄認證

環境檔案會儲存容器登錄的認證，並與 IoT Edge 執行階段共用這些認證。 執行階段需要有這些認證才能將私人映像提取到 IoT Edge 裝置。 使用您 Azure 容器登錄的 [存取金鑰]  區段中的認證。

IoT Edge 擴充功能會嘗試從 Azure 提取您的容器登錄認證，並將這些認證填入到環境檔案中。 請查看您的認證是否已包含其中。 如果沒有，請立即新增：

1. 在 VS Code 總管中，開啟 **.env** 檔案。
2. 使用 Azure 容器登錄中的 **使用者名稱** 和 **密碼** 值來更新欄位。
3. 儲存這個檔案。

### <a name="select-your-target-architecture"></a>選取您的目標架構

目前，Visual Studio Code 可以開發適用於 Linux AMD64 和 Linux ARM32v7 裝置的 C# 模組。 您必須為每個解決方案都選取要作為目標的架構，因為容器是針對每個架構類型，以不同方式建置和執行。 預設值為 Linux AMD64。

1. 開啟命令選擇區並搜尋 **Azure IoT Edge:Set Default Target Platform for Edge Solution**，或選取視窗底部側邊欄的捷徑圖示。

2. 在命令選擇區中，從選項清單中選取目標架構。 針對此教學課程，我們是使用 Ubuntu 虛擬機器作為 IoT Edge 裝置，因此會保留預設 **amd64**。

### <a name="update-the-module-with-custom-code"></a>使用自訂程式碼來更新模組

1. 請在 VS Code 總管中，開啟 [modules] > [CSharpModule] > [Program.cs]。

2. 在 **CSharpModule** 命名空間頂端，為稍後會用到的類型新增三個 **using** 陳述式：

    ```csharp
    using System.Collections.Generic;     // For KeyValuePair<>
    using Microsoft.Azure.Devices.Shared; // For TwinCollection
    using Newtonsoft.Json;                // For JsonConvert
    ```

3. 將 **temperatureThreshold** 變數新增至 [Program] 類別。 此變數會設定在將資料傳送至 IoT 中樞之前，測量的溫度必須超過的值。

    ```csharp
    static int temperatureThreshold { get; set; } = 25;
    ```

4. 將 [MessageBody]、[Machine] 和 [Ambient] 類別新增至 [Program] 類別。 這些類別會定義內送郵件本文的預期結構描述。

    ```csharp
    class MessageBody
    {
        public Machine machine {get;set;}
        public Ambient ambient {get; set;}
        public string timeCreated {get; set;}
    }
    class Machine
    {
        public double temperature {get; set;}
        public double pressure {get; set;}
    }
    class Ambient
    {
        public double temperature {get; set;}
        public int humidity {get; set;}
    }
    ```

5. 尋找 **Init** 函式。 此函式會建立並設定 **ModuleClient** 物件，其可讓模組連線至本機 Azure IoT Edge 執行階段，以便傳送和接收訊息。 在建立 **ModuleClient** 之後，程式碼會從模組對應項所需的屬性中讀取 **temperatureThreshold** 值。 程式碼會註冊回撥，以透過 **input1** 端點接收來自 IoT Edge 中樞的訊息。 將 **SetInputMessageHandlerAsync** 方法取代為新的方法，並將更新的 **SetDesiredPropertyUpdateCallbackAsync** 方法新增至所需的屬性。 若要進行這項變更，請使用下列程式碼取代 **Init** 方法的最後一行：

    ```csharp
    // Register a callback for messages that are received by the module.
    // await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", PipeMessage, iotHubModuleClient);

    // Read the TemperatureThreshold value from the module twin's desired properties
    var moduleTwin = await ioTHubModuleClient.GetTwinAsync();
    await OnDesiredPropertiesUpdate(moduleTwin.Properties.Desired, ioTHubModuleClient);

    // Attach a callback for updates to the module twin's desired properties.
    await ioTHubModuleClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertiesUpdate, null);

    // Register a callback for messages that are received by the module.
    await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FilterMessages, ioTHubModuleClient);
    ```

6. 將 **onDesiredPropertiesUpdate** 方法新增至 [Program] 類別。 此方法會從模組對應項接收所需的屬性，並會更新 **temperatureThreshold** 變數以符合該屬性。 所有模組都具有自己的模組對應項，這可讓您直接從雲端設定於模組內執行的程式碼。

    ```csharp
    static Task OnDesiredPropertiesUpdate(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            Console.WriteLine("Desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));

            if (desiredProperties["TemperatureThreshold"]!=null)
                temperatureThreshold = desiredProperties["TemperatureThreshold"];

        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error when receiving desired property: {0}", exception);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error when receiving desired property: {0}", ex.Message);
        }
        return Task.CompletedTask;
    }
    ```

7. 使用 **FilterMessages** 方法取代 **PipeMessage** 方法。 每當模組從 IoT Edge 中樞接收到訊息時，就會呼叫此方法。 它會篩選所報告溫度低於 (透過模組對應項所設定) 之溫度閾值的訊息。 針對具有設定為 [警示]  之值的訊息，此方法也會將 **MessageType** 屬性新增至該訊息。

    ```csharp
    static async Task<MessageResponse> FilterMessages(Message message, object userContext)
    {
        var counterValue = Interlocked.Increment(ref counter);
        try
        {
            ModuleClient moduleClient = (ModuleClient)userContext;
            var messageBytes = message.GetBytes();
            var messageString = Encoding.UTF8.GetString(messageBytes);
            Console.WriteLine($"Received message {counterValue}: [{messageString}]");

            // Get the message body.
            var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

            if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
            {
                Console.WriteLine($"Machine temperature {messageBody.machine.temperature} " +
                    $"exceeds threshold {temperatureThreshold}");
                using (var filteredMessage = new Message(messageBytes))
                {
                    foreach (KeyValuePair<string, string> prop in message.Properties)
                    {
                        filteredMessage.Properties.Add(prop.Key, prop.Value);
                    }

                    filteredMessage.Properties.Add("MessageType", "Alert");
                    await moduleClient.SendEventAsync("output1", filteredMessage);
                }
            }

            // Indicate that the message treatment is completed.
            return MessageResponse.Completed;
        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error in sample: {0}", exception);
            }
            // Indicate that the message treatment is not completed.
            var moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
            // Indicate that the message treatment is not completed.
            ModuleClient moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
    }
    ```

8. 儲存 Program.cs 檔案。

9. 在 VS Code 總管中，於 IoT Edge 解決方案工作區中開啟 **deployment.template.json** 檔案。

10. 在部署資訊清單中新增 **CSharpModule** 模組對應項。 在 **modulesContent** 區段底部，於 **$edgeHub** 模組對應項後面插入下列 JSON 內容：

    ```json
       "CSharpModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
    ```

    ![將模組對應項新增至部署範本](./media/tutorial-csharp-module/module-twin.png)

11. 儲存 deployment.template.json 檔案。

## <a name="build-and-push-your-module"></a>建置和推送模組

在上一節中，您已建立 IoT Edge 解決方案，並將程式碼新增至 CSharpModule。 新的程式碼會篩選出回報的機器溫度在可接受限制內的訊息。 現在，您需要建置容器映像形式的解決方案，並將它推送到容器登錄。

1. 選取 [檢視]   > [終端機]  ，以開啟 VS Code 整合式終端機。

1. 在終端機中輸入下列命令來登入 Docker。 使用您 Azure 容器登錄中的使用者名稱、密碼和登入伺服器登入。 您可以在 Azure 入口網站中，從登錄的 [存取金鑰]  區段擷取這些值。

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   您可能會收到安全性警告，建議您使用 `--password-stdin`。 雖然建議生產案例使用該最佳做法，但是不在本教學課程的討論範圍內。 如需詳細資訊，請參閱 [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin) 參考。

1. 在 VS Code 總管中，以滑鼠右鍵按一下 **deployment.template.json** 檔案，然後選取 [建置並推送 IoT Edge 解決方案]。

   建置和推送命令會啟動三項作業。 首先，它會在解決方案中建立名為 **config** 的新資料夾，以保存完整部署資訊清單 (根據部署範本中的資訊建立)，以及其他解決方案檔案。 接著，它會執行 `docker build`，以根據目標架構的適當 dockerfile 建置容器映像。 然後，它會執行 `docker push` 以將映像存放庫推送至您的容器登錄。

   此程序第一次進行時可能需要幾分鐘的時間，但下一次執行命令時速度就會變快。

## <a name="deploy-and-run-the-solution"></a>部署並執行解決方案

使用 Visual Studio Code 總管和 Azure IoT Tools 擴充功能，將模組專案部署到您的 IoT Edge 裝置。 您已備妥您的案例所需的部署資訊清單，即 config 資料夾中的 **deployment.amd64.json** 檔案。 現在您只需選取要接收部署的裝置即可。

請確定您的 IoT Edge 裝置已啟動並執行。

1. 在 Visual Studio Code 總管中，展開 [Azure IoT 中樞] 區段下的 [裝置] 來查看您的 IoT 裝置清單。

2. 以滑鼠右鍵按一下 IoT Edge 裝置的名稱，然後選取 [建立單一裝置的部署]。

3. 選取 **config** 資料夾中的 **deployment.amd64.json** 檔案，然後按一下 [選取 Edge 部署資訊清單]。 請勿使用 deployment.template.json 檔案。

4. 請展開裝置下的 **模組**，以查看已部署且執行中的模組清單。 按一下 [重新整理] 按鈕。 您應該會看到新的 **CSharpModule** 正在與 **SimulatedTemperatureSensor** 模組以及 **$edgeAgent** 和 **$edgeHub** 一起執行。

    模組可能需要幾分鐘才會啟動。 IoT Edge 執行階段需要接收其新的部署資訊清單、從容器執行階段提取模組映像，然後啟動每個新的模組。

## <a name="view-generated-data"></a>檢視產生的資料

在您將部署資訊清單套用至 IoT Edge 裝置後，裝置的 IoT Edge 執行階段即會收集新的部署資訊，並開始在裝置上執行。 裝置上任何執行中、但未包含在部署資訊清單中的模組都會停止。 裝置中遺漏的任何模組都會啟動。

1. 在 Visual Studio Code 總管中，以滑鼠右鍵按一下 IoT Edge 裝置的名稱，然後選取 [開始監視內建事件端點]  。

2. 檢視送達 IoT 中樞的訊息。 訊息可能需要一段時間才能送達，因為 IoT Edge 裝置必須接收其新的部署和啟動所有模組。 然後，我們對 CModule 程式碼所做的變更會等到機器溫度達到 25 度時才會傳送訊息。 它也會將 [警示] 訊息類型新增至任何觸達該溫度閾值的訊息。

   ![檢視送達 IoT 中樞的訊息](./media/tutorial-csharp-module/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>編輯模組對應項

我們已使用部署資訊清單中的 CSharpModule 模組對應項，設定 25 度的溫度閾值。 您可以使用模組對應項來變更此功能，而不必更新模組程式碼。

1. 在 Visual Studio Code 中，展開您 IoT Edge 裝置底下的詳細資料，以查看執行中的模組。

2. 以滑鼠右鍵按一下 **CSharpModule** 並選取 [編輯模組對應項]。

3. 在所需的屬性中尋找 **TemperatureThreshold**。 將其值變更為高於最近報告溫度 5 到 10 度的新溫度。

4. 儲存模組對應項檔案。

5. 以滑鼠右鍵按一下模組對應項編輯窗格中的任意位置，然後選取 [更新模組對應項]  。

6. 監視傳入的裝置到雲端訊息。 在新的溫度閾值送達前，您應該會看到訊息停止。

## <a name="clean-up-resources"></a>清除資源

如果您打算繼續閱讀下一篇建議的文章，則可以保留您所建立的資源和組態，並加以重複使用。 您可以也繼續使用相同的 IoT Edge 裝置作為測試裝置。

否則，可以刪除您在本文中使用的本機設定和 Azure 資源，以避免產生費用。

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立包含程式碼的 IoT Edge 模組，可篩選您 IoT Edge 裝置所產生的原始資料。

您可以繼續進行後續教學課程，以了解 Azure IoT Edge 如何協助您部署 Azure 雲端服務，以在邊緣處理和分析資料。

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [串流分析](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [自訂視覺服務](tutorial-deploy-custom-vision.md)