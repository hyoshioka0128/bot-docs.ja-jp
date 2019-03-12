---
title: ボットをデバッグする | Microsoft Docs
description: Bot Service を使用して作成されたボットをデバッグする方法について説明します。
author: v-ducvo
ms.author: v-ducvo
keywords: Bot Framework SDK, ボットのデバッグ, ボットのテスト, ボット エミュレーター, エミュレーター
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/26/2019
ms.openlocfilehash: 1e806358ffdba90848f0c8124c8315fd4e2dec76
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224840"
---
# <a name="debug-a-bot"></a>ボットをデバッグする

この記事では、Visual Studio や Visual Studio Code などの統合開発環境 (IDE) と Bot Framework Emulator を使用して、ボットをデバッグする方法について説明します。 これらの方法を使用してローカルで任意のボットをデバッグできますが、この記事ではクイック スタートで作成された [C#](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) および [JS](~/javascript/bot-builder-javascript-quickstart.md) ボットを使います。

## <a name="prerequisites"></a>前提条件 
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をダウンロードしてインストールします。
- [Visual Studio Code](https://code.visualstudio.com) または [Visual Studio](https://www.visualstudio.com/downloads) (Community Edition 以上) をダウンロードしてインストールします。

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>コマンドラインとエミュレーターを使用して JavaScript のボットをデバッグする

コマンド ラインを使用して JavaScript のボットを実行し、エミュレーターでボットをテストするには、次のようにします。
1. コマンド ラインで、ボット プロジェクトのディレクトリに移動します。
1. **node app.js** コマンドを実行してボットを開始します。
1. エミュレーターを起動し、ボットのエンドポイントに接続します (例: **http://localhost:3978/api/messages**)。 初めてボットを実行する場合は、**[File]\(ファイル\) > [New Bot]\(新しいボット\)** をクリックして、画面の指示に従います。 それ以外の場合は、**[File]\(ファイル\) > [Open Bot]\(ボットを開く\)** をクリックして既存のボットを開きます。 このボットはお使いのコンピューターでローカルに実行されているため、**[MSA app ID]\(MSA アプリ ID\)** と **[MSA app password]\(MSA アプリ パスワード\)** フィールドは空白のままでかまいません。 詳細については、[エミュレーターを使用したデバッグ](bot-service-debug-emulator.md)に関するページをご覧ください。
1. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 
1. エミュレーター ウィンドウの右側にある **[Inspector]\(インスペクター\)** パネルと **[ログ]** パネルを使用して、ボットをデバッグします。 たとえば、いずれかのメッセージ バブル (下のスクリーンショットでは "Hi" メッセージ バブル) をクリックすると、そのメッセージの詳細が **[Inspector]\(インスペクター\)** パネルに表示されます。 これを使って、エミュレーターとボットの間でメッセージが交換されるときに、要求と応答を確認できます。 または、**[ログ]** パネルでいずれかのリンクされたテキストをクリックすると、**[Inspector]\(インスペクター\)** パネルに詳細が表示されます。


   ![エミュレーターの [Inspector]\(インスペクター\) パネル](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Visual Studio Code でブレークポイントを使用して JavaScript のボットをデバッグする

Visual Studio Code では、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS Code でブレークポイントを設定するには、次の手順のようにします。

1. VS Code を起動し、ボット プロジェクト フォルダーを開きます。
2. メニュー バーの **[デバッグ]** をクリックし、**[デバッグの開始]** をクリックします。 コードを実行するランタイム エンジンの選択を求められた場合は、**Node.js** を選択します。 この時点で、ボットはローカルに実行されています。 
<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->
3. 必要に応じて、ブレークポイントを設定します。 VS Code では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。

   ![VS Code でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Bot Framework Emulator を開始し、前のセクションで説明したようにボットに接続します。 
5. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS Code でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught.png)

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Visual Studio でブレークポイントを使用して C# のボットをデバッグする

Visual Studio (VS) では、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS でブレークポイントを設定するには、次の手順のようにします。

1. ボット フォルダーに移動し、**.sln** ファイルを開きます。 これにより VS でソリューションが開きます。
2. メニュー バーの **[ビルド]** をクリックし、**[ソリューションのビルド]** をクリックします。
3. **ソリューション エクスプローラー**で、**[EchoWithCounterBot.cs]** をクリックします。 このファイルでは、必要に応じて、メイン ボットの logic.Set ブレークポイントが定義されます。 VS では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。
5. メニュー バーの **[デバッグ]** をクリックし、**[デバッグの開始]** をクリックします。 この時点で、ボットはローカルに実行されています。 

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->

   ![VS でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

7. Bot Framework Emulator を開始し、前のセクションで説明したようにボットに接続します。 
8. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> 従量課金プランの C\# Functions ボットをデバッグする

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
> Visual Studio 2017 では、Visual Studio による依存関係の処理方法が変更されています。 Visual Studio 2015 では **project.json** を使って依存関係を処理しますが、Visual Studio 2017 では Visual Studio に読み込みときに **.csproj** モデルを使います。 Visual Studio 2017 を使っている場合は、リポジトリ内の **/messages** フォルダーに<a href="https://aka.ms/bf-debug-project">この **.csproj** ファイルをダウンロード</a>した後で、`dotnet restore` コマンドを実行します。

![コマンド プロンプト](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

次に、`debughost.cmd` を実行し、ボットを読み込んで開始します。 

![コマンド プロンプトで debughost.cmd を実行する](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

この時点で、ボットはローカルに実行されています。 コンソール ウィンドウで、debughost がリッスンしているエンドポイント (この例では `http://localhost:3978`) をコピーします。 次に、Bot Framework Emulator を開始し、エミュレーターのアドレス バーにエンドポイントを貼り付けます。 この例では、エンドポイントに `/api/messages` を追加する必要もあります。 ローカル デバッグではセキュリティは必要ないので、**[Microsoft App ID]\(Microsoft アプリ ID\)** フィールドと **[Microsoft App Password]\(Microsoft アプリ パスワード\)** フィールドは空白のままでかまいません。 **[Connect]\(接続\)** をクリックし、指定したエンドポイントを使用してボットへの接続を確立します。

![エミュレーターを構成する](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

エミュレーターをボットに接続した後、エミュレーター ウィンドウの下部にあるテキスト ボックス (つまり、左下隅の **[Type your message...]\(メッセージを入力...\)** と表示されている場所) にテキストを入力して、ボットにメッセージを送信します。 エミュレーター ウィンドウの右側の **[Log]\(ログ\)** パネルと **[Inspector]\(インスペクター\)** パネル使って、エミュレーターとボットの間でメッセージが交換されるときの要求と応答を確認できます。

![エミュレーターでテストする](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

さらに、コンソール ウィンドウでログの詳細を表示できます。

![コンソール ウィンドウ](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="additional-resources"></a>その他のリソース

- [一般的な問題のトラブルシューティング](bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションに示されているトラブルシューティングに関するその他の記事をご覧ください。
- [ngrok を使用してチャネルをローカルでデバッグする](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/)方法に関するブログ記事をご覧ください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [トランスクリプト ファイルを使用してお使いのボットをデバッグする](v4sdk/bot-builder-debug-transcript.md)。
