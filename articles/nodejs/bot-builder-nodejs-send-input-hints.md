---
title: メッセージへの入力ヒントの追加 | Microsoft Docs
description: Bot Framework SDK for .NET を使用してメッセージに入力ヒントを追加する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 513a5aa5b71bf8ec051a26027a56e67c01b1784a
ms.sourcegitcommit: 4c5c08e7c7eaa5f74c6ac35d8478954b998625f9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/29/2019
ms.locfileid: "64906292"
---
# <a name="add-input-hints-to-messages"></a>メッセージへの入力ヒントの追加

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

メッセージの入力ヒントを指定すれば、メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すことができます。 多くのチャネルでは、これによってクライアントが適宜、ユーザー入力コントロールの状態を設定できます。 たとえば、ボットがユーザーの入力を無視していることをメッセージの入力ヒントが示している場合、クライアントはユーザーが入力を提供できないよう、マイクを閉じて入力ボックスを無効にすることができます。

## <a name="accepting-input"></a>入力の受け付け

ボットが受動的に入力の準備ができているが、ユーザーからの応答を待っているわけではないことを示すには、メッセージの入力ヒントを `builder.InputHint.acceptingInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクは閉じられますがユーザーはまだマイクにアクセスできます。 たとえば、ユーザーがマイク ボタンを押し下げたままにすると、Cortana はマイクを開いてユーザーからの入力を受け付けます。 次のコード例は、ボットがユーザーの入力を受け付けていることを示すメッセージを作成します。

[!code-javascript[IMessage.speak](../includes/code/node-input-hints.js#InputHintAcceptingInput)]

## <a name="expecting-input"></a>入力の期待

ボットがユーザーからの応答を待っていることを示すには、メッセージの入力ヒントを `builder.InputHint.expectingInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクが開きます。 次のコード例は、ボットがユーザーの入力を期待していることを示すプロンプトを作成します。

[!code-javascript[Prompt](../includes/code/node-input-hints.js#InputHintExpectingInput)]

## <a name="ignoring-input"></a>入力の無視

ボットがユーザーから入力を受け取る準備ができていないことを示すには、メッセージの入力ヒントを `builder.InputHint.ignoringInput` に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが無効になり、マイクが閉じられます。 次のコード例は、`session.say()` メソッドを使用して、ボットがユーザーの入力を無視していることを示すメッセージを送信します。

[!code-javascript[Session.say()](../includes/code/node-input-hints.js#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>入力ヒントの既定値

メッセージの入力ヒントを設定しない場合、Bot Framework SDK が次のロジックを使用して自動的に入力ヒントを設定します。 

- ボットがプロンプトを送信する場合、メッセージの入力ヒントは、ボットが**入力を期待している**ことを指定します。</li>
- ボットが単一のメッセージを送信する場合、メッセージの入力ヒントは、ボットが**入力を受け付けている**ことを指定します。</li>
- ボットが連続したメッセージを送信する場合、最後を除いたすべてのメッセージの入力ヒントはボットが**入力を無視している**ことを指定し、最後のメッセージの入力ヒントはボットが**入力を受け付けている**ことを指定します。

## <a name="additional-resources"></a>その他のリソース

- [メッセージへの音声の追加](bot-builder-nodejs-text-to-speech.md)
