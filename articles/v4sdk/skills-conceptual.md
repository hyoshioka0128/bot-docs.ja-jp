---
title: スキルについて | Microsoft Docs
description: Bot Framework SDK を使用して、あるボットの会話ロジックを別のボットから使用できるようにする方法について説明します。
keywords: ボット スキル, ホスト ボット, スキル ボット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e4f11b6a1e8bcfce06e1518db7865d6b65ec3d62
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2020
ms.locfileid: "76753776"
---
# <a name="about-skills"></a>スキルについて

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

<!-- Value prop: why skills -->
Bot Framework SDK のバージョン 4.7 以降では、別のボット (スキル) を使用してボットを拡張することができます。
スキルは、他のさまざまなボットで使用されて再利用を促進します。これにより、ユーザー向けに作成したボットを、独自のスキルやサードパーティのスキルを使用して拡張できます。

<!-- Terminology -->
- "_スキル_" は別のボットに対して一連のタスクを実行できるボットであり、マニフェストを使用してそのインターフェイスが記述されます。
  設計によっては、スキルを一般的なユーザー向けボットとして機能させることもできます。
- "_スキル コンシューマー_" は、1 つ以上のスキルを呼び出すことができるボットです。 スキルに関しては、"_ルート ボット_" がユーザー向けボットとスキル コンシューマーを兼ねています。
- "_スキル マニフェスト_" は、スキルが実行できるアクティビティ、スキルの入力パラメーターと出力パラメーター、およびスキルのエンドポイントが記述された JSON ファイルです。
  - スキルのソース コードにアクセスできない開発者は、マニフェストの情報を使用してスキル コンシューマーを設計できます。 サンプルのスキル マニフェストについては、「[スキルを実装する](./skill-implement-skill.md)」を参照してください。
  - "_スキル マニフェスト スキーマ_" は、スキル マニフェストのスキーマが記述された JSON ファイルです。 現在のバージョンは [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) です。

つまり、ユーザーはルート ボットと直接対話し、ルート ボットはその会話ロジックの一部をスキルに委任します。

<!-- Requirements/contract -->
スキル機能は次のように設計されています。

- スキルは、Bot Framework アダプターとカスタム アダプターの両方と連携できます。
- スキルは Microsoft Teams チャネルと連携でき、そのときに `invoke` アクティビティが多用されます。
- スキルはユーザー認証をサポートしますが、ユーザー認証はスキルでローカルに行われ、別のボットに転送することはできません。
- スキル コンシューマーは複数のスキルを使用できます。
- スキル コンシューマーは複数のスキルを並行して実行できます。
- Bot Connector サービスでボット間認証が提供されますが、エミュレーターを使用してルート ボットをローカルにテストすることができます。
<!--TBD: - Skills support proactive messaging. -->
<!--TBD: - A skill can also be a skill consumer. -->

## <a name="architecture"></a>Architecture

スキルとスキル コンシューマーは別々のボットであり、ユーザーはそれらを個別に公開します。

- スキルとスキル コンシューマーは HTTP を介して通信します。
- スキル コンシューマーは複数のスキルを呼び出すことができます。
- 1 つのスキルを複数のコンシューマーから呼び出すことができます。

スキルには、完了時に `endOfConversation` アクティビティを送信するロジックを追加で組み込む必要があります。これにより、スキル コンシューマーはスキルへのアクティビティの転送を停止するタイミングを認識できます。

ルート ボットは少なくとも 2 つの HTTP エンドポイントを実装します。1 つはユーザーからアクティビティを受信し、もう 1 つはスキルからアクティビティを受信します。 スキル コンシューマーでは、HTTP メソッド要求を受信するコードとスキル ハンドラーを組み合わせる必要があります。
また、スキルを管理する (スキルの呼び出しやキャンセルなど) ためのロジックを追加する必要があります。 コンシューマーには、通常のボット オブジェクトとアダプター オブジェクトに加えて、アクティビティをスキルとやり取りするために使用されるスキル関連オブジェクトがいくつか含まれています。

次の図は、ユーザーから送信されたアクティビティがルート ボットを経由してスキルに到達し、再びユーザーに戻るまでのフローを示しています。

![アーキテクチャの図](./media/skills-conceptual-architecture.png)

1. ルート ボットのアダプターは、ユーザーからアクティビティを受信して、ルート ボットのアクティビティ ハンドラーに転送します。
1. ルート ボットは、スキル HTTP クライアントを使用してアクティビティをスキルに送信します。 さらに、スキル定義とスキル会話 ID ファクトリからコンシューマー/スキル間の会話の情報を取得します。 これには、スキルがアクティビティへの応答に使用するサービス URL が含まれます。
1. スキルのアダプターは、スキル コンシューマーからアクティビティを受信して、スキルのアクティビティ ハンドラーに転送します。
1. スキルが応答すると、ルート ボットのスキル ハンドラーはアクティビティを受信します。 さらに、スキル会話 ID ファクトリからルート/ユーザー間の会話の情報を取得します。 その後に、アクティビティをルート ボットのアダプターに転送します。
1. ルート ボットのアダプターは、内部でプロアクティブ メッセージを生成して、ユーザーとの会話を再開します。
1. ルート ボットのアダプターは、スキルからのメッセージをすべてユーザーに送信します。

スキルの管理とスキル トラフィックのルーティングには、次のオブジェクトを利用します。

- "_Bot Framework スキル_" は、スキルのルーティング情報を記述したもので、スキル コンシューマーの構成ファイルから読み取られます。
- "_スキル HTTP クライアント_" は、スキルにアクティビティを送信します。
- "_スキル ハンドラー_" は、スキルからアクティビティを受信します。
- "_スキル会話 ID ファクトリ_" は、ユーザー/ルート間の会話リファレンスとルート/スキル間の会話リファレンスの間で変換を行います。
- Bot Connector サービスは、チャネルとボット間認証の両方を提供します。 "_認証構成_" オブジェクトを使用して要求検証をボット (スキルまたはスキル コンシューマー) に追加することにより、アクセスできるアプリケーションやユーザーを制限できます。

スキル クライアントとスキル ハンドラーの両オブジェクトは、"_会話 ID ファクトリ_" を使用して、ルート ボットがユーザーとの対話に使用する会話と、ルート ボットがスキルとの対話に使用する会話の間で変換を行います。

## <a name="bot-to-bot-communication"></a>ボット間通信

設計するボットの種類に関係なく、この設計の一定の側面を理解しておくことが重要です。

<!--
- infrastructure concerns:
  - stateless, cross-server application (memory management and middleware).
  - authentication, in both directions, plus claims validation.
- implementation concerns:
  - classes, objects, and logic you need to add to your host (and skill).
  - when to start and stop a skill.
  - managing multiple skills.
-->

### <a name="conversation-references"></a>会話リファレンス

ユーザー/ルート間の会話とルート/スキル間の会話は異なります。

"_会話 ID ファクトリ_" は、スキル コンシューマーとスキルの間のトラフィックの管理を支援します。 このファクトリは、ルート/ユーザー間の会話 ID とルート/スキル間の会話 ID の間で変換を行います。
つまり、ルートとスキルの間で使用される会話 ID を生成し、その ID から元のユーザー/ルート間の会話 ID を復旧します。

<!-- Hopefully, this just gets folded into the SDK and does not need to get described in detail.
- The host needs to save or encode original conversation ID and service URL and create a conversation ID for use between it and the skill.
  - Generated conversation IDs must be usable as a URL path parameter.
  - A modified activity gets the new conversation ID and service URL (of the host, as the host provides a channel interface to the skill).
- Upon receiving an activity from the skill, host needs to decode or recover the original conversation ID and service URL so that the activity can get forwarded back to the user in the original conversation.
-->

### <a name="cross-server-coordination"></a>サーバー間の調整
<!-- or, Statelessness in the host -->

ルートとスキルは HTTP を介して通信します。
そのため、スキルからアクティビティを受信するルート インスタンスと、開始アクティビティを送信したインスタンスが同じでない場合があります。つまり、この 2 つの要求が別々のサーバーで処理される可能性があります。

- アクティビティをスキルに転送する前に、必ずスキル コンシューマーで状態を保存してください。
  これにより、スキルからトラフィックを受信するインスタンスは、スキルを呼び出す前に前のインスタンスが中断した箇所から処理を再開できます。
- スキル ハンドラーは、スキルからアクティビティを受信すると、それをスキル コンシューマーに適した形式に変換し、コンシューマーのアダプターに転送します。

### <a name="skill-consumer-and-skill-state"></a>スキル コンシューマーとスキルの状態

スキル コンシューマーとスキルは、それぞれの状態を別々に管理します。 ただしコンシューマーでは、スキルとの通信に使用する会話 ID が作成されます。 これがスキルの会話状態に影響することがあります。

> [!IMPORTANT]
> 既に説明したように、スキル コンシューマーがユーザー アクティビティをスキルに委任した場合に、そのコンシューマーの別のインスタンスがスキルの応答を受信することがあります。 コンシューマーは、アクティビティをスキルに転送する直前に、会話状態を保存する必要があります。

### <a name="bot-to-bot-authentication"></a>ボット間認証

<!-- TODO Add appropriate info about this new(?) feature to the bot basics article. -->

サービス レベルの認証は、Bot Connector サービスで管理されます。 このフレームワークでは、ベアラー トークンとボット アプリケーション ID を使用して各ボットの ID が検証されます。

Bot Framework では、"_認証構成_" オブジェクトを使用して、受信した要求の認証ヘッダーが検証されます。

#### <a name="claims-validation"></a>要求検証

認証構成に "_要求検証コントロール_" を追加することができます。 要求は認証ヘッダーの後に評価されます。 要求を拒否する場合は、検証コードからエラーまたは例外をスローする必要があります。

通常であれば認証される要求を、さまざまな理由で拒否することができます。

- スキル コンシューマーが、会話を開始した相手である可能性があるスキルからのトラフィックのみを受け入れる必要がある場合。
- スキルが有料サービスの一部であり、データベースに登録されていないユーザーのアクセスを禁止する必要がある場合。
- スキルへのアクセスを特定のスキル コンシューマーに限定する場合。

<!--TODO Need a link for more information about claims and claims-based validation.-->

## <a name="skill-consumers"></a>スキル コンシューマー

<!--
### Skill consumer design
Supports: Bot Framework adapter, custom adapters, MS Teams channel (what Teams calls "proactive" 1-1 conversations, created off a group/channel/team conversation)
-->

ユーザーにとってルート ボットは、対話の相手であるボットです。
スキルにとってスキル コンシューマーは、ユーザーとの通信に使用するチャネルです。

スキル コンシューマーでもあるルート ボットには、スキルとの間のトラフィックを管理するためのロジックが追加されています。

- ルートが使用する各スキルの構成情報。
- ルートがユーザーとの会話とスキルとの会話を切り替えるために使用する "_会話 ID ファクトリ_"。
- アクティビティをパッケージ化してスキル ボットに転送する "_スキル クライアント_"。
- スキル ボットからアクティビティを受信してアンパックする "_スキル ハンドラー_"。

### <a name="managing-skills"></a>スキルの管理

1 つのスキルの開始から完了までは、ホストに追加されたいくつかの機能で管理されます。 複数のスキルや会話スレッドを使用する複雑なシナリオにも対応できます。

#### <a name="skill-descriptions"></a>スキルの説明

スキルごとに、"_Bot Framework スキル_" オブジェクトをスキル コンシューマーの構成ファイルに追加します。 各オブジェクトには ID、アプリ ID、およびスキルのエンドポイントが設定されます。

| プロパティ | [説明]
| :--- | :--- |
| _ID_ | スキルの ID またはキー (スキル コンシューマーに固有)。 |
| _App ID_ | スキルが Azure に登録されたときにボット リソースに割り当てられた `appId`。 |
| _Skill endpoint_ | スキルのメッセージング エンドポイント。 コンシューマーがスキルとの通信に使用する URL です。 |

#### <a name="skill-client-and-skill-handler"></a>スキル クライアントとスキル ハンドラー

<!-- Is this still accurate? -->
スキル コンシューマーは、スキル クライアントを使用してアクティビティをスキルに送信します。 クライアントは次のことを行います。

- ユーザーのアクティビティまたはコンシューマーで生成されたアクティビティを取得してスキルに転送します。
- 元の会話リファレンスをコンシューマー/スキル間の会話リファレンスで置換します。
- ボット間認証トークンを追加します。
- 更新されたアクティビティをスキルに送信します。

スキル コンシューマーは、スキル ハンドラーを使用してスキルからアクティビティを受信します。 ハンドラーは次のことを行います。

- チャネルサービスの REST API メソッド `SendToConversation` と `ReplyToActivity` を処理します。
- 認証と要求検証を実施します。
- 元の会話の参照を取得します。
- コンシューマーのアダプターに対してアクティビティを生成します。 このアクティビティは、スキルが完了したことを通知するか、またはユーザーに転送されます。

<!-- These section should probably reference a how-to. -->
#### <a name="managing-a-skill-from-a-skill-consumer"></a>スキル コンシューマーからのスキルの管理

アクティブなスキルを追跡するには、スキル コンシューマーにロジックを追加する必要があります。
スキルの全般的な管理方法や、複数のアクティブなスキルを並行して管理できるかどうかは、コンシューマーが判断します。
検討の対象となるシナリオは次のとおりです。

- コンシューマー/スキル間の会話を新規に開始する。 (特定のコンシューマー/ユーザー間の会話に関連付けられます。)
  - スキルにパラメーターを渡すには、初期アクティビティの _value_ プロパティをそのスキルに設定します。
- 既存のコンシューマ/スキル間の会話を続行する。
- スキルからの `endOfConversation` アクティビティを、コンシューマー/スキル間の会話終了の通知として認識する。
  - スキルからの戻り値を取得するには、アクティビティの _value_ プロパティを確認します。
  - スキルが終了する理由を確認するには、アクティビティの _code_ パラメーターを確認します (スキルでエラーが発生したことを示している可能性があります)。
- `endOfConversation` アクティビティをスキルに送信して、コンシューマーからスキルをキャンセルする。

## <a name="skill-bots"></a>スキル ボット

小さな変更を加えることで、どのボットもスキルとして使用できます。 スキル ボットには次の機能があります。

- 要求検証コントロールによってアクセス制限を実装します。
- 必要に応じて、初期アクティビティの _value_ プロパティで初期化パラメーターを確認します。
- `message` アクティビティを通じて、通常どおりにユーザーにメッセージを送信します。
- `endOfConversation` アクティビティを通じて、スキルの完了またはキャンセルを通知します。
  - アクティビティの _value_ プロパティに戻り値 (存在する場合) を提供します。
  - アクティビティの _code_ プロパティにエラー コードを提供します (存在する場合)。

### <a name="skill-manifest"></a>スキル マニフェスト

スキル コンシューマーは必ずしもスキル コードにアクセスできるわけではないため、スキル マニフェストを使用して、スキルが受信および生成できるアクティビティ、スキルの入力パラメーターと出力パラメーター、およびスキルのエンドポイントを記述します。 スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="additional-information"></a>関連情報

スキルとスキル コンシューマーのボットを実装する方法については、「[スキルを実装する](skill-implement-skill.md)」および「[スキル コンシューマーを実装する](skill-implement-consumer.md)」を参照してください。

<!--
You publish skill and skill consumer bots separately. For more information, see _TBD: link to create a skills bot from scratch how-to or tutorial_.

For more about skill manifests, see the _TBD: creating a manifest section of implementing a skill bot_.
-->
