---
title: ボットを Bot Framework API v3 にアップグレードする | Microsoft Docs
description: ボットを Bot Framework API v3 にアップグレードする方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 8d9b2ea2e2133c86428b537427433f9dd15216ee
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225947"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>ボットを Bot Framework API v3 にアップグレードする

Build 2016 で、Microsoft は Microsoft Bot Framework と Bot Connector API の初期イテレーションのほか、Bot Builder と Bot Connector SDK を発表しました。 それ以降、ユーザーのフィードバックを収集し、REST API と SDK の改良に積極的に取り組んできました。

2016 年 7 月、Bot Framework API v3 がリリースされ、Bot Framework API v1 は非推奨になりました。 API v1 を使用するボットは、Skype では 2016 年 12 月に機能しなくなり、他のすべてのチャネルでは 2017 年 2 月 23 日に機能しなくなりました。 API v1 を使用して作成したボットを再度機能させるには、この記事の手順に従って API v3 にアップグレードする必要があります。 アップグレードのプロセス全体を確実に理解するために、この記事を通読してから、作業を開始してください。 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>手順 1:Bot Framework ポータルからアプリ ID とパスワードを取得する

[Bot Framework ポータル](https://dev.botframework.com/)にサインインし、**[My bots]\(マイ ボット\)** をクリックしてボットを選択してから、そのダッシュボードを開きます。 次に、ページ左側の **[ボット管理]** の下にある **[設定]** リンクをクリックします。 

設定ページの **[構成]** セクション内で、**[Microsoft App ID]\(Microsoft アプリ ID\)** フィールドの内容を調べ、次の手順に進みます。

<!-- TODO: Remove this 
### Case 1: App ID field is already populated

If the **App ID** field is already populated, complete these steps:
-->

1. **[Manage Microsoft App ID and password]\(Microsoft アプリ ID とパスワードの管理\)** をクリックします。  
![構成](./media/upgrade/manage-app-id.png)

2. **[Generate New Password]\(新しいパスワードを生成\)** をクリックします。  
![新しいパスワードを生成](./media/upgrade/generate-new-password.png)

3. 新しいパスワードと MSA アプリ ID をコピーして保存します。これらの値は、後で必要になります。  
![新しいパスワード](./media/upgrade/new-password-generated.png)

こちらの[手順](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret/)に従って、ご自身の **Microsoft アプリ ID とパスワード**を取得する別の方法を実行することもできます。

<!-- TODO: These steps are no longer valid. AppID will always be generated, confirmed with Support Engineers
### Case 2: App ID field is empty

If the **App ID** field is empty, complete these steps:

1. Click **Create Microsoft App ID and password**.  
   ![Create App ID and password](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > Do not select the **Version 3.0** radio button yet. You will do this later, after you have [updated your bot code](#update-code).</div>

2. Click **Generate a password to continue**.  
   ![Generate app password](~/media/upgrade/generate-a-password-to-continue.png)

3. Copy and save the new password along with the MSA App Id; you will need these values in the future.  
   ![New password](~/media/upgrade/new-password-generated.png)

4. Click **Finish and go back to Bot Framework**.  
   ![Finish and go back to Portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Back on the bot settings page in the Bot Framework Portal, scroll to the bottom of the page and click **Save changes**.  
   ![Save changes](~/media/upgrade/save-changes.png)
-->

## <a id="update-code"></a>手順 2: ボット コードをバージョン 4.0 に更新する

V1 のボットは互換性がなくなりました。 お使いのボットを更新するには、V3 で新しいボットを作成する必要があります。 使用中の古いコードを保持する必要がある場合は、そのコードを手動で移行する必要があります。

最も簡単な解決策は、ご自身のボットを新しい [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0) で再作成し、デプロイすることです。 

使用中の古いコードを保持するには、次の手順に従ってください。

1. 新しいボット アプリケーションを作成します。
2. 古いコードを新しいボット アプリケーションにコピーします。
3. NuGet パッケージ マネージャーを使用して、SDK を最新バージョンにアップグレードします。
4. 表示されたすべてのエラーを修正し、新しい [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0) を参照します。
5. 次の[手順](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)に従って、ご自身のボットを Azure にデプロイします

<!-- TODO: Remove outdated code 
To update your bot code to version 3.0, complete these steps:

1. Update to the latest version of the [Bot Framework SDK](https://github.com/Microsoft/BotBuilder) for your bot's language.
2. Update your code to apply the necessary changes, according the guidance below.
3. Use the [Bot Framework Emulator](~/bot-service-debug-emulator.md) to test your bot locally and then in the cloud.

The following sections describe the key differences between API v1 and API v3. After you have updated your code to API v3, you can finish the upgrade process by [updating your bot settings](#step-3) in the Bot Framework Portal.
-->

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder と Connector が 1 つの SDK に

複数の NuGet パッケージ (または NPM モジュール) を使用して Builder 用と Connector 用の個別の SDK をインストールする代わりに、単一の Bot Framework SDK で両方のライブラリを取得できるようになりました。

- Bot Framework SDK for .NET:`Microsoft.Bot.Builder` NuGet パッケージ
- Bot Framework SDK for Node.js:`botbuilder` NPM モジュール

スタンドアロンの `Microsoft.Bot.Connector` SDK は非推奨になり、今後は保守されません。

### <a name="message-is-now-activity"></a>Message が Activity に

API v3 では、`Message` オブジェクトが `Activity` オブジェクトに置き換えられました。 最も一般的な種類のアクティビティは**メッセージ**ですが、さまざまな種類の情報をボットまたはチャネルに伝達するのに使用できるアクティビティの種類が他にもあります。 メッセージの詳細については、「[メッセージの作成](~/dotnet/bot-builder-dotnet-create-messages.md)」と「[アクティビティを送受信する](~/dotnet/bot-builder-dotnet-connector.md)」を参照してください。

### <a name="activity-types--events"></a>アクティビティの種類とイベント

API v3 では、一部のイベントに名前の変更やリファクタリングが施されました。 さらに、新しい `ActivityTypes` 列挙型が Connector に追加され、特定のアクティビティの種類を覚えておく必要がなくなりました。

- アクティビティの種類の `conversationUpdate` は、Bot/User Added/Removed To/From Conversation を単一のメソッドに置き換えるものです。
- 新しいアクティビティの種類の `typing` を使用すると、ボットが応答をコンパイル中であることを示したり、ユーザーが応答を入力中であることを把握したりできます。
- 新しいアクティビティの種類の `contactRelationUpdate` を使用すると、ボットはそれ自体がユーザーの連絡先リストで追加または削除されたかどうかを知ることができます。

ボットが `conversationUpdate` アクティビティを受信すると、`MembersRemoved` プロパティと `MembersAdded` プロパティには、会話に追加された人や会話から削除された人が示されます。 ボットが `contactRelationUpdate` アクティビティを受信すると、`Action` プロパティはユーザーが連絡先リストでボットを追加または削除したかどうかを示します。 アクティビティの種類の詳細については、「[アクティビティの概要](~/dotnet/bot-builder-dotnet-activities.md)」を参照してください。

### <a name="addressing-messages"></a>メッセージのアドレス指定

API v3 では、メッセージ内で送信者、受信者、およびチャネル情報を指定する場所が、わずかに変更されました。

|API v1 フィールド | API v3 フィールド|
|--------|--------|
| `From` オブジェクト | `From` オブジェクト |
| `To` オブジェクト | `Recipient` オブジェクト |
| `ChannelConversationID` プロパティ | `Conversation` オブジェクト|
| `ChannelId` プロパティ | `ChannelId` プロパティ |

メッセージのアドレス指定の詳細については、「[アクティビティを送受信する](~/dotnet/bot-builder-dotnet-connector.md)」を参照してください。

### <a name="sending-replies"></a>返信の送信

Bot Framework API v3 では、ユーザーへのすべての返信は、ボットへの着信メッセージの HTTP POST でインラインに行われるのではなく、個別に開始される HTTP 要求を介して非同期で送信されます。 Connector を介してメッセージがインラインでユーザーに返されることはないので、ボットの post メソッドの戻り値の型は `HttpResponseMessage` になります。 これは、ユーザーに送信する文字列をボットが同期的に "返す" のではなく、コード内のいずれかの位置で返信メッセージを送信することを意味します。受信 POST への応答として返信する必要はありません。 次の 2 つのメソッドは、どちらも会話にメッセージを送信します。

- `SendToConversation`
- `ReplyToConversation`

`SendToConversation` メソッドは、指定されたメッセージを会話の最後に追加します。一方、`ReplyToConversation` メソッドは、指定されたメッセージを会話内の前のメッセージに対する直接の返信として追加します (このメソッドがサポートされている会話の場合)。 これらのメソッドの詳細については、「[アクティビティを送受信する](~/dotnet/bot-builder-dotnet-connector.md)」を参照してください。

### <a name="starting-conversations"></a>会話の開始

Bot Framework API v3 では、会話を開始するために、新しいメソッド `CreateDirectConversation` を使用して単一のユーザーとのプライベートな会話を開始するか、新しいメソッド `CreateConversation` を使用して複数のユーザーとグループ会話を開始します。 会話の開始の詳細については、「[アクティビティを送受信する](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation)」を参照してください。

### <a name="attachments-and-options"></a>添付ファイルとオプション

Bot Framework API v3 では、添付ファイルとカードのより堅牢な実装が導入されています。 `Options` 型は API v3 ではサポートされなくなり、カードに置き換えられました。 .NET を使用してメッセージに添付ファイルを追加する方法の詳細については、「[メッセージへのメディア添付ファイルの追加](~/dotnet/bot-builder-dotnet-add-media-attachments.md)」と「[メッセージへのリッチ カードの追加](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md)」を参照してください。

### <a name="bot-data-storage-bot-state"></a>ボットのデータ ストレージ (ボット状態)

Bot Framework API v1 では、ボット状態データを管理するための API がメッセージング API に組み込まれていました。 Bot Framework API v3 では、これらの API は別々になっています。 そのため、状態データを取得し (`Message` オブジェクトにデータが含められると想定するのではなく)、状態データを格納する (`Message` オブジェクトの一部として渡すのではなく) には、Bot State サービスを使用する必要があります。 Bot State サービスを使用してボット状態データを管理する方法については、「[状態データの管理](~/dotnet/bot-builder-dotnet-state.md)」を参照してください。

> [!IMPORTANT]
> Bot Framework State Service API を運用環境で使用することは推奨されていません。将来のリリースで非推奨となる可能性があります。 テストが目的の場合はメモリ内ストレージを使用するようにボット コードを更新し、運用ボットの場合はいずれかの **Azure 拡張機能**を使用することをお勧めします。 [.NET](~/dotnet/bot-builder-dotnet-state.md) または [Node](~/nodejs/bot-builder-nodejs-state.md) での実装の詳細については、「**状態データの管理**」トピックを参照してください。

### <a name="webconfig-changes"></a>Web.config の変更

Bot Framework API v1 では、認証プロパティを以下のキーで **Web.Config** に保存していました。

- `AppID`
- `AppSecret`

Bot Framework API v3 では、認証プロパティを以下のキーで **Web.Config** に保存します。

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> 手順 3:自分の更新ボットを Azure にデプロイする。

ご自身のボット コードを API v3 にアップグレードしたら、こちらの[手順](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)に従って、そのボットを Azure にデプロイします。 V1 はサポートされていないため、Azure サービスにデプロイされたすべてのボットで V3 API が自動的に使用されます。

<!-- TODO: Documentation set for removal 
1. Sign in to the [Bot Framework Portal](https://dev.botframework.com/).

2. Click **My bots** and select your bot to open its dashboard. 

3. Click the **SETTINGS** link that is located near the top-right corner of the page. 

4. Under **Version 3.0** within the **Configuration** section, paste your bot's endpoint into the **Messaging endpoint** field.  
![Version 3 configuration](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Select the **Version 3.0** radio button.  
![Select version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Scroll to the bottom of the page and click **Save changes**.  
![Save changes](~/media/upgrade/save-changes.png)
-->