---
title: ボットをデバッグする - Bot Service
description: Bot Service を使用して作成されたボットをデバッグする方法について説明します。
author: v-ducvo
ms.author: kamrani
keywords: Bot Framework SDK, ボットのデバッグ, ボットのテスト, ボット エミュレーター, エミュレーター
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 999c664f78d11e67647f9efa28bc7f059c43882b
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250121"
---
# <a name="debug-a-bot"></a>ボットをデバッグする

この記事では、Visual Studio や Visual Studio Code などの統合開発環境 (IDE) と Bot Framework Emulator を使用して、ボットをデバッグする方法について説明します。 これらの方法を使用してローカルで任意のボットをデバッグできますが、この記事ではクイック スタートで作成された [C# ボット](~/dotnet/bot-builder-dotnet-sdk-quickstart.md)または [Javascript ボット](~/javascript/bot-builder-javascript-quickstart.md)を使います。

> [!NOTE]
> この記事では、Bot Framework Emulator を使用して、デバッグ中にボットとメッセージを送受信します。 Bot Framework Emulator を使用して、他の方法でボットをデバッグするには、「[Bot Framework Emulator を使用したデバッグ](https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator)」の記事を参照してください。 

## <a name="prerequisites"></a>前提条件 
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をダウンロードしてインストールします。
- [Visual Studio Code](https://code.visualstudio.com) または [Visual Studio](https://www.visualstudio.com/downloads) (Community Edition 以上) をダウンロードしてインストールします。

<!-- ### Debug a JavaScript bot using command-line and emulator

To run a JavaScript bot using the command line and testing the bot with the emulator, do the following:
1. From the command line, change directory to your bot project directory.
1. Start the bot by running the command **node app.js**.
1. Start the emulator and connect to the bot's endpoint (e.g.: **http://localhost:3978/api/messages**). If this is the first time you are running 
the bot then click **File > New Bot** and follow the instructions on screen. Otherwise, click **File > Open Bot** to open an existing bot. 
Since this bot is running locally on your computer, you can leave the **MicrosoftAppId** and **MicrosoftAppPassword** fields blank. 
For more information, see [Debug with the Emulator](bot-service-debug-emulator.md).
1. From the emulator, send your bot a message (e.g.: send the message "Hi"). 
1. Use the **Inspector** and **Log** panels on the right side of the emulator window to debug your bot. For example, clicking on any of the messages bubble (e.g.: the "Hi" message bubble in the screenshot below) will show you the detail of that message in the **Inspector** panel. You can use it to view requests and responses as messages are exchanged between the emulator and the bot. Alternatively, you can click on any of the linked text in the **Log** panel to view the details in the **Inspector** panel.


   ![Inspector panel on the Emulator](~/media/bot-service-debug-bot/emulator_inspector.png) -->

## <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Visual Studio Code でブレークポイントを使用して JavaScript のボットをデバッグする

Visual Studio Code では、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS Code でブレークポイントを設定するには、次の手順のようにします。

1. VS Code を起動し、ボット プロジェクト フォルダーを開きます。
2. メニュー バーの **[デバッグ]** をクリックし、 **[デバッグの開始]** をクリックします。 コードを実行するランタイム エンジンの選択を求められた場合は、**Node.js** を選択します。 この時点で、ボットはローカルに実行されています。 
3. **.js** ファイルをクリックし、必要に応じてブレークポイントを設定します。 VS Code では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。

   ![VS Code でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set.png)

4. 「[Bot Framework Emulator を使用したデバッグ](https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator)」の記事の説明に従って、Bot Framework Emulator を起動してボットに接続します。 
5. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS Code でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught.png)

## <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Visual Studio でブレークポイントを使用して C# のボットをデバッグする

Visual Studio (VS) では、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS でブレークポイントを設定するには、次の手順のようにします。

1. ボット フォルダーに移動し、 **.sln** ファイルを開きます。 これにより VS でソリューションが開きます。
2. メニュー バーの **[ビルド]** をクリックし、 **[ソリューションのビルド]** をクリックします。
3. **ソリューション エクスプローラー**で、 **.cs** ファイルをクリックし、必要に応じてブレークポイントを設定します。 このファイルでは、ボットのメイン ロジックが定義されています。 VS では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 このドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。
4. メニューの **[デバッグ]** をクリックし、 **[デバッグの開始]** をクリックします。 この時点で、ボットはローカルに実行されています。 

   ![VS でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->

5. Bot Framework Emulator を開始し、前のセクションで説明したようにボットに接続します。 
6. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0"

## <a name="debug-a-consumption-plan-c-functions-bot"></a><a id="debug-csharp-serverless"></a> 従量課金プランの C\# Functions ボットをデバッグする

Bot Service での従量課金プランのサーバーレス C\# 環境は、Node エンジンとよく似たランタイム ホストを必要とするため、一般的な C\# アプリケーションより、Node.js との方が共通点があります。 Azure では、ランタイムはクラウド内のホスティング環境の一部ですが、その環境をデスクトップ上にローカルにレプリケートする必要があります。 

### <a name="prerequisites"></a>前提条件

従量課金プランの C# ボットをデバッグする前に、次のタスクを完了する必要があります。

- ボットのソース コードを (Azure から) ダウンロードします (「[Set up continuous deployment](bot-service-continuous-deployment.md)」(継続的デプロイの設定) を参照)。
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をダウンロードしてインストールします。
- <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">Azure Functions CLI</a> をインストールします。
- <a href="https://github.com/dotnet/cli" target="_blank">DotNet CLI</a> をインストールします。
  
Visual Studio 2017 でブレークポイントを使用してコードをデバッグしたい場合は、次のタスクも完了する必要もあります。
  
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition 以上) をダウンロードしてインストールします。
- <a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">Command Task Runner Visual Studio 拡張機能</a>をダウンロードしてインストールします。

> [!NOTE]
> 現在、Visual Studio Code はサポートされていません。

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>エミュレーターを使用して従量課金プラン C# Functions ボットをデバッグする

ボットをローカルでデバッグする最も簡単な方法は、ボットを開始し、Bot Framework Emulator からボットに接続することです。 
最初に、コマンド プロンプトを開き、リポジトリで **project.json** ファイルがあるフォルダーに移動します。 次に、`dotnet restore` コマンドを実行して、ボットで参照されているさまざまなパッケージを復元します。

> [!NOTE]
> Visual Studio 2017 では、Visual Studio による依存関係の処理方法が変更されています。 Visual Studio 2015 では **project.json** を使って依存関係を処理しますが、Visual Studio 2017 では Visual Studio に読み込みときに **.csproj** モデルを使います。 Visual Studio 2017 を使っている場合は、リポジトリ内の **/messages** フォルダーにこの [ **.csproj** ファイル](https://aka.ms/v3-dotnet-debug-csproj)をダウンロードしてから、`dotnet restore` コマンドを実行します。

![コマンド プロンプト](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

次に、`debughost.cmd` を実行し、ボットを読み込んで開始します。 

![コマンド プロンプトで debughost.cmd を実行する](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

この時点で、ボットはローカルに実行されています。 コンソール ウィンドウで、debughost がリッスンしているエンドポイント (この例では `http://localhost:3978`) をコピーします。 次に、Bot Framework Emulator を開始し、エミュレーターのアドレス バーにエンドポイントを貼り付けます。 この例では、エンドポイントに `/api/messages` を追加する必要もあります。 ローカル デバッグではセキュリティは必要ないので、 **[Microsoft App ID]\(Microsoft アプリ ID\)** フィールドと **[Microsoft App Password]\(Microsoft アプリ パスワード\)** フィールドは空白のままでかまいません。 **[Connect]\(接続\)** をクリックし、指定したエンドポイントを使用してボットへの接続を確立します。

![エミュレーターを構成する](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

エミュレーターをボットに接続した後、エミュレーター ウィンドウの下部にあるテキスト ボックス (つまり、左下隅の **[Type your message...]\(メッセージを入力...\)** と表示されている場所) にテキストを入力して、ボットにメッセージを送信します。 エミュレーター ウィンドウの右側の **[Log]\(ログ\)** パネルと **[Inspector]\(インスペクター\)** パネル使って、エミュレーターとボットの間でメッセージが交換されるときの要求と応答を確認できます。

![エミュレーターでテストする](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

さらに、コンソール ウィンドウでログの詳細を表示できます。

![コンソール ウィンドウ](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="debug-a-python--bot-using-breakpoints-in-visual-studio-code"></a>Visual Studio Code でブレークポイントを使用して Python のボットをデバッグする

Visual Studio Code では、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 「[Bot Framework SDK for Python を使用したボットの作成](~/python/bot-builder-python-quickstart.md)」も参照してください。

1. VS Code を起動し、ボット プロジェクト フォルダーを開きます。
1. 必要に応じてブレークポイントを設定します。 ブレークポイントを設定するには、行番号の左側の列にマウス カーソルを合わせます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。
1. `app.py` を選択します。
1. メニュー バーの **[デバッグ]** をクリックし、 **[デバッグの開始]** をクリックします。
1. 現在選択されているファイルをデバッグするには、 **[Python ファイル]** を選択します。

   ![ブレークポイントを設定する](~/media/bot-service-debug-bot/bot-debug-python-breakpoints.png)

1. 「[Bot Framework Emulator を使用したデバッグ](https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator)」の記事の説明に従って、Bot Framework Emulator を起動してボットに接続します。 
1. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS Code でデバッグする](~/media/bot-service-debug-bot/bot-debug-python-breakpoint-caught.png)

詳細については、「[Python コードのデバッグ](https://aka.ms/bot-debug-python)」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [一般的な問題のトラブルシューティング](bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。
- [Emulator を使用してデバッグする](bot-service-debug-emulator.md)方法をご覧ください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [トランスクリプト ファイルを使用してお使いのボットをデバッグする](v4sdk/bot-builder-debug-transcript.md)。
