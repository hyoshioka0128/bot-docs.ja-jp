---
title: 状態データの管理 (v3 JS) - Bot Service
description: Bot Framework SDK for Node.js を使用して、状態データを保存および取得する方法について説明します。
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 40293ff3756687f270847dd9045a04a19363ad1b
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75790327"
---
# <a name="manage-state-data"></a>状態データの管理

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>インメモリ データ ストレージ

メモリ内データ ストレージはテスト専用です。 このストレージは揮発性であり、一時的なものです。 ボットの再起動ごとにデータがクリアされます。 メモリ内ストレージをテスト目的で使用するには、2 つの作業を行う必要があります。 最初に、メモリ内ストレージの新しいインスタンスを作成します。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

その後、**UniversalBot** の作成時にそれをボットに設定します。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

この方法を使用して、独自のカスタム データ ストレージを設定できます。また、*Azure 拡張機能*のいずれかを使用することもできます。

## <a name="manage-custom-data-storage"></a>カスタム データ ストレージを管理する

運用環境でのパフォーマンスとセキュリティ上の理由から、独自のデータ ストレージを実装するか、次のデータ ストレージ オプションのいずれかを実装することを検討できます。

1. [Cosmos DB を使用して状態データを管理する](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [Table Storage を使用して状態データを管理する](bot-builder-nodejs-state-azure-table-storage.md)

これらの [Azure 拡張機能](https://www.npmjs.com/package/botbuilder-azure)オプションのどちらを使用しても、Bot Framework SDK for Node.js によるデータの設定と保持のメカニズムがメモリ内データ ストレージであることに変わりはありません。

## <a name="storage-containers"></a>ストレージ コンテナー

Bot Framework SDK for Node.js では、`session` オブジェクトにより、状態データを格納するための以下のプロパティが公開されます。

| プロパティ | スコープ | 説明 |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | User | 指定されたチャネルのユーザー用に保存されるデータが含まれます。 このデータは複数の会話間で保持されます。 |
| [`privateConversationData`][privateConversationDataURL] | 会話 | 指定されたチャネルでの特定の会話のコンテキスト内のユーザー用に保存されるデータが含まれます。 このデータは現在のユーザー専用であり、現在の会話でのみ保持されます。 会話が終了したときや `endConversation` が明示的に呼び出されたときに、プロパティがクリアされます。 |
| [`conversationData`][conversationDataURL] | 会話 | 指定されたチャネルでの特定の会話のコンテキストで保存されるデータが含まれます。 このデータは、会話に参加しているすべてのユーザーで共有され、現在の会話でのみ保持されます。 会話が終了したときや `endConversation` が明示的に呼び出されたときに、プロパティがクリアされます。 |
| [`dialogData`][dialogDataURL] | ダイアログ | 現在のダイアログ用にのみ保存されるデータが含まれます。 各ダイアログでは、このプロパティの独自のコピーが保持されます。 ダイアログがダイアログ スタックから削除されたときに、プロパティがクリアされます。 |

これら 4 つのプロパティは、データを格納するために使用できる 4 つのデータ ストレージ コンテナーに対応しています。 データを格納するために使用するプロパティは、格納するデータの適切なスコープ、格納するデータの性質、およびデータを保持する期間によって異なります。 たとえば、複数の会話間で利用可能なユーザー データを格納する必要がある場合は、`userData` プロパティの使用を検討してください。 ダイアログのスコープ内でローカル変数の値を一時的に格納する必要がある場合は、`dialogData` プロパティの使用を検討してください。 複数のダイアログ間でアクセスできる必要があるデータを一時的に格納する必要がある場合は、`conversationData` プロパティの使用を検討してください。

## <a name="data-persistence"></a>データの永続化

既定では、`userData`、`privateConversationData`、および `conversationData` プロパティを使用して格納されるデータは、会話の終了後に保持するように設定されます。 `userData` コンテナーでデータを保持しない場合は、`persistUserData` フラグを **false** に設定します。 `conversationData` コンテナーでデータを保持しない場合は、`persistConversationData` フラグを **false** に設定します。 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> `privateConversationData` コンテナーに対してデータの永続化を無効にすることはできません。データは常に保持されます。

## <a name="set-data"></a>データを設定する

シンプルな JavaScript オブジェクトはストレージ コンテナーに直接保存することで格納できます。 `Date` のような複合オブジェクトの場合は、`string` への変換を検討してください。 これは、状態データがシリアル化され、JSON として格納されるためです。 次のコード サンプルは、プリミティブ データ、配列、オブジェクト マップ、複合 `Date` オブジェクトを格納する方法を示しています。 

**プリミティブ データを格納する**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**配列を格納する**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**オブジェクト マップを格納する**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**日付と時刻を格納する** 

複合 JavaScript スクリプトの場合は、ストレージ コンテナーに保存する前に文字列に変換します。 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>データの保存

各ストレージ コンテナーで作成されているデータは、コンテナーが保存されるまでメモリに残ります。 Bot Framework SDK for Node.js では、`ChatConnector` サービスにデータがバッチで送信され、送信するメッセージがある場合は保存されます。 いずれのメッセージも送信せずにストレージ コンテナーに存在するデータを保存する場合は、[`save`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save) メソッドを手動で呼び出すことができます。 `save` メソッドを呼び出さない場合、ストレージ コンテナーに存在するデータはバッチ処理の一環として保持されます。

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>データを取得する

特定のストレージ コンテナーに保存されているデータにアクセスするには、単に対応するプロパティを参照します。 次のコード サンプルは、プリミティブ データ、配列、オブジェクト マップ、複合 Date オブジェクトとして以前に格納されたデータにアクセスする方法を示しています。

**プリミティブ データにアクセスする**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**配列にアクセスする**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**オブジェクト マップにアクセスする**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**Date オブジェクトにアクセスする** 

日付データを文字列として取得し、JavaScript の Date オブジェクトに変換します。 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>データの削除

既定では、`dialogData` コンテナーに格納されているデータは、ダイアログ スタックからダイアログが削除されたときにクリアされます。 同様に、`conversationData` コンテナーと `privateConversationData` コンテナーに格納されているデータは、`endConversation` メソッドが呼び出されたときにクリアされます。 しかし、`userData` コンテナーに格納されたデータを削除するには、それを明示的にクリアする必要があります。

ストレージ コンテナーのいずれかに格納されているデータを明示的にクリアするには、次のコード サンプルで示すように、単にコンテナーをリセットします。 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

データ コンテナーを `null` に設定したり、`session` オブジェクトから削除したりしないでください。そのようにすると、次回、コンテナーにアクセスしようとしたときにエラーが発生します。 また、以前に保持された対応するデータをすべてクリアする場合は、メモリ内のコンテナーを手動でクリアしてから `session.save();` を手動で呼び出すことができます。

## <a name="next-steps"></a>次のステップ

これでユーザーの状態データを管理する方法について理解できました。次は、状態データを使用して会話フローの管理を強化する方法を見てみましょう。

> [!div class="nextstepaction"]
> [会話フローを管理する](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>その他のリソース
- [ユーザーに入力を要求する](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
