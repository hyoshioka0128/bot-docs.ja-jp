---
title: Microsoft Teams のボットのしくみ
description: ボットのしくみについての記事の続きであり、特に Microsoft Teams のボットに関するものです
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: overview
ms.service: bot-service
ms.date: 10/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3a7e60e1fa72402b5f783676f5281579a2be5371
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592202"
---
# <a name="how-microsoft-teams-bots-work"></a>Microsoft Teams のボットのしくみ

この記事では、「[ボットのしくみ](https://docs.microsoft.com/azure/bot-service/bot-builder-basics)」の記事で学習した内容を基に説明しています。この記事を読む前に理解しておく必要があります。

Microsoft Teams 向けに開発されたボットの主な違いは、アクティビティの処理方法です。 Microsoft Teams のアクティビティ ハンドラーは Bot Framework のアクティビティ ハンドラーから派生しており、チーム固有でないアクティビティの処理を許可する前に、すべてのチーム アクティビティをルーティングします。

## <a name="teams-activity-handlers"></a>Teams アクティビティ ハンドラー

他のボットと同様に、Microsoft Teams 向けに設計されているボットがアクティビティを受信した場合、*アクティビティ ハンドラー*に渡されます。 実際には "*ターン ハンドラー*" と呼ばれる基本ハンドラーが 1 つあり、そこですべてのアクティビティがルーティングされます。 *ターン ハンドラー*は、受信したあらゆる種類のアクティビティを処理するために必要なアクティビティ ハンドラーを呼び出します。 Microsoft Teams 向けに設計されたボットが違う点は、Bot Framework の `Activity Handler` クラスから派生した _Teams_ `Activity Handler` クラスから派生したものであるということです。  _Teams_ `Activity Handler` クラスには、この記事で説明するさまざまな Microsoft Teams 固有のさまざまなアクティビティ ハンドラーが含まれています。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Microsoft Bot Framework で作成されたあらゆるボットと同じように、ボットがメッセージ アクティビティを受信すると、ターン ハンドラーはその受信アクティビティを確認して、`OnMessageActivityAsync` アクティビティ ハンドラーに送信します。 この機能は変わりませんが、ボットが会話の更新アクティビティを受信した場合、ターン ハンドラーはその受信アクティビティを確認し、`OnConversationUpdateActivityAsync` _Teams_ アクティビティ ハンドラーに送信します。これは、最初に Teams 固有のイベントがあるかを確認し、見つからない場合は Bot Framework のアクティビティ ハンドラーに渡します。

Teams アクティビティ ハンドラー クラスには、すべての会話更新アクティビティをルーティングする `OnConversationUpdateActivityAsync` と、すべての Teams が呼び出すアクティビティをルーティングする `OnInvokeActivityAsync` という 2 つの主要な Teams アクティビティ ハンドラーがあります。  `How bots work` 記事の「[ボット ロジック](https://aka.ms/how-bots-work#bot-logic)」セクションに記載されているすべてのアクティビティ ハンドラーは、追加されたメンバーと削除されたアクティビティの処理を除き、Teams 以外のボットと同様に機能します。これらはチームという点で異なり、新しいメンバーは、メッセージ スレッドではなくチームに追加されます。  詳細については、「[ボット ロジック](#bot-logic)」セクションの _Teams の会話更新アクティビティ_に関する表を参照してください。

これらの Teams 固有のアクティビティ ハンドラーのロジックを実装するには、以下の「[ボット ロジック](#bot-logic)」セクションで示されるように、お使いのボットでこれらのメソッドをオーバーライドします。 これらのハンドラーそれぞれに基本実装はありません。このため、必要なロジックをご自身のオーバーライドに追加するだけです。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Microsoft Bot Framework で作成されたあらゆるボットと同じように、ボットがメッセージ アクティビティを受信すると、ターン ハンドラーはその受信アクティビティを確認して、`onMessage` アクティビティ ハンドラーに送信します。 この機能は変わりませんが、ボットが会話更新アクティビティを受信した場合、ターン ハンドラーはその受信アクティビティを確認し、`dispatchConversationUpdateActivity` _Teams_ アクティビティ ハンドラーに送信します。これは、最初に Teams 固有のイベントがあるかを確認し、見つからない場合は Bot Framework のアクティビティ ハンドラーに渡します。

Teams アクティビティ ハンドラー クラスには、すべての会話更新アクティビティをルーティングする `dispatchConversationUpdateActivity` と、すべての Teams が呼び出すアクティビティをルーティングする `onInvokeActivity` という 2 つの主要な Teams アクティビティ ハンドラーがあります。  `How bots work` 記事の「[ボット ロジック](https://aka.ms/how-bots-work#bot-logic)」セクションに記載されているすべてのアクティビティ ハンドラーは、追加されたメンバーと削除されたアクティビティの処理を除き、Teams 以外のボットと同様に機能します。これらはチームという点で異なり、新しいメンバーは、メッセージ スレッドではなくチームに追加されます。  詳細については、「[ボット ロジック](#bot-logic)」セクションの _Teams の会話更新アクティビティ_に関する表を参照してください。

あらゆるボット ハンドラーと同じように、Teams ハンドラーのロジックを実装するには、以下の「[ボット ロジック](#bot-logic)」セクションで示すように、お使いのボットでこれらのメソッドをオーバーライドします。 ハンドラーごとにボット ロジックを定義し、**最後に必ず `next()` を呼び出します**。 `next()` を呼び出すことで、次のハンドラーが確実に実行されます。

---

### <a name="bot-logic"></a>ボット ロジック

ボット ロジックは 1 つ以上のボットのチャネルからの受信アクティビティを処理し、応答の送信アクティビティを生成します。  これは、Teams アクティビティがあるかどうかを最初に確認し、その他のすべてのアクティビティを Bot Framework のアクティビティハンドラーに渡す、Teams アクティビティ ハンドラー クラスから派生したボットにも当てはまります。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="teams-conversation-update-activities"></a>Teams の会話更新アクティビティ

次に示すのは、`OnConversationUpdateActivityAsync` _Teams_ アクティビティ ハンドラーから呼び出されたすべての Teams アクティビティ ハンドラーの一覧です。 [会話更新イベント](https://aka.ms/azure-bot-subscribe-to-conversation-events)に関する記事では、ボットでこれらのイベントを使用する方法について説明します。

| Event | Handler | 説明 |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | 作成された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネルの作成](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created)」を参照してください。 |
| channelDeleted | `OnTeamsChannelDeletedAsync` | 削除された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネルの削除](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted)」を参照してください。 |
| channelRenamed | `OnTeamsChannelRenamedAsync` | 名前が変更された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネル名の変更](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed)」を参照してください。 |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` 名前が変更された Teams のチームを処理するには、これをオーバーライドします。 詳細については、「[チームの名前変更](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed)」を参照してください。 |
| MembersAdded | `OnTeamsMembersAddedAsync` | `ActivityHandler` で `OnMembersAddedAsync` メソッドを呼び出します。 これをオーバーライドして、チームに参加するメンバーを処理します。 詳細については、「[チームメンバーの追加](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added)」(追加されたチーム メンバー) を参照してください。|
| MembersRemoved | `OnTeamsMembersRemovedAsync` | `ActivityHandler` で `OnMembersRemovedAsync` メソッドを呼び出します。 これをオーバーライドして、チームから削除するメンバーを処理します。 詳細については、「[チームメンバーの削除](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed)」(削除されたチームメンバー) を参照してください。|


<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedAsync` | Calls the `OnMembersAddedAsync` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Calls the `OnMembersRemovedAsync` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>チームが呼び出したアクティビティ

次に示すのは、`OnInvokeActivityAsync` _Teams_ アクティビティ ハンドラーから呼び出されたすべての Teams アクティビティ ハンドラーの一覧です。

| 呼び出しの種類                    | Handler                              | 説明                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `OnTeamsCardActionInvokeAsync`       | Teams のカード アクションの呼び出しです。 |
| fileConsent/invoke              | `OnTeamsFileConsentAcceptAsync`      | Teams のファイルの同意の許可です。 |
| fileConsent/invoke              | `OnTeamsFileConsentAsync`            | Teams のファイルの同意です。 |
| fileConsent/invoke              | `OnTeamsFileConsentDeclineAsync`     | Teams のファイルの同意です。 |
| actionableMessage/executeAction | `OnTeamsO365ConnectorCardActionAsync` | Teams O365 コネクタ カード アクションです。 |
| signin/verifyState              | `OnTeamsSigninVerifyStateAsync`      | Teams のサインイン確認の状態です。 |
| task/fetch                      | `OnTeamsTaskModuleFetchAsync`        | Teams のタスク モジュールのフェッチです。 |
| task/submit                     | `OnTeamsTaskModuleSubmitAsync`       | Teams のタスク モジュールの送信です。 |

上記の呼び出しアクティビティは、Teams の会話ボット用です。 また、Bot Framework SDK もメッセージングの拡張機能固有の呼び出しをサポートします。 詳細については、「[What are messaging extensions](https://aka.ms/azure-bot-what-are-messaging-extensions)」(メッセージングの拡張機能とは) を参照してください

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="teams-conversation-update-activities"></a>Teams の会話更新アクティビティ

次に示すのは、`dispatchConversationUpdateActivity` _Teams_ アクティビティ ハンドラーから呼び出されたすべての Teams アクティビティ ハンドラーの一覧です。 [会話更新イベント](https://aka.ms/azure-bot-subscribe-to-conversation-events)に関する記事では、ボットでこれらのイベントを使用する方法について説明します。

| Event | Handler | 説明 |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | 作成された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネルの作成](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created)」を参照してください。 |
| channelDeleted | `OnTeamsChannelDeletedEvent` | 削除された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネルの削除](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted)」を参照してください。|
| channelRenamed | `OnTeamsChannelRenamedEvent` | 名前が変更された Teams チャネルを処理するには、これをオーバーライドします。 詳細については、「[チャネル名の変更](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed)」を参照してください。 |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` 名前が変更された Teams のチームを処理するには、これをオーバーライドします。 詳細については、「[チームの名前変更](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed)」を参照してください。 |
| MembersAdded | `OnTeamsMembersAddedEvent` | `ActivityHandler` で `OnMembersAddedEvent` メソッドを呼び出します。 これをオーバーライドして、チームに参加するメンバーを処理します。 詳細については、「[チームメンバーの追加](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added)」を参照してください。 |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | `ActivityHandler` で `OnMembersRemovedEvent` メソッドを呼び出します。 これをオーバーライドして、チームから削除するメンバーを処理します。 詳細については、「[チームメンバーの削除](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed)」を参照してください。 |

<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedEvent` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedEvent` | Calls the `OnMembersAddedEvent` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Calls the `OnMembersRemovedEvent` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>チームが呼び出したアクティビティ

次に示すのは、`onInvokeActivity` _Teams_ アクティビティ ハンドラーから呼び出されたすべての Teams アクティビティ ハンドラーの一覧です。

| 呼び出しの種類                    | Handler                              | 説明                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `handleTeamsCardActionInvoke`       | Teams のカード アクションの呼び出しです。 |
| fileConsent/invoke              | `handleTeamsFileConsentAccept`      | Teams のファイルの同意の許可です。 |
| fileConsent/invoke              | `handleTeamsFileConsent`            | Teams のファイルの同意です。 |
| fileConsent/invoke              | `handleTeamsFileConsentDecline`     | Teams のファイルの同意です。 |
| actionableMessage/executeAction | `handleTeamsO365ConnectorCardAction` | Teams O365 コネクタ カード アクションです。 |
| signin/verifyState              | `handleTeamsSigninVerifyState`      | Teams のサインイン確認の状態です。 |
| task/fetch                      | `handleTeamsTaskModuleFetch`        | Teams のタスク モジュールのフェッチです。 |
| task/submit                     | `handleTeamsTaskModuleSubmit`       | Teams のタスク モジュールの送信です。 |

上記の呼び出しアクティビティは、Teams の会話ボット用です。また、Bot Framework SDK もメッセージングの拡張機能固有の呼び出しをサポートします。詳細については、「[メッセージングの拡張機能とは](https://aka.ms/azure-bot-what-are-messaging-extensions)」を参照してください

---

## <a name="next-steps"></a>次の手順
Teams のボットをビルドするには、Microsoft Teams 開発者[ドキュメント](https://aka.ms/teams-docs)を参照してください。
