---
title: v4 ボットで JavaScript v3 ユーザー状態を使用する | Microsoft Docs
description: v4 ボットで v3 ユーザー状態を使用する方法の例
keywords: JavaScript, ボットの移行, v3 ボット
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/14/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4697ecf47464114de68ec6c0d872b45ff1ee5e54
ms.sourcegitcommit: d493caf74b87b790c99bcdaddb30682251e3fdd4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/26/2019
ms.locfileid: "71278981"
---
<!-- This article is on hold -->

# <a name="using-javascript-v3-user-state-in-a-v4-bot"></a>v4 ボットで JavaScript v3 ユーザー状態を使用する

この記事では、v4 ボットで v3 ユーザー状態情報の読み取り、書き込み、および削除の各操作を実行する方法の例を示します。

[こちら](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot)にサンプル コードがあります。

> [!NOTE]
> ボットは、**会話の状態**を維持し、会話を追跡および制御して、ユーザーに質問します。 **ユーザーの状態**を維持して、ユーザーの回答を追跡します。

## <a name="prerequisites"></a>前提条件

- npm バージョン 6.9.0 以降 (パッケージのエイリアス化をサポートするために必要)。

- Node.js バージョン 10.14.1 以降。

    コンピューターにインストールされているバージョンを確認するには、端末で次のコマンドを実行します。

    ```bash
    # determine node version
    node --version
    ```

## <a name="setup"></a>セットアップ

1. リポジトリの複製

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. 端末で、`BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot` に移動します。

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot
    ```

1. 次の場所で `npm install` を実行します。

    ```bash
    root
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. 次の場所で ``npm run build`` または ``tsc`` を実行して、`StorageMapper` モジュールと `UserState` モジュールをビルドします。

    ```bash
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. データベースの構成

    1. `.env.example` ファイルの内容をコピーします。
    1. `.env` という名前の新しいファイルを作成し、前の内容をそこに貼り付けます。 
    1. ストレージ プロバイダーに関する値を入力します。
        Azure portal の特定のストレージ プロバイダー (*Cosmos DB*、*Table Storage*、*SQL Database* など) のセクションで、*ユーザー名*、*パスワード*、および*ホスト情報*を見つけることができます。 テーブル名とコレクション名はユーザー定義です。
  
1. ボットのストレージ プロバイダーの設定

    1. プロジェクトのルートにある `index.js` ファイルを開きます。 ファイルの最初の方 (おおよそ 38 から 98 行) に、コメントに示されているように、各ストレージ プロバイダーの構成があります。 ノード `process.env` を介して `.env` ファイルから構成値が読み込まれます。 次のコード スニペットは、SQL Database を構成する方法を示しています。

        [!code-javascript[Storage configuration](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=77-92)]

    1. 選択したストレージ クライアント インスタンスを`StorageMapper` アダプターに渡すことで (おおよそ 107 行目)、ボットでどのストレージ プロバイダーを使用するかを指定します。  

        [!code-javascript[StorageMapper](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=105-107)]

        既定の設定は *Cosmos DB* です。 指定できる値は、

        ```bash
            cosmosStorageClient
            tableStorage
            sqlStorage
        ```

1. アプリケーションを起動します。 プロジェクトのルートから、次のコマンドを実行します。

    ```bash
    npm run start
    ```

## <a name="adapter-classes"></a>アダプター クラス

### <a name="v4v3storagemapper"></a>V4V3StorageMapper

`StorageMapper` クラスには、アダプターの主要な機能が含まれます。 このクラスにより、v4 ストレージ インターフェイスが実装され、ストレージ プロバイダーのメソッド (読み取り、書き込み、および削除) が v3 ストレージ プロバイダーのクラスにマップされます。これにより、v3 形式のユーザー状態を v4 ボットから使用できます。

### <a name="v4v3userstate"></a>V4V3UserState

このクラスは、v3 形式のキーを使用するために v4 `BotState` クラス (`botbuilder-core`) を拡張します。これにより、v3 ストレージに対して読み取り、書き込み、および削除を行うことができます。

## <a name="testing-the-bot-using-bot-framework-emulator"></a>Bot Framework Emulator を使用したボットのテスト

[Bot Framework Emulator][5] は、ローカルホストで、またはトンネルを介したリモート実行でボットをテストおよびデバッグできるデスクトップ アプリケーションです。

- [こちら][6]から、Bot Framework Emulator バージョン 4.3.0 以降をインストールします。

### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Bot Framework Emulator を使用したボットへの接続

1. Bot Framework Emulator を起動します。
1. 次の URL (エンド ポイント) を入力します: `http://localhost:3978/api/messages`。

### <a name="testing-steps"></a>テスト手順

1. エミュレーターでボットを開き、メッセージを送信します。 プロンプトが表示されたら、名前を入力します。
1. ターンが終了した後、ボットに別のメッセージを送信します。
1. 名前の入力を求めるプロンプトが再表示されないことを確認します。 ボットは、ストレージから読み取りを行い、既にプロンプトを表示したということを認識するはずです。
1. ボットは、メッセージをエコー バックします。
1. Azure のストレージ プロバイダーにアクセスし、名前がユーザー データとしてデータベースに格納されていることを確認します。

## <a name="deploy-the-bot-to-azure"></a>Azure へのボットのデプロイ

Azure へのボットのデプロイについて詳しくは、「[ボットを Azure にデプロイする][40]」で、詳細なデプロイ手順を参照してください。

## <a name="further-reading"></a>参考資料

- [Azure Bot Service のドキュメント][21]
- [ボットの状態][7]
- [ストレージに直接書き込む][8]
- [会話とユーザー状態を管理する][9]
- [Restify][30]
- [dotenv][31]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=javascript
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=javascript
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[30]: https://www.npmjs.com/package/restify
[31]: https://www.npmjs.com/package/dotenv
[40]: https://aka.ms/azuredeployment
