---
title: メッセージへの入力ヒントの追加 | Microsoft Docs
description: Bot Builder SDK を使用してメッセージに入力ヒントを追加する方法について説明します。
keywords: Inputhints, 入力の受け付け, 入力の期待, 入力の無視, 音声
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: acf119a05d4c9f37b74c4fcaf2ad944978504560
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707248"
---
# <a name="add-input-hints-to-messages"></a>メッセージへの入力ヒントの追加

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

メッセージの入力ヒントを指定すれば、メッセージがクライアントに配信された後、ボットがユーザー入力を受け付けるか、期待するか、無視するかを示すことができます。 多くのチャネルでは、これによってクライアントが適宜、ユーザー入力コントロールの状態を設定できます。 たとえば、ボットがユーザーの入力を無視していることをメッセージの入力ヒントが示している場合、クライアントはマイクを閉じて入力ボックスを無効にし、ユーザーが入力を提供するのを防ぐことができます。

入力ヒントに必要なライブラリが含まれていることを確認してください。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

これらの例で使用される **MessageFactory** クラスは、次の名前空間で定義されています。

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>入力の受け付け

ボットが受動的に入力の準備ができているが、ユーザーからの応答を待っていないことを示すには、メッセージの入力ヒントを _accepting input_ に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクは閉じられますがユーザーはまだマイクにアクセスできます。 たとえば、ユーザーがマイクボタンを押し下げたままにすると、Cortana はマイクを開いてユーザーからの入力を受け付けます。 次のコードは、ボットがユーザーの入力を受け付けていることを示すメッセージを作成します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>入力の期待

ボットがユーザーからの応答を待っていることを示すには、メッセージの入力ヒントを _expecting input_ に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが有効になり、マイクが開きます。 次のコード例は、ボットがユーザーの入力を期待していることを示すメッセージを作成します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>入力の無視

ボットがユーザーから入力を受け取る準備ができていないことを示すには、メッセージの入力ヒントを _ignoring input_ に設定します。 多くのチャネルでは、これによってクライアントの入力ボックスが無効になり、マイクが閉じられます。 次のコード例は、ボットがユーザーの入力を無視していることを示すメッセージを作成します。

# <a name="ctabcs"></a>[C# を選択した場合](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript を選択した場合](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>入力ヒントの既定値

メッセージの入力ヒントを設定しない場合、Bot Builder SDK が次のロジックを使用して自動的に入力ヒントを設定します。

- ボットがプロンプトを送信する場合、メッセージの入力ヒントは、ボットが入力を期待している (**expecting input**) ことを指定します。</li>
- ボットが単一のメッセージを送信する場合、メッセージの入力ヒントは、ボットが入力を受け付けている (**accepting input**) ことを指定します。</li>
- ボットが連続したメッセージを送信する場合、最後を除いたすべてのメッセージの入力ヒントはボットが入力を無視している (**ignoring input**) ことを指定し、最後のメッセージの入力ヒントはボットが入力を受け付けている (**accepting input**) ことを指定します。

