---
title: Bot Service を使用して作成されたボットをデバッグする | Microsoft Docs
description: Bot Service を使用して作成されたボットをデバッグする方法について説明します。
author: v-ducvo
ms.author: v-ducvo
keywords: Bot Builder SDK, 継続的デプロイ, App Service, エミュレーター
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/13/2018
ms.openlocfilehash: 73cf6cedecd9acaef828bd41f4fccdaad2ae5731
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707168"
---
# <a name="debug-a-bot-service-bot"></a>Bot Service のボットをデバッグする

この記事では、Visual Studio や Visual Studio Code などの統合開発環境 (IDE) と Bot Framework Emulator を使用して、ボットをデバッグする方法について説明します。 これらの方法を使用して任意のボットをローカルにデバッグできますが、この記事では、記事「[Create a bot with Bot Service](bot-service-quickstart.md)」(Bot Service を使用してボットを作成する) で作成される **EchoBot** を使います。

## <a name="debug-a-javascript-bot"></a>JavaScript のボットをデバッグする

JavaScript で記述されたボットをデバッグするには、このセクションの手順のようにします。

### <a name="prerequisites"></a>前提条件

JavaScript のボットをデバッグする前に、次のタスクを完了する必要があります。

- [ボットのソース コードのダウンロード](bot-service-build-download-source-code.md)に関するページの説明のとおり、(Azure から) ボットのソース コードをダウンロードします。
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) をダウンロードしてインストールします。
- <a href="https://code.visualstudio.com" target="_blank">Visual Studio Code</a> などのコード エディターをダウンロードしてインストールします。

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>コマンド ラインとエミュレーターを使用して JavaScript のボットをデバッグする

コマンド ラインを使用して JavaScript のボットを実行し、エミュレーターでボットをテストするには、次のようにします。
1. コマンド ラインで、ボット プロジェクトのディレクトリに移動します。
2. **node app.js** コマンドを実行してボットを開始します。
3. エミュレーターを起動し、ボットのエンドポイントに接続します (例: **http://localhost:3978/api/messages**)。 初めてボットを実行する場合は、**[File]\(ファイル\) > [New Bot]\(新しいボット\)** をクリックして、画面の指示に従います。 それ以外の場合は、**[File]\(ファイル\) > [Open Bot]\(ボットを開く\)** をクリックして既存のボットを開きます。 このボットはお使いのコンピューターでローカルに実行されているため、**[MSA app ID]\(MSA アプリ ID\)** と **[MSA app password]\(MSA アプリ パスワード\)** フィールドは空白のままでかまいません。 詳細については、[エミュレーターを使用したデバッグ](bot-service-debug-emulator.md)に関するページをご覧ください。
4. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 
5. エミュレーター ウィンドウの右側にある **[Inspector]\(インスペクター\)** パネルと **[Log]\(ログ\)** パネルを使用して、ボットをデバッグします。 たとえば、いずれかのメッセージ バブル (下のスクリーンショットでは "Hi" メッセージ バブル) をクリックすると、そのメッセージの詳細が **[Inspector]\(インスペクター\)** パネルに表示されます。 これを使って、エミュレーターとボットの間でメッセージが交換されるときに、要求と応答を確認できます。 または、**[Log]\(ログ\)** パネルでいずれかのリンクされたテキストをクリックすると、**[Inspector]\(インスペクター\)** パネルに詳細が表示されます。

   ![エミュレーターの [Inspector]\(インスペクター\) パネル](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Visual Studio Code でブレークポイントを使用して JavaScript のボットをデバッグする

Visual Studio Code などの IDE を使うと、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS Code でブレークポイントを設定するには、次の手順のようにします。

1. VS Code を起動し、ボット プロジェクト フォルダーを開きます。
2. メニュー バーの **[デバッグ]** をクリックし、**[デバッグの開始]** をクリックします。 コードを実行するランタイム エンジンの選択を求められた場合は、**Node.js** を選択します。 この時点で、ボットはローカルに実行されています。 

   > [!NOTE]
   > "値を null 値にすることはできません" というエラーが発生する場合は、**Table Storage** の設定が有効であることを確認します。
   > **EchoBot** は既定で **Table Storage** を使います。 ボットで Table Storage を使うには、テーブルの "*名前*" と "*キー*" が必要です。 Table Storage インスタンスの準備ができていない場合は、テスト用に Table Storage を作成するか、または **TableBotDataStore** を使うコードをコメント化し、**InMemoryDataStore** を使うコード行をコメント解除してもかまいません。 **InMemoryDataStore** は、テストおよびプロトタイプ作成だけが対象です。

3. 必要に応じて、ブレークポイントを設定します。 VS Code では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。

   ![VS Code でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Bot Framework Emulator を開始し、前のセクションで説明したようにボットに接続します。 
5. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS Code でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught.png)


## <a name="debug-a-c-bot"></a>C# のボットをデバッグする

C# で記述されたボットをデバッグするには、このセクションの手順のようにします。


### <a name="prerequisites"></a>前提条件

Web アプリの C# ボットをデバッグする前に、次のタスクを完了する必要があります。

- [ボットのソース コードのダウンロード](bot-service-build-download-source-code.md)に関するページの説明のとおり、(Azure から) ボットのソース コードをダウンロードします。
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) をダウンロードしてインストールします。
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition 以上) をダウンロードしてインストールします。

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Visual Studio でブレークポイントを使用して C# のボットをデバッグする

Visual Studio (VS) などの IDE を使うと、ブレークポイントを設定し、デバッグ モードでボットを実行して、コードをステップ実行できます。 VS でブレークポイントを設定するには、次の手順のようにします。

1. ボット フォルダーに移動し、**.sln** ファイルを開きます。 これにより VS でソリューションが開きます。
2. メニュー バーの **[ビルド]** をクリックし、**[ソリューションのビルド]** をクリックします。
3. **ソリューション エクスプローラー**で **[Dialogs]** フォルダーを展開し、**EchoDialog.cs** をクリックします。 このファイルでは、ボットのメイン ロジックが定義されています。
4. メニュー バーの **[デバッグ]** をクリックし、**[デバッグの開始]** をクリックします。 この時点で、ボットはローカルに実行されています。 

   > [!NOTE]
   > "値を null 値にすることはできません" というエラーが発生する場合は、**Table Storage** の設定が有効であることを確認します。
   > **EchoBot** は既定で **Table Storage** を使います。 ボットで Table Storage を使うには、テーブルの "*名前*" と "*キー*" が必要です。 Table Storage インスタンスの準備ができていない場合は、テスト用に Table Storage を作成するか、または **TableBotDataStore** を使うコードをコメント化し、**InMemoryDataStore** を使うコード行をコメント解除してもかまいません。 **InMemoryDataStore** は、テストおよびプロトタイプ作成だけが対象です。

5. 必要に応じて、ブレークポイントを設定します。 VS では、行番号の左側に列をマウスでポイントすることによって、ブレークポイントを設定できます。 小さな赤いドットが表示されます。 ドットをクリックすると、ブレークポイントが設定されます。 もう一度ドットをクリックすると、ブレークポイントが削除されます。

   ![VS でブレークポイントを設定する](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

6. Bot Framework Emulator を開始し、前のセクションで説明したようにボットに接続します。 
7. エミュレーターからボットにメッセージを送信します (例: "Hi" というメッセージを送信)。 ブレークポイントを設定した行で、実行が停止します。

   ![VS でデバッグする](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> 従量課金プランの C\# Functions ボットをデバッグする

Bot Service での従量課金プランのサーバーレス C\# 環境は、Node エンジンとよく似たランタイム ホストを必要とするため、一般的な C\# アプリケーションより、Node.js との方が共通点があります。 Azure では、ランタイムはクラウド内のホスティング環境の一部ですが、その環境をデスクトップ上にローカルにレプリケートする必要があります。 

### <a name="prerequisites"></a>前提条件

従量課金プランの C# ボットをデバッグする前に、次のタスクを完了する必要があります。

- ボットのソース コードを (Azure から) ダウンロードします (「[Set up continuous deployment](bot-service-continuous-deployment.md)」(継続的デプロイの設定) を参照)。
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) をダウンロードしてインストールします。
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

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [エミュレーターを使用したデバッグ](bot-service-debug-emulator.md)。
