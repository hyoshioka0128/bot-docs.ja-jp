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
ms.openlocfilehash: 6ee7120536d42257dde2ed1411df32d807268e33
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000021"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>ボットを Bot Framework API v3 にアップグレードする

Build 2016 で、Microsoft は Microsoft Bot Framework と Bot Connector API の初期イテレーションのほか、Bot Builder と Bot Connector SDK を発表しました。 それ以降、ユーザーのフィードバックを収集し、REST API と SDK の改良に積極的に取り組んできました。

2016 年 7 月、Bot Framework API v3 がリリースされ、Bot Framework API v1 は非推奨になりました。 API v1 を使用するボットは、Skype では 2016 年 12 月に機能しなくなり、他のすべてのチャネルでは 2017 年 2 月 23 日に機能しなくなりました。 API v1 を使用して作成したボットを再度機能させるには、この記事の手順に従って API v3 にアップグレードする必要があります。 アップグレードのプロセス全体を確実に理解するために、この記事を通読してから、作業を開始してください。 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>手順 1: Bot Framework ポータルからアプリ ID とパスワードを取得する

[Bot Framework ポータル](https://dev.botframework.com/)にサインインし、**[My bots]\(マイ ボット\)** をクリックしてボットを選択してから、そのダッシュボードを開きます。 次に、ページの右上隅の近くにある **[SETTINGS]\(設定\)** リンクをクリックします。 

設定ページの **[Configuration]\(構成\)** セクション内で、**[App ID]\(アプリ ID\)** フィールドの内容を調べ、**[App ID]\(アプリ ID\)** フィールドが既に設定されているかどうかに応じて次の手順に進みます。

### <a name="case-1-app-id-field-is-already-populated"></a>ケース 1: [App ID]\(アプリ ID\) フィールドが既に設定されている

**[App ID]\(アプリ ID\)** フィールドが既に設定されている場合は、次の手順を実行します。

1. **[Manage Microsoft App ID and password]\(Microsoft アプリ ID とパスワードの管理\)** をクリックします。  
![構成](~/media/upgrade/manage-app-id.png)

2. **[Generate New Password]\(新しいパスワードを生成\)** をクリックします。  
![新しいパスワードを生成](~/media/upgrade/generate-new-password.png)

3. 新しいパスワードと MSA アプリ ID をコピーして保存します。これらの値は、後で必要になります。  
![新しいパスワード](~/media/upgrade/new-password-generated.png)

### <a name="case-2-app-id-field-is-empty"></a>ケース 2: [App ID]\(アプリ ID\) フィールドが空である

**[App ID]\(アプリ ID\)** フィールドが空である場合は、次の手順を実行します。

1. **[Create Microsoft App ID and password]\(Microsoft アプリ ID とパスワードの作成\)** をクリックします。  
   ![アプリ ID とパスワードの作成](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > まだ **[Version 3.0]\(バージョン 3.0\)** ボタンを選択しないでください。 この操作は、[ボット コードを更新](#update-code)した後で行います。</div>

2. **[Generate a password to continue]\(パスワードを生成して続行する\)** をクリックします。  
   ![アプリのパスワードを生成する](~/media/upgrade/generate-a-password-to-continue.png)

3. 新しいパスワードと MSA アプリ ID をコピーして保存します。これらの値は、後で必要になります。  
   ![新しいパスワード](~/media/upgrade/new-password-generated.png)

4. **[Finish and go back to Bot Framework]\(終了してボットのフレームワークに戻る\)** をクリックします。  
   ![終了してポータルに戻る](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Bot Framework ポータルのボット設定ページに戻り、ページの一番下までスクロールして、**[Save changes]\(変更を保存\)** をクリックします。  
   ![変更を保存](~/media/upgrade/save-changes.png)

## <a id="update-code"></a> 手順 2: ボット コードをバージョン 3.0 に更新する

ボット コードをバージョン 3.0 に更新するには、次の手順に従います。

1. ボットの言語用の [Bot Builder SDK](https://github.com/Microsoft/BotBuilder) の最新バージョンに更新します。
2. 以下のガイダンスに従って、必要な変更が適用されるようにコードを更新します。
3. [Bot Framework Emulator](~/bot-service-debug-emulator.md) を使用してボットをローカルでテストし、その後クラウドでテストします。

以降のセクションでは、API v1 と API v3 の主な違いについて説明します。 コードを API v3 に更新した後、Bot Framework ポータルで[ボット設定を更新](#step-3)することで、アップグレード プロセスを完了することができます。

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder と Connector が 1 つの SDK に

複数の NuGet パッケージ (または NPM モジュール) を使用して Builder 用と Connector 用の個別の SDK をインストールする代わりに、単一の Bot Builder SDK で両方のライブラリを取得できるようになりました。

- Bot Builder SDK for .NET: `Microsoft.Bot.Builder` NuGet パッケージ
- Bot Builder SDK for Node.js: `botbuilder` NPM モジュール

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

## <a id="step-3"></a> 手順 3: Bot Framework ポータルでボット設定を更新する

ボット コードを API v3 にアップグレードしてクラウドにデプロイした後、次の手順を実行してアップグレード プロセスを完了します。 

1. [Bot Framework ポータル](https://dev.botframework.com/)にサインインします。

2. **[My bots]\(マイ ボット\)** をクリックし、ボットを選択してダッシュボードを開きます。 

3. ページの右上隅の近くにある **[SETTINGS]\(設定\)** リンクをクリックします。 

4. **[Configuration]\(構成\)** セクション内の **[Version 3.0]\(バージョン 3.0\)** で、ボットのエンドポイントを **[Messaging endpoint]\(メッセージング エンドポイント\)** フィールドに貼り付けます。  
![バージョン 3 の構成](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. **[Version 3.0]\(バージョン 3.0\)** ボタンを選択します。  
![バージョン 3.0 を選択する](~/media/upgrade/switch-to-v3-endpoint.png)

6. ページの下部までスクロールし、**[Save changes]\(変更の保存\)** をクリックします。  
![変更を保存](~/media/upgrade/save-changes.png)