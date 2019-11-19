---
title: 検査ミドルウェアを使用してボットをデバッグする | Microsoft Docs
description: 検査ミドルウェアを使用してボットをデバッグする方法について説明します
author: zxyanliu
ms.author: v-liyan
keywords: Bot Framework SDK, ボットのデバッグ, 検査ミドルウェア, ボット エミュレーター, Azure Bot Channels Registration
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: 0e59c6d3548e273a8fb164526ddeb6ba66f48e3e
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933525"
---
# <a name="debug-a-bot-with-inspection-middleware"></a>検査ミドルウェアを使用してボットをデバッグする
この記事では、検査ミドルウェアを使用してボットをデバッグする方法について説明します。 この機能を使用すると、Bot Framework Emulator で、ボットの現在の状態を確認でき、さらにボットとの間のトラフィックをデバッグできます。 トレース メッセージを使用してエミュレーターにデータを送信し、会話のある特定のターンでのボットの状態を調べることができます。 

Bot Framework v4 ([C#](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) | [JavaScript](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)) を使用してローカルに構築された EchoBot を使用して、ボットのメッセージの状態をデバッグおよび検査する方法を示します。 また、[IDE を使用してボットをデバッグ](./bot-service-debug-bot.md)したり、[Bot Framework Emulator を使用してデバッグ](./bot-service-debug-emulator.md)したりすることもできますが、状態をデバッグするには、検査ミドルウェアをボットに追加する必要があります。 検査ボットのサンプルを入手できます:[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) および [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 

## <a name="prerequisites"></a>前提条件
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) をダウンロードしてインストールする
- ボットの[ミドルウェア](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)の知識
- ボットの[管理状態](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0)の知識
- [ngrok](https://ngrok.com/) をダウンロードしてインストールする (追加のチャネルを使用するために、Azure に構成されているボットをデバッグする場合)

## <a name="update-your-emulator-to-the-latest-version"></a>エミュレーターを最新バージョンに更新する 
ボット検査ミドルウェアを使用してボットをデバッグする前に、エミュレーターをバージョン 4.5 以降に更新する必要があります。 [最新バージョン](https://github.com/Microsoft/BotFramework-Emulator/releases)で更新プログラムを確認してください。 

エミュレーターのバージョンを確認するには、メニューで **[ヘルプ]**  ->  **[バージョン情報]** の順に選択します。 お使いのエミュレーターの現在のバージョンが表示されます。 

![現在のバージョン](./media/bot-debug-inspection-middleware/bot-debug-check-emulator-version.png) 

## <a name="update-your-bots-code"></a>ボットのコードを更新する

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
**Startup** ファイルで検査状態を設定します。 検査ミドルウェアをアダプターに追加します。 検査状態は、依存関係の挿入によって提供されます。 以下のコード更新をご覧になるか、こちらの検査サンプルを参照してください:[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection)。 

**Startup.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Startup.cs?range=17-37)]

**AdapterWithInspection.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/AdapterwithInspection.cs?range=11-37)]

**EchoBot.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Bots/EchoBot.cs?range=14-43)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
ボットのコードを更新する前に、ターミナルで次のコマンドを実行して、パッケージを最新バージョンに更新する必要があります。 
```cmd
npm install --save botbuilder@latest 
```  
次に、JavaScript ボットのコードを次のように更新する必要があります。 詳細については、次を参照してください。「[Update your bot's code](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md#1-update-your-bots-code)」(ボットのコードを更新する) または以下の検査サンプルを参照してください:[JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 

**index.js**

検査状態を設定し、**index.js** ファイル内のアダプターに検査ミドルウェアを追加します。 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/index.js?range=10-43)]

**bot.js**

**bot.js** ファイル内のボット クラスを更新します。 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/bot.js?range=6-50)]

---

## <a name="test-your-bot-locally"></a>ボットをローカルでテストする 
コードを更新した後、ボットをローカルで実行し、2 つのエミュレーター (1 つはメッセージを送受信するためのもので、もう 1 つはデバッグ モードでメッセージの状態を検査するためのもの) を使用してデバッグ機能をテストできます。 ボットをローカルでテストするには、次の手順を実行します。 

1. ターミナルでボットのディレクトリに移動し、次のコマンドを実行してボットをローカルで実行します。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
dotnet run
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
npm start 
```

---

2. エミュレーターを開きます。 **[Open Bot]\(ボットを開く\)** をクリックします。 [Bot URL]\(ボットの URL\) に「 http://localhost:3978/api/messages 」を入力し、**MicrosoftAppId** と **MicrosoftAppPassword** の値を入力します。 JavaScript ボットの場合、これらの値はボットの **.env** ファイルにあります。 C# ボットの場合、これらの値は **appsettings.json** ファイルにあります。 **[接続]** をクリックします。 

3. 次に、別のエミュレーターを開きます。 この 2 つ目のエミュレーターは、デバッガーとして機能します。 前の手順で説明した指示に従います。 **[Open in debug mode]\(デバッグ モードで開く\)** をオンにし、 **[接続]** をクリックします。 

4. この時点で、デバッグ エミュレーターに UUID (`/INSPECT attach <identifier>`) が表示されます。 UUID をコピーし、最初のエミュレーターのチャット ボックスに貼り付けます。 

> [!NOTE]
> 汎用一意識別子 (UUID) は、情報を識別するための一意の ID です。 UUID は、ボットのコードに検査ミドルウェアを追加した後、デバッグ モードでエミュレーターを起動するたびに生成されます。 

5. これで、最初のエミュレーターのチャット ボックスでメッセージを送信し、デバッグ エミュレーターでメッセージを検査できるようになりました。 メッセージの状態を確認するには、デバッグ エミュレーターで **[Bot State]\(ボットの状態\)** をクリックし、右側の **JSON** ウィンドウで**値**を展開します。 ボットの状態は次のように表示されます。![ボットの状態](./media/bot-debug-inspection-middleware/bot-debug-bot-state.png)

## <a name="inspect-the-state-of-a-bot-configured-in-azure"></a>Azure に構成されたボットの状態を検査する 
Azure に構成され、チャネル (Teams など) に接続されているボットの状態を調べるには、[ngrok](https://ngrok.com/) をインストールして実行する必要があります。

### <a name="run-ngrok"></a>ngrok を実行する
これで、エミュレーターが最新バージョンに更新され、ボットのコードに検査ミドルウェアが追加されました。 次の手順では、ngrok を実行し、ローカル ボットを　Azure Bot Channels Registration に対して構成します。 ngrok を実行する前に、ボットをローカルで実行する必要があります。 

ボットをローカルで実行するには、次の操作を行います。 
1. ターミナルでボットのフォルダーに移動し、[最新のビルド](https://botbuilder.myget.org/feed/botbuilder-v4-js-daily/package/npm/botbuilder-azure)を使用するように npm 登録を設定します。

2. ボットをローカルで実行します。 ボットで 3978 のようなポート番号が公開されるのを確認できます。 

3. 別のコマンド プロンプトを開き、ボットのプロジェクト フォルダーに移動します。 次のコマンドを実行します。
```
ngrok http 3978
```
4. これで、ローカルに実行されているボットに ngrok が接続されました。 パブリック IP アドレスをコピーします。 
![ngrok 成功](./media/bot-debug-inspection-middleware/bot-debug-ngrok.png)

### <a name="update-channel-registrations-for-your-bot"></a>ボットのチャネル登録を更新する
ローカル ボットが ngrok に接続されたので、ローカル ボットを Azure の Bot Channels Registration に構成できます。

1. Azure の Bot Channels Registration に移動します。 左側メニューの **[設定]** をクリックし、ngrok IP を使用して **[Messaging endpoint]\(メッセージング エンドポイント\)** を設定します。 必要な場合、IP アドレスの後に **/api/messages** を追加します。 (たとえば、 https://e58549b6.ngrok.io/api/messages) 。 **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** をオンにして、 **[保存]** を選択します。
![endpoint](./media/bot-debug-inspection-middleware/bot-debug-channels-setting-ngrok.png)
> [!TIP]
> **[保存]** が有効になっていない場合は、 **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** をオフにして **[保存]** をクリックしてから、再び **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** をオンにして **[保存]** をクリックできます。 **[Enable Streaming Endpoint]\(ストリーミング エンドポイントを有効にする\)** がオンになっており、エンドポイントの構成が保存されていることを確認する必要があります。 

2. ボットのリソース グループにアクセスし、 **[デプロイ]** をクリックして、以前正常にデプロイされた Bot Channels Registration を選択します。 左側で **[入力]** をクリックして、**appId** と **appSecret** を取得します。 **appId** と **appSecret** を使用して、ボットの **.env** ファイル (C# ボットを使用している場合は **appsettings.json** ファイル) を更新します。 
![入力の取得](./media/bot-debug-inspection-middleware/bot-debug-get-inputs-id-secret.png)

3. エミュレーターを開始し、 **[Open Bot]\(ボットを開く\)** をクリックし、 **[Bot URL]\(ボットの URL\)** に「 http://localhost:3978/api/messages 」を入力します。 **[Microsoft App ID]\(Microsoft アプリ ID\)** と **[Microsoft App password]\(Microsoft アプリ パスワード\)** に、ボットの **.env** (**appsettings.json**) ファイルに追加したのと同じ **appId** と **appSecret** を入力します。 次いで **[接続]** をクリックします。 

4. これで、実行中のボットが Azure の Bot Channels Registration に接続されました。 Web チャットをテストするには、 **[Test in Web Chat]\(Web チャットでのテスト\)** をクリックし、チャット ボックスでメッセージを送信します。 
![Web チャットのテスト](./media/bot-debug-inspection-middleware/bot-debug-test-webchat.png)

5. 次に、エミュレーターでデバッグ モードを有効にします。 エミュレーターで、 **[デバッグ]**  ->  **[デバッグの開始]** を選択します。 **[Bot URL]\(ボットの URL\)** に ngrok の IP アドレスを入力します。 **/api/messages** を忘れずに追加してください (たとえば、 https://e58549b6.ngrok.io/api/messages) 。 **[Microsoft App ID]\(Microsoft アプリ ID\)** に **appId** を入力し、 **[Microsoft App password]\(Microsoft アプリ パスワード\)** に **appSecret** を入力します。 また、 **[Open in debug mode]\(デバッグ モードで開く\)** がオンになっていることも確認します。 **[接続]** をクリックします。 

6. デバッグ モードが有効になっていると、UUID がエミュレーターで生成されます。 UUID は、エミュレーターでデバッグ モードを開始するたびに生成される一意の ID です。 UUID をコピーして **[Test in Web Chat]\(Web チャットでのテスト\)** チャット ボックス、またはお使いのチャネルのチャット ボックスに貼り付けます。 チャット ボックスに、"セッションに接続しました。すべてのトラフィックは検査のためにレプリケートされています (Attached to session, all traffic is being replicated for inspection)" というメッセージが表示されます。 

 構成済みチャネルのチャット ボックスでメッセージを送信することにより、ボットのデバッグを開始できます。 ローカル エミュレーターは、デバッグのためのすべての詳細情報でメッセージを自動的に更新します。 ボットのメッセージの状態を検査するには、 **[Bot State]\(ボットの状態\)** をクリックし、右側の JSON ウィンドウの**値**を展開します。 

 ![デバッグ、検査ミドルウェア](./media/bot-debug-inspection-middleware/debug-state-inspection-channel-chat.gif)

## <a name="additional-resources"></a>その他のリソース
- こちらの新しい検査ボット サンプルをお試しください:[C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) および [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection)。 
- [一般的な問題のトラブルシューティング](bot-service-troubleshoot-bot-configuration.md)に関する記事、およびそのセクションにあるその他のトラブルシューティングに関する記事をお読みください。
- 「[エミュレーターを使用したデバッグ](bot-service-debug-emulator.md)」の記事をお読みください。
