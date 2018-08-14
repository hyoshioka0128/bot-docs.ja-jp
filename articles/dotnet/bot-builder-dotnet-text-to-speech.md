---
title: メッセージへの音声の追加 | Microsoft Docs
description: Bot Builder SDK for .NET を使用してメッセージに音声を追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0c972b913ab0639363a1e2f1307bcaa4bea60c54
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303836"
---
# <a name="add-speech-to-messages"></a>メッセージへの音声の追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana などの音声対応チャネルのボットを作成している場合は、ボットが読み上げるテキストを指定するメッセージを構成できます。 また、[入力ヒント](bot-builder-dotnet-add-input-hints.md)を指定して、ボットがユーザー入力を受け入れるか、期待するか、無視するかを示し、クライアントのマイクの状態に影響を与えることを試すこともできます。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>ボットが読み上げるテキストの指定

Bot Builder SDK for .NET を使用すると、音声対応チャネルでボットが読み上げるテキストを指定する方法は複数あります。 [メッセージ][IMessageActivity]の `Speak` プロパティを設定することや、`IDialogContext.SayAsync()` メソッドを呼び出すこと、組み込みプロンプトを使用してメッセージを送信する場合はプロンプト オプション `speak` および `retrySpeak` を指定することができます。

### <a id="message-speak"></a> IMessageActivity.Speak

[メッセージ][IMessageActivity]を作成して個々のプロパティを設定している場合は、メッセージの `Speak` プロパティを設定してボットが読み上げるテキストを指定できます。 次のコード例では、表示するテキストと読み上げるテキストを指定し、ボットが[ユーザー入力](bot-builder-dotnet-add-input-hints.md)を受け入れていることを示すメッセージを作成します。

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

[ダイアログ](bot-builder-dotnet-dialogs.md)を使用している場合、表示するテキストとその他のオプションに加え、読み上げるテキストを指定するメッセージを作成して送信するために `SayAsync()` メソッドを呼び出すことができます。 次のコード例では、表示するテキストと読み上げるテキストを指定するメッセージを作成します。

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> プロンプト オプション

任意の組み込みプロンプトを使用すると、オプション `speak` と `retrySpeak` を設定し、ボットで読み上げるテキストを指定できます。 次のコード例では、表示するテキスト、最初に読み上げるテキスト、ユーザーの入力をしばらく待ってから読み上げるテキストを指定するプロンプトを作成します。 これには [SSML](#ssml) 形式を使用し、単語 "sure" を適度な強調で読み上げる必要があることを示します。

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a> 音声合成マークアップ言語 (SSML)

ボットで読み上げるテキストを指定するには、プレーンテキスト文字列または音声合成マークアップ言語 (SSML) として書式設定された文字列のいずれかを使用できます。SSML は XML ベースのマークアップ言語であり、声、速度、音量、発音、声の高さなどボットの音声のさまざまな特性を制御できます。 SSML の詳細については、「<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language Reference (音声合成マークアップ言語リファレンス)</a>」を参照してください。

## <a name="input-hints"></a>入力ヒント

音声対応チャネルでメッセージを送信する場合、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すための入力ヒントを含めることによっても、クライアントのマイクの状態に影響を与えることを試すことができます。 詳細については、「[メッセージへの入力ヒントの追加](bot-builder-dotnet-add-input-hints.md)」を参照してください。

## <a name="sample-code"></a>サンプル コード 

Bot Builder SDK for .NET を使用して音声対応ボットを作成する方法を示す完全なサンプルについては、GitHub で<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-RollerSkill" target="_blank">ローラー スキルのサンプル</a>を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- [メッセージへの入力ヒントの追加](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">音声合成マークアップ言語 (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-RollerSkill" target="_blank">ローラー スキルのサンプル (GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity インターフェイス</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">DialogContext クラス</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Prompt クラス</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

