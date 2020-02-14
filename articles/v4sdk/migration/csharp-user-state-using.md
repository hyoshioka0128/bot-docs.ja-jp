---
title: v4 ボットで .NET v3 ユーザー状態を使用する - Bot Service
description: v4 ボットで v3 ユーザー状態を使用する方法の例
keywords: Csharp, ボットの移行, v3 ボット
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/21/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 51aef13bee896feeb901040cef641f7b1011dbd5
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798085"
---
# <a name="using-net-v3-user-state-in-a-v4-bot"></a>v4 ボットで .NET v3 ユーザー状態を使用する

この記事では、v4 ボットで v3 ユーザー状態情報の読み取り、書き込み、および削除の各操作を実行する方法の例を示します。
ボットは、ユーザーに質問する間、`MemoryStorage` を使用して会話の状態を維持し、会話を追跡および制御します。  `V3V4Storage` というカスタムの `IStorage` クラスを使用して、v3 形式で**ユーザー状態**を保持し、ユーザーの回答を追跡します。  このクラスの引数の 1 つは `IBotDataStore` です。 v3 SDK コード ベースが `Bot.Builder.Azure.V3V4` にコピーされており、3 つすべての v3 SDK ストレージ プロバイダー (Azure SQL、Azure Table、Cosmos DB) が含まれています。  その目的は、既存の v3 **ユーザー状態**を移行済みの v4 ボットに取り込めるようにすることです。

[こちら](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/V4StateBotFromV3Providers)にサンプル コードがあります。

## <a name="prerequisites"></a>前提条件

- [.NET Core SDK](https://dotnet.microsoft.com/download) バージョン 2.1

    ```bash
    # determine dotnet version
    dotnet --version
    ```

## <a name="setup"></a>セットアップ

1. リポジトリの複製

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. 端末で、`MigrationV3V4/CSharp/V4StateBotFromV3Providers` に移動します。

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers
    ```

1. 端末または Visual Studio からボットを実行し、オプション A または B を選択します。

    - 端末から

        ```bash
        # run the bot
        dotnet run
        ```

    - Visual Studio から

        - Visual Studio を起動し、[ファイル] -> [開く] -> [プロジェクト/ソリューション] を選択します。
        - `BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers` フォルダーに移動します。
        - `V4StateBot.sln` ファイルを選択します。
        - F5 キーを押して、プロジェクトを実行します。


## <a name="storage-provider-setup"></a>ストレージ プロバイダーのセットアップ

既存の v3 状態ストアが構成され、使用されていることを前提とします。 この場合、次に示すように、この例では既存の状態ストアを使用するように設定するときに、ストレージ プロバイダーの接続情報を `web.config` ファイルに追加するだけですみます。

- Cosmos DB

```json
  "v3CosmosEndpoint": "https://yourcosmosdb.documents.azure.com:443/",
  "v3CosmosKey": "YourCosmosDbKey",
  "v3CosmosDatataseName": "v3botdb",
  "v3CosmosCollectionName": "v3botcollection",
```

- Azure SQL

```json
 "ConnectionStrings": {
    "SqlBotData": "Server=YourServer;Initial Catalog=BotData;Persist Security Info=False;User ID=YourUserName;Password=YourUserPassword;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;"
  },
```

- Azure テーブル

```json
 "ConnectionStrings": {
    "AzureTable": "DefaultEndpointsProtocol=https;AccountName=YourAccountName;AccountKey=YourAccountKey;EndpointSuffix=core.windows.net"
  },
```

- ボットのストレージ プロバイダーの設定

    `V4V3StateBot` プロジェクトのルートにある `Startup.cs` ファイルを開きます。 ファイルの中間部分 (おおよそ 52 から 76 行) に、各ストレージ プロバイダーの構成があります。 web.config から構成値が読み込まれます。 

    [!code-csharp[Storage configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=52-76)]

    選択したインスタンスの対応する行をコメント解除することで、ボットでどのストレージ プロバイダーを使用するかを指定します。 プロバイダーが適切に構成された後、プロバイダー クラスが `V3V4Storage` に渡されることを確認します (おおよそ 72 から 75 行)。 

    [!code-csharp[Storage provider](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=72-75)]

    Cosmos DB (以前の Document DB) は既定で設定されています。 指定できる値は、

    ```bash
    documentDbBotDataStore
    tableBotDataStore
    tableBotDataStore2
    sqlBotDataStore
    ```

- アプリケーションを起動します。 

## <a name="v3v4-storage-and-state-classes"></a>V3V4 ストレージ クラスと状態クラス

### <a name="v3v4storage"></a>V3V4Storage

`V3V4Storage` クラスには、主要なストレージ マッピング機能が含まれます。 このクラスにより、v4 `IStorage` インターフェイスが実装され、ストレージ プロバイダーのメソッド (読み取り、書き込み、および削除) が v3 ストレージ プロバイダーのクラスにマップされます。これにより、v3 形式のユーザー状態を v4 ボットから使用できます。

### <a name="v3v4state"></a>V3V4State

このクラスは、v4 `BotState` クラスを継承しており、v3 形式のキー (`IAddress`) を使用します。 これにより、v3 状態ストレージの通常の操作方法と同じ方法で、v3 ストレージに対して読み取り、書き込み、および削除の各操作を実行できます。


## <a name="testing-the-bot-using-bot-framework-emulator"></a>Bot Framework Emulator を使用したボットのテスト

[Bot Framework Emulator][5] は、ボット開発者がローカルホストで、またはトンネルを介したリモート実行でボットをテストおよびデバッグできるデスクトップ アプリケーションです。

- [こちら][6]から、Bot Framework Emulator バージョン 4.3.0 以降をインストールします。


### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Bot Framework Emulator を使用したボットへの接続

- Bot Framework Emulator を起動します。
- [File] -> [Open Bot]\(ボットを開く\) を選択します。
- ボットの URL として `http://localhost:3978/api/messages` を入力します。


## <a name="further-reading"></a>関連項目

- [Azure Bot Service のドキュメント][21]
- [ボットの状態][7]
- [ストレージに直接書き込む][8]
- [会話とユーザー状態を管理する][9]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=csharp
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=csharp
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[40]: https://aka.ms/azuredeployment
