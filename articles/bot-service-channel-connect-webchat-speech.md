---
title: Web チャットで音声認識を有効にする | Microsoft Docs
description: Web チャット チャンネルに接続されたボットの Web チャット コントロールで音声認識を有効にする方法について説明します。
keywords: 音声認識, web チャット, 音声, マイク, オーディオ
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 563fdd480ac5165d5301faa3ed43af3118d2f7d7
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707478"
---
# <a name="enable-speech-in-web-chat"></a>Web チャットで音声認識を有効にする
Web チャット コントロールで音声インターフェイスを有効にすることができます。 ユーザーは、Web チャット コントロールのマイクを使って音声インターフェイスと対話します。

![Web チャットの音声認識のサンプル](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

ユーザーが音声で応答する代わりに、応答を入力すると、Web チャットが音声認識機能をオフにし、ボットは音声ではなくテキストでのみ応答します。 音声による応答を再度有効にするには、次回ボットに応答するときにユーザーがマイクを使います。 マイクが入力を受け入れている場合、黒または塗りつぶしで表示されます。 マイクが淡色表示されている場合は、ユーザーがマイクをクリックして有効にします。

## <a name="prerequisites"></a>前提条件

  サンプルを実行するには、Web チャット コントロールを使用して実行するボットの Direct Line シークレットまたはトークンが必要です。 
  * ボットに関連付けられた Direct Line シークレットを取得する方法については、「[Connect a bot to Direct Line (ボットを Direct Line に接続する)](bot-service-channel-connect-directline.md)」をご覧ください。
  * シークレットをトークンと交換する方法については、「[Generate a Direct Line token (Direct Line トークンを生成する)](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)」をご覧ください。

## <a name="customizing-web-chat-for-speech"></a>音声認識に対応するために Web チャットをカスタマイズする
Web チャットで音声認識機能を有効にするには、Web チャット コントロールを呼び出す JavaScript コードをカスタマイズする必要があります。 次の手順に従って、音声対応 Web チャットをローカルで試すことができます。

1. [サンプル index.html](https://aka.ms/web-chat-speech-sample) をダウンロードします。 <!-- this aka.ms link needs to be updated if the sample location changes -->
2. 追加する音声認識サポートの種類に応じて、`index.html` 内のコードを編集します。 音声認識実装の種類については、「[音声認識サービスを有効にする](#enable-speech-services)」をご覧ください。 
3. Web サーバーを起動します。 これを実行する 1 つの方法として、Node.js コマンド プロンプトで `npm http-server` を使用します。

   * コマンド ラインから実行できるように、`http-server` をグローバルにインストールするには、次のコマンドを実行します。

     ```
     npm install http-server -g
     ```

   * ポート 8000 を使用して Web サーバーを起動するには、`index.html` が格納されているディレクトリから、次のコマンドを実行します。

     ```
     http-server -p 8000
     ```
4. ブラウザーで `http://localhost:8000/samples?parameters` を参照します。 たとえば、`http://localhost:8000/samples?s=YOURDIRECTLINESECRET` は、Direct Line シークレットを使用してボットを呼び出します。 クエリ文字列に設定できるパラメーターを次の表に示します。

   | パラメーター | 説明 |
   |-----------|-------------|
   | s | Direct Line シークレット。 Direct Line シークレットを取得する方法については、「[Connect a bot to Direct Line (ボットを Direct Line に接続する)](bot-service-channel-connect-directline.md)」をご覧ください。 |
   | t | Direct Line トークン。 このトークンを生成する方法については、「[Generate a Direct Line token (Direct Line トークンを生成する)](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)」をご覧ください。 |
   | domain | 省略可能。 代替 Direct Line エンドポイントの URL。  |
   | webSocket | 省略可能。 WebSocket を使用してメッセージを受信するには、"true" に設定します。 既定値は `false`です。 |
   | userid | 省略可能。 ボット ユーザーのID。  |
   | username | 省略可能。 ボット ユーザーのユーザー名。  |
   | botid | 省略可能。 ボットの ID。 |
   | botname | 省略可能。 ボットの名前。 |


## <a name="enable-speech-services"></a>音声認識サービスを有効にする
カスタマイズにより、次のいずれかの方法で音声認識機能を追加できます。

* **ブラウザーが提供する音声認識** - Web ブラウザーに組み込まれた音声認識機能を使用します。 現時点では、この機能は Chrome ブラウザーでのみ使用できます。
* **Bing Speech Service の使用** - Bing Speech Service を使用して、音声認識と音声合成を提供できます。 音声認識機能へのこのアクセス方法は、さまざまなブラウザーでサポートされています。 この場合、ブラウザー上ではなく、サーバー上で処理が行われます。
* **カスタム音声認識サービスの作成** - 独自のカスタム音声認識コンポーネントと音声合成コンポーネントを作成できます。

### <a name="browser-provided-speech"></a>ブラウザーが提供する音声認識

次のコードでは、ブラウザーに搭載された音声認識エンジンと音声合成コンポーネントをインスタンス化します。 音声認識のこの追加方法は、すべてのブラウザーでサポートされているわけではありません。 

> [!NOTE] 
> Google Chrome では、ブラウザーの音声認識エンジンがサポートされています。 ただし、Chrome では、次の場合にマイクがブロックされることがあります。
> * Web チャットを含むページの URL が、`https://` ではなく `http://` で始まる場合。
> * URL が、`http://localhost:8000` ではなく `file://` プロトコルを使用するローカル ファイルの場合。

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

### <a name="bing-speech-service"></a>Bing Speech Service

次のコードでは、Bing Speech Service を使用する音声認識エンジンと音声合成コンポーネントをインスタンス化します。 音声の認識と生成はサーバー上で実行されます。 このメカニズムは複数のブラウザーでサポートされています。 

> [!TIP]
> Bing Speech Service を使用する場合、音声認識の準備を使用してボットの音声認識の精度を向上させることができます。 詳細については、[Bot Framework における音声認識サポート](https://blog.botframework.com/2017/06/26/Speech-To-Text)に関するブログ記事をご覧ください。

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### <a name="use-the-bing-speech-service-with-a-token"></a>トークンを使って Bing Speech Service を使用する

トークンを使用して、Cognitive Services の音声認識を有効にすることもできます。 トークンは、API キーを使用して、セキュリティで保護されたバックエンドで生成されます。

次のコード例は、API キーが公開されないように、セキュリティで保護されたバックエンドからトークンを取得する方法を示しています。

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]

### <a name="custom-speech-service"></a>カスタム音声認識サービス

ISpeechRecognizer を実装した独自のカスタム音声認識や、ISpeechSynthesis を実装したカスタム音声合成を提供することもできます。 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>音声認識オプションを Web チャットに渡す

次のコードでは、音声認識オプションを Web チャット コントロールに渡します。

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>次の手順
これで、Web チャットとの音声対話を有効にすることができました。次に、ボットで音声メッセージを作成し、マイクの状態を調整する方法を確認しましょう。
* [メッセージへの音声の追加 (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [メッセージへの音声の追加 (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>その他のリソース

* GitHub で Web チャット コントロールの[ソース コードをダウンロード](https://github.com/Microsoft/BotFramework-WebChat)できます。
* Bing Speech API の詳細については、[Bing Speech API ドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech/home)をご覧ください。

