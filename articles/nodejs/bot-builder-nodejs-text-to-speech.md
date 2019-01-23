---
title: メッセージに音声を追加する | Microsoft Docs
description: Bot Framework SDK for Node.js を使用してメッセージに音声を追加する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f7e68b9ab6ef1fca189108ed4117c0ab17f4d9f2
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224297"
---
# <a name="add-speech-to-messages"></a>メッセージに音声を追加する

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana などの音声対応チャネルのボットを作成している場合は、ボットが読み上げるテキストを指定するメッセージを構成できます。 また、[入力ヒント](bot-builder-nodejs-send-input-hints.md)を指定して、ボットがユーザー入力を受け入れるか、期待するか、無視するかを示し、クライアントのマイクの状態に影響を与えることを試すこともできます。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>ボットが読み上げるテキストを指定する

Bot Framework SDK for Node.js を使用すると、音声対応チャネルでボットが読み上げるテキストを指定する方法は複数あります。 `IMessage.speak` プロパティを設定し、`session.send()` メソッド、`session.say()` メソッド (表示テキスト、音声テキスト、オプションを指定するパラメーターを渡す)、または組み込みプロンプト (オプション `speak` と `retrySpeak` を指定する) を使用してメッセージを送信することができます。

### <a id="message-speak"></a> IMessage.speak 

`session.send()` メソッドを使用して送信されるメッセージを作成している場合は、`speak` プロパティを設定して、ボットが読み上げるテキストを指定します。 次のコード例では、読み上げるテキストを指定し、ボットが[ユーザー入力を受け付けている](bot-builder-nodejs-send-input-hints.md)ことを示すメッセージを作成します。

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

`session.send()` の使用に代わって `session.say()` メソッドを呼び出して、表示するテキストとその他のオプションだけでなく、読み上げるテキストも指定するメッセージを作成し、送信できます。 メソッドの定義は次のとおりです。

`session.say(displayText: string, speechText: string, options?: object)`

| パラメーター | 説明 |
|----|----|
| `displayText` | 表示されるテキスト。 |
| `speechText` | 読み上げるテキスト (プレーンテキストまたは <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 形式)。 |
| `options` | 添付ファイルや[入力ヒント](bot-builder-nodejs-send-input-hints.md)を含めることができる [IMessage][IMessage] オブジェクト。 |

次のコード例では、表示するテキストと読み上げるテキストを指定し、ボットが[ユーザー入力を無視している](bot-builder-nodejs-send-input-hints.md)ことを示すメッセージを送信します。

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> プロンプト オプション

任意の組み込みプロンプトを使用すると、オプション `speak` と `retrySpeak` を設定し、ボットで読み上げるテキストを指定できます。 次のコード例では、表示するテキスト、最初に読み上げるテキスト、ユーザーの入力をしばらく待ってから読み上げるテキストを指定するプロンプトを作成します。 これは、ボットが[ユーザー入力を期待している](bot-builder-nodejs-send-input-hints.md)ことを示しており、[SSML](#ssml) 書式設定を使用して、単語 "sure" を適度な強調で発音するように指定します。

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a> 音声合成マークアップ言語 (SSML)

ボットで読み上げるテキストを指定するには、プレーンテキスト文字列または音声合成マークアップ言語 (SSML) として書式設定された文字列のいずれかを使用できます。SSML は XML ベースのマークアップ言語であり、声、速度、音量、発音、声の高さなどボットの音声のさまざまな特性を制御できます。 SSML の詳細については、「<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language Reference (音声合成マークアップ言語リファレンス)</a>」を参照してください。

> [!TIP]
> <a href="https://www.npmjs.com/search?q=ssml" target="_blank">SSML ライブラリ</a>を使用して、正しい形式の SSML を作成します。

## <a name="input-hints"></a>入力ヒント

音声対応チャネルでメッセージを送信する場合、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すための入力ヒントを含めることによっても、クライアントのマイクの状態に影響を与えることを試すことができます。 詳細については、「[メッセージへの入力ヒントの追加](bot-builder-nodejs-send-input-hints.md)」を参照してください。

## <a name="sample-code"></a>サンプル コード 

Bot Framework SDK for .NET を使用して音声対応ボットを作成する方法を示す完全なサンプルについては、GitHub の「<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">Roller sample (Roller のサンプル)</a>」を参照してださい。

## <a name="additional-resources"></a>その他のリソース

- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">音声合成マークアップ言語 (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">Roller のサンプル (GitHub)</a>
- [Bot Framework SDK for Node.js リファレンス][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
