---
title: Web チャットで音声認識を有効にする - Bot Service
description: Web チャット チャンネルに接続されたボットの Web チャット コントロールで音声認識を有効にする方法について説明します。
keywords: 音声認識, web チャット, 音声, マイク, オーディオ
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 72a247fe0e8373323626a5d01360d2a923b09b09
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75793249"
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

   | パラメーター | [説明] |
   |-----------|-------------|
   | s | Direct Line シークレット。 Direct Line シークレットを取得する方法については、「[Connect a bot to Direct Line (ボットを Direct Line に接続する)](bot-service-channel-connect-directline.md)」をご覧ください。 |
   | t | Direct Line トークン。 このトークンを生成する方法については、「[Generate a Direct Line token (Direct Line トークンを生成する)](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)」をご覧ください。 |
   | domain | 省略可能。 代替 Direct Line エンドポイントの URL。  |
   | webSocket | 省略可能。 WebSocket を使用してメッセージを受信するには、"true" に設定します。 既定値は `false` です。 |
   | userid | 省略可能。 ボット ユーザーのID。  |
   | username | 省略可能。 ボット ユーザーのユーザー名。  |
   | botid | 省略可能。 ボットの ID。 |
   | botname | 省略可能。 ボットの名前。 |


## <a name="enable-speech-services"></a>音声認識サービスを有効にする
カスタマイズにより、次のいずれかの方法で音声認識機能を追加できます。

* **ブラウザーが提供する音声認識** - Web ブラウザーに組み込まれた音声認識機能を使用します。 現時点では、この機能は Chrome ブラウザーでのみ使用できます。
<!--* **Use Bing Speech service** - You can use the Bing Speech service to provide speech recognition and synthesis. This way of access speech functionality is supported by a variety of browsers. In this case, the processing is done on a server instead of on the browser.-->
* **カスタム音声認識サービスの作成** - 独自のカスタム音声認識コンポーネントと音声合成コンポーネントを作成できます。

### <a name="browser-provided-speech"></a>ブラウザーが提供する音声認識

次のコードでは、ブラウザーに搭載された音声認識エンジンと音声合成コンポーネントをインスタンス化します。 音声認識のこの追加方法は、すべてのブラウザーでサポートされているわけではありません。 

> [!NOTE] 
> Google Chrome では、ブラウザーの音声認識エンジンがサポートされています。 ただし、Chrome では、次の場合にマイクがブロックされることがあります。
> * Web チャットを含むページの URL が、`https://` ではなく `http://` で始まる場合。
> * URL が、`http://localhost:8000` ではなく `file://` プロトコルを使用するローカル ファイルの場合。

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

<!--### Bing Speech service

The following code instantiates speech recognizer and speech synthesis components that use the Bing Speech service. The recognition and generation of speech is performed on the server. This mechanism is supported in multiple browsers. 

> [!TIP]
> You can use speech recognition priming to improve your bot's speech recognition accuracy if you use the Bing Speech service. For more information, check out the [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text) blog post.

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### Use the Bing Speech service with a token

You also have the option to enable Cognitive Services speech recognition using a token. The token is generated in a secure back end using your API key.

The following example code shows how the token fetch is done from a secure back end to avoid exposing the API key.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]
-->
### <a name="custom-speech-service"></a>カスタム音声認識サービス

ISpeechRecognizer を実装した独自のカスタム音声認識や、ISpeechSynthesis を実装したカスタム音声合成を提供することもできます。 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>音声認識オプションを Web チャットに渡す

次のコードでは、音声認識オプションを Web チャット コントロールに渡します。

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>次のステップ
これで、Web チャットとの音声対話を有効にすることができました。次に、ボットで音声メッセージを作成し、マイクの状態を調整する方法を確認しましょう。
* [メッセージへの音声の追加 (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [メッセージへの音声の追加 (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>その他のリソース

* GitHub で Web チャット コントロールの[ソース コードをダウンロード](https://github.com/Microsoft/BotFramework-WebChat)できます。
* Bing Speech API の詳細については、[Bing Speech API ドキュメント](https://docs.microsoft.com/azure/cognitive-services/speech/home)をご覧ください。

