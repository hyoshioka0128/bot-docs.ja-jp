---
title: Bot Service についてよく寄せられる質問 - Bot Service
description: Bot Framework の要素と、新しい機能が利用可能になる時期についてよく寄せられる質問を示します。
author: scheyal
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2020
ms.openlocfilehash: c1e95370a58fa1f020bbc170c83ee732b10e92b9
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "81395675"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework についてよく寄せられる質問

この記事では、Bot Framework についてよく寄せられる質問とその回答を示します。

## <a name="background-and-availability"></a>背景と提供状況

### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft が Bot Framework を開発したのはなぜですか?

Microsoft は、開発者が優れたボットを作成し、Microsoft のプレミア チャンネル上など、会話する場所がどこであってもユーザーに接続することを容易にするために、Bot Framework を開発しました。

### <a name="what-is-the-v4-sdk"></a>v4 SDK とは何ですか?
Bot Framework v4 SDK は、以前の Bot Framework SDK から得たフィードバックと知識を基に構築されています。 ボットの構成要素の豊富なコンポーネント化を可能にしながら、適切なレベルの抽象化が導入されます。 簡単なボットから始め、モジュール式の拡張可能なフレームワークを使用してボットを高度化していくことができます。 GitHub で SDK の [FAQ](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ) をご覧ください。

### <a name="running-a-bot-offline"></a>ボットをオフラインで実行する

<!-- WIP -->
ボットのオフラインでの使用について説明する前に、Azure または一部のその他のホスト サービス上ではなく、オンプレミスにデプロイされるボットについて、いくつかの点を明確にしておきます。

- ボットは UI を持たない Web サービスです。そのためユーザーはそれとの対話を、[Bot Framework Service](rest-api/bot-framework-rest-connector-concepts.md) を使用する、チャンネルの形式の他の方法で行う必要があります。 コネクタは、"*プロキシ*" として機能して、クライアントとボット間のメッセージを中継します。
- **コネクタ**は、Azure ノードでホストされるグローバル アプリケーションであり、可用性とスケーラビリティのために地理的に分散されています。 
- ボットをコネクタに登録するには、[ボット チャネル登録](bot-service-quickstart-registration.md)を使用します。
    >[!NOTE]
    > ボットのエンドポイントは、コネクタによってパブリックに到達可能である必要があります。

機能制限付きでボットをオフラインで実行することができます。 たとえば、LUIS 機能を持つボットをオフラインで使用する場合は、ボット用のコンテナー、必要なツール、LUIS 用のコンテナーを作成する必要があります。 いずれも Docker Compose のブリッジされたネットワーク経由で接続されます。

これは、Cognitive Services コンテナーに定期的なオンライン接続が必要なため、"部分的な" オフライン ソリューションです。

> [!NOTE]
> QnA サービスは、オフラインで実行されるボットではサポートされていません。

詳細については、次を参照してください。

- [Language Understanding (LUIS) コンテナーを Azure Container Instances にデプロイする](https://docs.microsoft.com/azure/cognitive-services/luis/deploy-luis-on-container-instances)
- [Azure Cognitive Services でのコンテナーのサポート](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support)

## <a name="bot-framework-sdk-version-3-lifetime-support-and-deprecation-notice"></a>Bot Framework SDK バージョン 3 のライフタイム サポートおよび非推奨化に関する通知
Microsoft Bot Framework SDK V4 は 2018 年 9 月にリリースされ、それ以降、数回にわたってドットリリースの改良が公開されました。 以前に発表したように、V3 SDK は廃止されています。 したがって、V3 リポジトリでの開発はこれ以上行われない予定です。 **既存の V3 ボット ワークロードは、中断することなく引き続き実行されます。実行中のワークロードを中断する予定はありません**。

前述のように、Bot Builder SDK V3 ボットは Azure Bot Service によって引き続き実行およびサポートされます。 Bot Builder SDK V3 は今後、重大なセキュリティ バグの修正、コネクタ、およびプロトコル層互換性の更新のみがサポートの対象となります。  

すべての新機能は [Bot Framework SDK V4](https://github.com/microsoft/botframework-sdk) でのみ開発されています。  お客様には、できるだけ早くボットを V4 に移行することをお勧めします。

V3 ボットから V4 への移行を開始することを強くお勧めします。 この移行をサポートするために、Microsoft では移行に関するドキュメントを作成しました。また、Stack Overflow や Microsoft カスタマー サポートなどの標準チャネルを介して移行イニシアチブに対する拡張サポートを提供する予定です。


詳細については、以下のリファレンスを参照してください。
* [基本的な移行ガイダンス](https://aka.ms/bf-migration-overview)
* Bot Framework ボットを開発するためのプライマリ V4 リポジトリ
  * [dotnet 用 Botbuilder](https://github.com/microsoft/botbuilder-dotnet)
  * [JS 用 Botbuilder](https://github.com/microsoft/botbuilder-js) 
* QnA Maker ライブラリは、次の V4 ライブラリに置き換えられました。
  * [dotnet 用ライブラリ](https://github.com/Microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder.AI.QnA)
  * [JS 用ライブラリ](https://github.com/Microsoft/botbuilder-js/blob/master/libraries/botbuilder-ai/src/qnaMaker.ts)
* Azure ライブラリは、次の V4 ライブラリに置き換えられました。
  * [JS Azure 用 Botbuilder](https://github.com/Microsoft/botbuilder-js/tree/master/libraries/botbuilder-azure)
  * [dotnet Azure 用 Botbuilder](https://github.com/Microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder.Azure)

### <a name="v3-status-summary"></a>V3 ステータスの概要

#### <a name="abs-service"></a>ABS サービス
1.    ABS サービス サイドでは、V3 ボットの実行が引き続きサポートされ、サポート終了の予定はなく、実行中のボットが中断されることはありません。 
2.    チャネルは V3 との互換性を維持し、中断またはサポート終了の予定はありません。
3.    新しい V3 ボットの作成はポータルで無効になっていますが、V3 ボットを ABS にデプロイするのではなく独立して (たとえば、webapp サービスとして) デプロイしたい上級ユーザーは、そのようにデプロイできます。

#### <a name="sdk-and-tools"></a>SDK とツール
1.    SDK サイドから V3 に投資することはなく、当面の間、重要なセキュリティ修正を SDK ブランチに適用するのみとなります (例外: V4 ボットからレガシ V3 ボットを呼び出せるよう、スキル コネクタの追加を計画しています)。
2.    SDK およびツールの開発は V4 のみで行われており、V3 に対する作業は実施も計画もされていません (Microsoft は既に "その段階" に達しています)。
3.    これは、ユーザーが各自の V3 ボットを管理するために古いツールを実行することを妨げるものではありません。 


## <a name="how-can-i-migrate-azure-bot-service-from-one-region-to-another"></a>Azure Bot Service をあるリージョンから別のリージョンに移行するにはどうすればよいですか?

Azure Bot Service は、リージョンの移動をサポートしていません。 特定のリージョンに関連付けられていないグローバル サービスです。

## <a name="channels"></a>チャンネル
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>Bot Framework にさらに多くの会話エクスペリエンスが追加されるのはいつですか?

Microsoft では、チャンネルの追加など、Bot Framework を継続的に改善していく予定ですが、現時点でスケジュールを提供することはできません。  
フレームワークに特定のチャンネルを追加することをご希望の場合は[お問い合わせください][Support]。

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>Bot Framework で構成可能にしたいコミュニケーション チャンネルがあります。 Microsoft と協力してこれを行うことはできますか?

開発者が Bot Framework に新しいチャンネルを追加する一般的なメカニズムは提供していませんが、[Direct Line API][DirectLineAPI] を使用してボットをアプリに接続できます。 お客様がコミュニケーション チャンネルの開発者であり、Microsoft と協力して Bot Framework でチャンネルを 有効にすることをご希望であれば、[ぜひお知らせください][Support]。

### <a name="if-i-want-to-create-a-bot-for-microsoft-teams-what-tools-and-services-should-i-use"></a>Microsoft Teams 用のボットを作成する場合、どのようなツールやサービスを使用すればよいですか?

Bot Framework は、Teams や他の多くのチャンネル向けに、高品質でパフォーマンスと応答性に優れたスケーラブルなボットを作成、接続、デプロイできるように設計されています。 SDK を使用して、(会話エクスペリエンス全体で現在のボットの対話において多くを占める) テキスト/SMS、画像、ボタン、カード対応のボットや、豊富なオーディオ/動画エクスペリエンスなどの Teams 固有のボットの対話を作成できます。

優れたボットが既にあり、Teams の対象ユーザーを獲得したい場合は、Bot Framework for REST API を使用して Teams (またはサポートされている任意のチャンネル) に簡単に接続できます (インターネットにアクセス可能な REST エンドポイントがある場合)。

### <a name="how-do-i-create-a-bot-that-uses-the-us-government-data-center"></a>米国政府データ センターを使用するボットはどのように作成しますか?

米国政府データ センターを使用するボットを作成するために必要となる主要な手順は 2 つあります。
1. appsettings.json (または App Service 設定) に "チャンネル プロバイダー" 設定を追加します。 これは、具体的にはこの名前/値の定数に設定する必要があります:ChannelService ="https://botframework.azure.us"。 以下に appsetting.json の使用例を示します。

```json
{
  "MicrosoftAppId": "", 
  "MicrosoftAppPassword": "",
  "ChannelService": "https://botframework.azure.us"
}
```
2. .NET core を使用している場合、startup.cs ファイルに ConfigurationChannelProvider を追加する必要があります。 この方法は、使用している SDK のバージョンによって異なります。

- バージョン 4.3 以降、ConfigureServices メソッドで、ConfigurationChannelProvider インスタンスを作成する必要があります。 BotFrameworkHttpAdapter クラスを使用している場合、これをシングルトンとしてサービス コレクションに次のように挿入します:

```csharp  
services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();
```
- 4\.3 より前のバージョンの場合、ConfigureServices メソッドで、AddBot メソッドを検索します。 オプションを設定するときは、必ず次を追加します:

```csharp
options.ChannelProvider = new ConfigurationChannelProvider();
```
政府機関のサービスに関する詳細については、[こちら](https://docs.microsoft.com/azure/azure-government/documentation-government-services-aiandcognitiveservices#azure-bot-service)を参照してください。

## <a name="security-and-privacy"></a>セキュリティとプライバシー
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>Bot Framework に登録されているボットは個人情報を収集するのでしょうか。 収集する場合、データの安全とセキュリティを確保するにはどうすればよいですか? プライバシーはどうですか?

各ボットは独自のサービスであり、これらのサービスの開発者には、開発者倫理規定に従ってサービス利用規約とプライバシーに関する声明を提供することが義務付けられています。 詳しくは、「[Bot レビュー ガイドライン](https://docs.microsoft.com/azure/bot-service/bot-service-review-guidelines?view=azure-bot-service-4.0)」をご覧ください。

### <a name="can-i-host-my-bot-on-my-own-servers"></a>自分のサーバー上でボットをホストできますか?
はい。 お使いのボットはインターネット上のあらゆる場所でホストできます。 これには、ご自身のサーバー、Azure、またはその他の任意のデータセンターが含まれます。 唯一の要件は、パブリックにアクセスできる HTTPS エンドポイントがボットによって公開されている必要があることです。

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>Microsoft はどのようにしてボットを禁止したり、サービスからボットを削除したりするのですか?

ユーザーは、ディレクトリ内のボットの連絡先カードを使用して、不正な動作をするボットを報告できます。 開発者がサービスに参加するには、Microsoft サービス利用規約に従う必要があります。

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>Bot Framework サービスにアクセスするために、会社のファイアウォールでホワイトリストに登録する必要がある URL を教えてください。

お客様のボットからインターネットへの送信トラフィックをブロックしているファイアウォールがある場合は、そのファイアウォールで次の URL をホワイトリストに登録する必要があります。

- `login.botframework.com` (ボット認証)
- `login.microsoftonline.com` (ボット認証)
- `westus.api.cognitive.microsoft.com` (Luis.ai NLP 統合の場合)
- `cortanabfchanneleastus.azurewebsites.net` (Cortana チャンネル)
- `cortanabfchannelwestus.azurewebsites.net` (Cortana チャンネル)
- `*.botframework.com` (チャンネル)
- `state.botframework.com` (下位互換性)
- 特定の Bot Framework チャンネルのその他の URL

> [!NOTE]
> アスタリスクを使用して URL をホワイトリストに登録するのを避けたい場合は、`<channel>.botframework.com` を使用できます。 `<channel>` は、ボットが使用するあらゆるチャネルに相当します (`directline.botframework.com`、`webchat.botframework.com`、`slack.botframework.com` など)。 ボットをテストする傍ら、ファイアウォール経由のトラフィックを監視して、他にブロックされているものがないか確認することも大切です。

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-framework-service"></a>Bot Framework Service からのトラフィックを除く、自分のボットへのすべてのトラフィックをブロックできますか?
Bot Framework サービスは世界中の Azure データセンターでホストされているため、Azure IP の一覧は常に変わっています。 特定の IP アドレスのホワイトリスト登録が有効なのは 1 日で、翌日に Azure IP アドレスが変わると無効になります。
 
### <a name="which-rbac-role-is-required-to-create-and-deploy-a-bot"></a>ボットの作成およびデプロイにはどの RBAC ロールが必要ですか?

Azure portal でボットを作成するには、サブスクリプションまたは特定のリソース グループのいずれかに共同作成者のアクセス権が必要です。 リソース グループの "*共同作成者*" ロールを持つユーザーは、その特定のリソース グループに新しいボットを作成できます。 サブスクリプションの "*共同作成者*" ロールのユーザーは、新規または既存のリソース グループにボットを作成できます。

Azure CLI を使用すると、ロールベースのアクセス制御の手法でカスタム ロールをサポートできます。 より制限されたアクセス許可を持つカスタム ロールを作成する場合は、次のセットを使用すると、LUIS、QnA Maker、および Application Insights もサポートするボットを作成およびデプロイできます。

  "Microsoft.Web/ *"、"Microsoft.BotService/* "、"Microsoft.Storage/ *"、"Microsoft.Resources/deployments/* "、"Microsoft.CognitiveServices/ *"、"Microsoft.Search/searchServices/* "、"Microsoft.Insights/ *"、"Microsoft.Insights/components/* "

LUIS と QnA Maker には Cognitive Services のアクセス許可が必要です。 また、QnA Maker には Search のアクセス許可も必要です。 カスタム ロールを作成する場合、継承された "*拒否*" のアクセス許可により、これらの "*許可*" のアクセス許可が上書きされることに注意してください。
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-service"></a>自分のボットは、Bot Framework Service を偽装するクライアントからどのように保護されますか?
1. すべての信頼できる Bot Framework 要求には、JWT トークンが付属しています。その暗号化署名は、[認証](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector)ガイドに従って検証できます。 このトークンは、信頼されたサービスを攻撃者が偽装できないように設計されています。

2. お使いのボットへのすべての要求に付いているセキュリティ トークンでは ServiceUrl がエンコードされています。つまり、攻撃者がトークンにアクセスしても、会話を新しい ServiceUrl にリダイレクトすることはできません。 これは、SDK のすべての実装によって適用され、Microsoft の認証[参考](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector)資料に記載されています。

3. 受信トークンが欠落しているか、形式に誤りがある場合、Bot Framework SDK の応答ではトークンが生成されません。 これにより、ボットが正しく構成されていない場合に発生する損害が制限されます。
4. ボット内で、トークンで提供される ServiceUrl を手動で確認できます。 これにより、サービス トポロジの変更が発生した場合にボットが脆弱になります。したがって、これは可能ですが、お勧めしません。


これらはボットからインターネットへの送信接続であることに注意してください。 ボットへの通信に Bot Framework Connector サービスによって使用される IP アドレスまたは DNS 名の一覧はありません。 受信 IP アドレスのホワイトリスト登録はサポートされていません。

## <a name="rate-limiting"></a>レート制限
### <a name="what-is-rate-limiting"></a>レート制限とは何ですか?
Bot Framework サービスは、どのボットも他のボットのパフォーマンスに悪影響を及ぼすことがないように、不正な呼び出しパターン (サービス拒否攻撃など) からサービス自体と顧客を保護する必要があります。 このような保護を実現するために、Microsoft のエンドポイントにレート制限 ("調整" とも呼ばれます) が追加されました。 レート制限を適用することで、クライアントまたはボットが特定の呼び出しを行うことができる頻度を制限できます。 たとえば、レート制限を有効にすると、ボットは多数のアクティビティを投稿する場合に、それらを一定期間にわたって分割して投稿する必要があります。 レート制限の目的は、ボットの総容量の上限を設けることではないことに注意してください。 人間の会話パターンに従っていない会話インフラストラクチャの不正使用を防ぐことを目的としています。 たとえば、2 人の人間が使用できる量を超えたコンテンツにより、2 つの会話を飽和状態にするなどです。

### <a name="how-will-i-know-if-im-impacted"></a>影響を受けているかどうかはどうすればわかりますか?
要求が大量であっても、お客様がレート制限を受けることはまずありません。 ほとんどのレート制限は、(ボットまたはクライアントからの) アクティビティの一括送信、極端なロード テスト、またはバグによってのみ発生します。 要求が調整されると、要求の再試行が成功するまでの待機時間 (秒単位) を示す Retry-After ヘッダーと共に、HTTP 429 (Too Many Requests) 応答が返されます。 この情報を収集するには、Azure Application Insights を使用してボットの分析を有効にします。 また、メッセージをログに記録するコードをボットに追加することもできます。 

### <a name="how-does-rate-limiting-occur"></a>レート制限はどのように発生しますか?
次の場合に発生する可能性があります。
-    ボットがメッセージを送信する頻度が高すぎる場合
-    ボットのクライアントがメッセージを送信する頻度が高すぎる場合
-    Direct Line クライアントが新しい Web ソケットを要求する頻度が高すぎる場合

### <a name="what-are-the-rate-limits"></a>レート制限値を教えてください。
サービスとユーザーを保護しながら、できるだけ緩やかなものにするために、レート制限値は継続的に調整されています。 しきい値が変更されることがあるため、現時点では数値を公開していません。 レート制限の影響を受けている場合は、[bf-reports@microsoft.com](mailto://bf-reports@microsoft.com) までお気軽にご連絡ください。

## <a name="bot-framework-sdk"></a>Bot Framework SDK
## <a name="why-doesnt-the-typing-activity-do-anything"></a>入力しても何も行われないのはなぜですか。
一部のチャネルでは、クライアントでの一時的な入力の更新がサポートされていません。

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>SDK のコネクタ ライブラリとビルダー ライブラリの違いは何ですか。

コネクタ ライブラリは、REST API の説明部分です。 ビルダー ライブラリには、会話ダイアログ プログラミング モデルやその他の機能 (プロンプト、ウォーターフォール、チェーン、ガイド付きフォーム入力など) が追加されています。 また、ビルダー ライブラリでは、LUIS などの Cognitive Services へのアクセスも提供されます。

## <a name="how-do-user-messages-relate-to-https-method-calls"></a>ユーザー メッセージは HTTPS メソッド呼び出しにどのように関連しますか。

ユーザーがチャネル経由でメッセージを送信するときに、Bot Framework Web サービスでは、ボットの Web サービス エンドポイントに対して HTTPS POST が発行されます。 ボットからは、そのチャネル上のユーザーに対して、0 件、1 件、または多くのメッセージが返信される場合があります。その場合、送信されるメッセージごとに Bot Framework に対して別の HTTPS POST が発行されます。


## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>ユーザーと自分のボット間のすべてのメッセージをインターセプトするにはどうすればよいですか。

Bot Framework SDK for .NET を使用することで、`Autofac` 依存関係挿入コンテナーに対して `IPostToBot` および `IBotToUser` インターフェイスの実装を提供することができます。 任意の言語に Bot Framework SDK を使用すれば、ほぼ同じ目的でミドルウェアを利用できます。 [BotBuilder Azure](https://github.com/Microsoft/BotBuilder-Azure) リポジトリには、C# および Node.js のライブラリが含まれており、Azure テーブルにこのデータが記録されます。

## <a name="what-is-the-idialogstackforward-method-in-the-bot-framework-sdk-for-net"></a>Bot Framework SDK for .NET の IDialogStack.Forward メソッドとは何ですか。

`IDialogStack.Forward` の主な目的は、既存の子ダイアログを再利用することです。このダイアログは多くの場合、"リアクティブ" であり、(`IDialog.StartAsync` の) 子ダイアログは `ResumeAfter` ハンドラーがあるオブジェクト `T` を待機します。 特に、`IMessageActivity` `T` を待機する子ダイアログがある場合、`IDialogStack.Forward` メソッドを使用して、着信 `IMessageActivity` (親ダイアログで既に受信されている) を転送できます。 たとえば、着信 `IMessageActivity` を `LuisDialog` に転送するには、`IDialogStack.Forward` を呼び出して `LuisDialog` をダイアログ スタックにプッシュし、次のメッセージの待機がスケジュールされるまで `LuisDialog.StartAsync` でコードを実行し、転送された `IMessageActivity` ですぐにその待機が満たされるようにします。

`T` は通常、`IMessageActivity` です。これは、`IDialog.StartAsync` が一般的にその種のアクティビティを待機するように構成されるためです。 既存の `LuisDialog` にメッセージを転送する前に、何らかの処理のためにユーザーからのメッセージをインターセプトするメカニズムとして、`LuisDialog` に対して `IDialogStack.Forward` を使用することができます。 また、その目的で `DispatchDialog` と `ContinueToNextGroup` を使用することもできます。

`StartAsync` によってスケジュールされる最初の `ResumeAfter` ハンドラー (`LuisDialog.MessageReceived` など) で、転送された項目が見つかります。


## <a name="what-is-the-difference-between-proactive-and-reactive"></a>"プロアクティブ" と "リアクティブ" の違いは何ですか。

ボットの観点から、"リアクティブ" は、ユーザーがメッセージをボットに送信して会話を開始し、ボットでそのメッセージに応答して反応することを意味します。 これに対して、"プロアクティブ" は、ボットでユーザーに最初のメッセージを送信して会話を開始することを意味します。 たとえば、タイマーの期限が切れたときやイベントが発生したときに、ユーザーに通知するプロアクティブ メッセージがボットから送信される場合があります。

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>ユーザーにプロアクティブ メッセージを送信するにはどうすればよいですか。

プロアクティブ メッセージを送信する方法を示す例については、GitHub の BotBuilder-Samples リポジトリ内にある [C# の V4 のサンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages)と [Node.js の V4 のサンプル](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)をご覧ください。

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs-in-sdk-v3"></a>SDK v3 の自分の C# ダイアログからシリアル化できないサービスを参照するにはどうすればよいですか。

次のような複数のオプションがあります。

* `Autofac` と `FiberModule.Key_DoNotSerialize` を使用して依存関係を解決します。 これは最もクリーンなソリューションです。
* [NonSerialized](https://msdn.microsoft.com/library/system.nonserializedattribute(v=vs.110).aspx) および [OnDeserialized](https://msdn.microsoft.com/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) 属性を使用して、逆シリアル化時に依存関係を復元します。 これは最もシンプルなソリューションです。
* シリアル化されなくなるため、その依存関係は格納しないでください。 このソリューションは、技術的には実行可能ですがお勧めしません。
* リフレクション シリアル化サロゲートを使用します。 このソリューションは、場合によっては実行できない可能性があり、シリアル化が過剰になるリスクがあります。

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>ETag とは何ですか。  ボット データ バッグ ストレージにどのように関連しますか。

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag) は、[オプティミスティック コンカレンシー](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)のためのメカニズムです。 ボット データ バッグ ストレージでは ETag を使用して、データ更新の競合を防ぎます。 HTTP 状態コード 412 "必須条件に失敗しました" の ETag エラーは、ボットが最初の操作を完了する前に複数のメッセージが並列で受信されたことを示します。

ダイアログ スタックおよび状態は、ボット データ バッグに格納されます。 たとえば、ご利用のボットで、該当する会話の新しいメッセージの受信時に前のメッセージがまだ処理されている場合、"前提条件が満たされていません" という ETag エラーが表示されることがあります。

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>LUIS の組み込みの日付、時刻、期間、およびセット エンティティのコンピューターで読み取り可能な解決策として何が考えられますか。

例の一覧については、LUIS ドキュメントの[作成済みエンティティ](/azure/cognitive-services/LUIS/luis-reference-prebuilt-entities)に関するセクションを参照してください。

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>最大数を超える LUIS の意図を使用するにはどうすればよいですか。

ご利用のモデルを分割し、LUIS サービスを順次または並行して呼び出すことを検討してください。

## <a name="how-can-i-use-more-than-one-luis-model"></a>複数の LUIS モデルを使用するにはどうすればよいですか。

Bot Framework SDK for Node.js と Bot Framework SDK for .NET の両方で、単一の LUIS 意図ダイアログからの複数の LUIS モデルの呼び出しがサポートされています。 以下の点に注意してください。

* 複数の LUIS モデルを使用する場合、LUIS モデルに含まれる意図のセットが重複していないことが前提となります。
* 複数の LUIS モデルを使用する場合、複数のモデル間で "最も一致する意図" を選択するために、さまざまなモデルからのスコアを比較できることが前提となります。
* 複数の LUIS モデルを使用することは、意図が 1 つのモデルと一致した場合に、他のモデルの "none" 意図にも厳密に一致することを意味します。 この場合、"none" 意図の選択を回避することができます。Bot Framework SDK for Node.js では、この問題を回避するために、"none" 意図のスコアが自動的に下げられます。

## <a name="where-can-i-get-more-help-on-luis"></a>LUIS の詳細情報はどこで入手できますか。

* [Language Understanding (LUIS) の概要 - Microsoft Cognitive Services](https://www.youtube.com/watch?v=jWeLajon9M8) (ビデオ)
* [Language Understanding (LUIS) の高度な学習セッション](https://www.youtube.com/watch?v=39L0Gv2EcSk) (ビデオ)
* [LUIS のドキュメント](/azure/cognitive-services/luis/)
* [Language Understanding のフォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>コミュニティで作成されるダイアログをいくつか教えてください。

* [BotAuth](https://www.nuget.org/packages/BotAuth) - Azure Active Directory 認証
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/) -ダイアログ メソッドに対するユーザー テキストの正規表現ベースのディスパッチ

## <a name="what-are-some-community-authored-templates"></a>コミュニティで作成されるテンプレートをいくつか教えてください。

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template) - ES6 Bot Builder のテンプレート

## <a name="related-services"></a>関連サービス
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Bot Framework は Cognitive Services にどのように関連していますか?

Bot Framework と [Cognitive Services](https://www.microsoft.com/cognitive) はいずれも、長年にわたる研究と一般的な Microsoft 製品での使用に基づいて構築されています。 これらの機能により、あらゆる組織が、データの力、クラウド、インテリジェンスを活用して、新たなチャンスを切り開き、ビジネスを加速させ、顧客にサービスを提供する業界をリードする独自のインテリジェント システムを構築できます。

### <a name="what-is-the-direct-line-channel"></a>Direct Line チャンネルとは何ですか?

Direct Line は、サービス、モバイル アプリ、または Web ページにボットを追加できる REST API です。

Direct Line API のクライアントは、任意の言語で記述できます。 [Direct Line プロトコル][DirectLineAPI]にコード化し、Direct Line 構成ページでシークレットを生成して、コードが存在する場所からボットに話しかけるだけです。

Direct Line は以下に適しています。

* iOS、Android、Windows Phone などの OS 上のモバイル アプリ
* Windows、OSX などの OS 上のデスクトップ アプリケーション
* [埋め込み可能な Web チャット チャンネル][WebChat]が提供するものよりも多くのカスタマイズが必要な Web ページ
* サービス間アプリケーション


## <a name="app-registration"></a>アプリケーションの登録

### <a name="i-need-to-manually-create-my-app-registration-how-do-i-create-my-own-app-registration"></a>自分のアプリケーションの登録を手動で作成する必要があります。 独自のアプリケーションの登録はどのように作成するのですか?

独自のアプリケーションの登録の作成は、次のような状況で必要になります。

- Bot Framework ポータル (https://dev.botframework.com/bots/new) など) で、ボットを作成しました。 
- 自分の組織でアプリケーションの登録をすることができず、構築しているボット用のアプリ ID を作成するために別のパーティーが必要になります。
- それ以外の場合、独自のアプリ ID (とパスワード) を手動で作成する必要があります。

独自のアプリ ID を作成するには、次の手順に従ってください。

1. [Azure アカウント](https://portal.azure.com)にサインインします。 Azure アカウントを持っていない場合は、[無料のアカウントにサインアップ ](https://azure.microsoft.com/free/) できます。
2. [[アプリの登録] ブレード ](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) にアクセスし、上部にある操作バーの **[新しい登録]** をクリックします。

    ![新しい登録](media/app-registration/new-registration.png)

3. *[名前]* フィールドにアプリケーション登録の表示名を入力し、サポートされているアカウントの種類を選択します。 この名前は、ボット ID と一致させる必要はありません。

    > [!IMPORTANT]
    > *[サポートされているアカウントの種類]* で、 *[任意の組織のディレクトリ内のアカウントと、個人用の Microsoft アカウント (Xbox、Outlook.com など)]* ラジオ ボタンを選択します。 その他のオプションのいずれかを選択すると、**ボットが使用できなくなります**。

    ![登録の詳細](media/app-registration/registration-details.png)

4. **[登録]** をクリックします。

    しばらくすると、新しく作成されたアプリケーションの登録によって、ブレードが開きます。 [概要] ブレードの *[アプリケーション クライアント ID]* をコピーし、[アプリ ID] フィールドに貼り付けます。

    ![アプリケーション ID](media/app-registration/app-id.png)

Bot Framework ポータルを介してボットを作成している場合、アプリケーションの登録の設定は完了しています。シークレットは自動的に生成されます。 

Azure portal でボットを作成している場合、アプリケーションの登録用のシークレットを生成する必要があります。 

1. アプリケーション登録のブレードの左側のナビゲーション列にある **[Certificates & secrets]\(証明書とシークレット\)** をクリックします。
2. そのブレードで、 **[New client secret]\(新しいクライアント シークレット\)** ボタンをクリックします。 ポップアップ表示されるダイアログ ボックスで、シークレットのオプションの説明を入力し、[有効期限] ラジオ ボタン グループから **[無期限]** を選択します。 

    ![新しいシークレット](media/app-registration/new-secret.png)

3. *[クライアント シークレット]* の下の表からシークレットの値をコピーして、アプリケーションの *[パスワード]* フィールドに貼り付け、そのブレードの下部にある **[OK]** をクリックします。 ボットの作成を続行します。 

    > [!NOTE]
    > シークレットは、このブレード上にいる間のみ表示されます。そのページから移動すると、取得することができなくなります。 必ず、これを安全な場所にコピーしてください。

    ![新しいアプリケーション ID](media/app-registration/create-app-id.png)


[DirectLineAPI]: https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md
