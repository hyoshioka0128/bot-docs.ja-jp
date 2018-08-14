---
title: メッセージへの入力ヒントの追加 | Microsoft Docs
description: Bot Builder SDK for .NET を使用してメッセージに入力ヒントを追加する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2f56a855990675ccae4845c13541150ab205379a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303857"
---
# <a name="add-input-hints-to-messages"></a>メッセージへの入力ヒントの追加
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

メッセージの入力ヒントを指定すれば、メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すことができます。 多くのチャネルでは、これによってクライアントが適宜、ユーザー入力コントロールの状態を設定できます。 たとえば、ボットがユーザーの入力を無視していることをメッセージの入力ヒントが示している場合、クライアントはユーザーが入力を提供できないよう、マイクを閉じて入力ボックスを無効にすることができます。

## <a name="accepting-input"></a>入力の受け付け

ボットが受動的に入力の準備ができているが、ユーザーからの応答を待っているわけではないことを示すには、メッセージの入力ヒントを `InputHints.AcceptingInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクは閉じられますがユーザーはまだマイクにアクセスできます。 たとえば、ユーザーがマイク ボタンを押し下げたままにすると、Cortana はマイクを開いてユーザーからの入力を受け付けます。 次のコード例は、ボットがユーザーの入力を受け付けていることを示すメッセージを作成します。

[!code-csharp[Accepting input](../includes/code/dotnet-input-hints.cs#InputHintAcceptingInput)]

## <a name="expecting-input"></a>入力の期待

ボットがユーザーからの応答を待っていることを示すには、メッセージの入力ヒントを `InputHints.ExpectingInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクが開きます。 次のコード例は、ボットがユーザーの入力を期待していることを示すメッセージを作成します。

[!code-csharp[Expecting input](../includes/code/dotnet-input-hints.cs#InputHintExpectingInput)]

## <a name="ignoring-input"></a>入力の無視
 
ボットがユーザーから入力を受け取る準備ができていないことを示すには、メッセージの入力ヒントを `InputHints.IgnorningInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが無効になり、マイクが閉じられます。 次のコード例は、ボットがユーザーの入力を無視していることを示すメッセージを作成します。

[!code-csharp[Ignoring input](../includes/code/dotnet-input-hints.cs#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>入力ヒントの既定値

メッセージの入力ヒントを設定しない場合、Bot Builder SDK が次のロジックを使用して自動的に入力ヒントを設定します。 

- ボットがプロンプトを送信する場合、メッセージの入力ヒントは、ボットが**入力を期待している**ことを指定します。</li>
- ボットが単一のメッセージを送信する場合、メッセージの入力ヒントは、ボットが**入力を受け付けている**ことを指定します。</li>
- ボットが連続したメッセージを送信する場合、最後を除いたすべてのメッセージの入力ヒントはボットが**入力を無視している**ことを指定し、最後のメッセージの入力ヒントはボットが**入力を受け付けている**ことを指定します。

## <a name="additional-resources"></a>その他のリソース

- [メッセージの作成](bot-builder-dotnet-create-messages.md)
- [メッセージへの音声の追加](bot-builder-dotnet-text-to-speech.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
- <a href="/dotnet/api/microsoft.bot.connector.inputhints" target="_blank">InputHints クラス</a>