---
title: スキルを実装する | Microsoft Docs
description: Bot Framework SDK を使用してスキルを実装する方法について説明します。
keywords: スキル
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f5ec4d8b336f35eb0e66cdb16e4b8eb36317b08a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80648227"
---
# <a name="implement-a-skill"></a>スキルを実装する

[!INCLUDE[applies-to](../includes/applies-to.md)]

スキルを使用して別のボットを拡張することができます。
"_スキル_" は、別のボットに対して一連のタスクを実行できるボットです。

- スキルのインターフェイスはマニフェストで記述されます。 スキルのソース コードにアクセスできない開発者は、マニフェストの情報を使用してスキル コンシューマーを設計できます。
- スキルでは、要求検証を使用して、スキルにアクセスできるボットやユーザーを管理できます。

この記事では、ユーザーの入力をエコーするスキルを実装する方法について説明します。

<!-- I haven't discussed passing values back-and-forth mid conversation. That could be the basis of another article. -->

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)と[スキル](skills-conceptual.md)に関する知識。
- Azure サブスクリプション。 お持ちでない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。
- **skills simple bot-to-bot** サンプルのコピー ([**C#** ](https://aka.ms/skills-simple-bot-to-bot-csharp)、[**JavaScript**](https://aka.ms/skills-simple-bot-to-bot-js)、または [**Python**](https://aka.ms/skills-simple-bot-to-bot-python))。

## <a name="about-this-sample"></a>このサンプルについて

**skills simple bot-to-bot** サンプルには、次の 2 つのボットのプロジェクトが含まれています。

- スキルを実装する "_エコー スキル ボット_"。
- スキルを使用するルート ボットを実装する "_単純なルート ボット_"。

この記事では、ボットとアダプターにサポート ロジックが組み込まれているスキルに焦点を当てます。

### <a name="c"></a>[C#](#tab/cs)

![スキルのクラス ダイアグラム](./media/skills-simple-skill-cs.png)

### <a name="javascript"></a>[JavaScript](#tab/javascript)

![スキルのクラス ダイアグラム](./media/skills-simple-skill-js.png)

### <a name="python"></a>[Python](#tab/python)

![スキルのクラス ダイアグラム](./media/skills-simple-skill-python.png)

---

単純なルート ボットについては、「[スキル コンシューマーを実装する](skill-implement-consumer.md)」を参照してください。

## <a name="resources"></a>リソース

ボット間認証を行うには、参加するボットのそれぞれに有効なアプリ ID とパスワードが必要です。

スキルをユーザー向けボットとしてテストできるようにするには、スキルを Azure に登録します。 Bot Channels Registration を使用できます。 詳細については、「[ボットを Azure Bot Service に登録する](../bot-service-quickstart-registration.md)」を参照してください。

## <a name="application-configuration"></a>アプリケーションの構成

スキルのアプリ ID とパスワードをスキルの構成ファイルに追加します。

"_許可される呼び出し元_" の配列 (AllowedCallers) では、スキルにアクセスできるスキル コンシューマーを制限できます。
すべてのスキル コンシューマーからの呼び出しを受け入れるには、この配列を空のままにします。

### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\appsettings.json**

スキルのアプリ ID とパスワードを appsettings.json ファイルに追加します。

[!code-csharp[configuration file](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/appsettings.json)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/.env**

スキルのアプリ ID とパスワードを .env ファイルに追加します。

[!code-javascript[configuration file](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/.env)]

### <a name="python"></a>[Python](#tab/python)

スキルのアプリ ID とパスワードを config.py ファイルに追加します。

**config.py**

[!code-python[configuration file](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/config.py?range=14-19)]

---

## <a name="activity-handler-logic"></a>アクティビティ ハンドラーのロジック

### <a name="to-accept-input-parameters"></a>入力パラメーターを受け入れる

スキル コンシューマーはスキルに情報を送信できます。 この情報を受け入れる方法の 1 つは、受信したメッセージの _value_ プロパティを介して受け入れることです。 もう 1 つの方法は、イベントを処理し、アクティビティを呼び出すことです。

この例のスキルは入力パラメーターを受け入れないようになっています。

### <a name="to-continue-or-complete-a-conversation"></a>会話を続行または完了する

スキルからアクティビティが送信されると、スキル コンシューマーからユーザーにそのアクティビティが転送されます。

ただし、スキルが終了したときは `endOfConversation` アクティビティを送信する必要があります。そうしないと、スキル コンシューマーからスキルにユーザー アクティビティが転送され続けます。
必要に応じて、アクティビティの _value_ プロパティに戻り値を入れ、アクティビティの _code_ プロパティを使用してスキルが終了する理由を示します。

#### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\Bots\EchoBot.cs**

[!code-csharp[Message handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Bots/EchoBot.cs?range=13-28)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/bot.js**

[!code-javascript[Message handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/bot.js?range=10-26)]

#### <a name="python"></a>[Python](#tab/python)

**echo-skill-bot/bots/echo_bot.py**

[!code-python[Message handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/bots/echo_bot.py?range=10-26)]

---

### <a name="to-cancel-the-skill"></a>スキルをキャンセルする

複数ターン スキルの場合は、スキル コンシューマーから `endOfConversation` アクティビティを受け入れて、コンシューマーが現在の会話をキャンセルできるようにすることもできます。

このスキルのロジックはターンごとに変わりません。 会話リソースを割り当てるスキルを実装する場合は、会話終了ハンドラーにリソース クリーンアップ コードを追加します。

#### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\Bots\EchoBot.cs**

[!code-csharp[End-of-conversation handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Bots/EchoBot.cs?range=30-36)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/bot.js**

 `onUnrecognizedActivityType` メソッドを使用して、会話終了ロジックを追加します。 ハンドラー内で、認識できないアクティビティの `type` が `endOfConversation` と等しいかどうかを確認します。

[!code-javascript[End-of-conversation handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/bot.js?range=28-35)]

#### <a name="python"></a>[Python](#tab/python)

**echo-skill-bot/bots/echo_bot.py**

[!code-python[End-of-conversation handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/bots/echo_bot.py?range=29-33)]

---

## <a name="claims-validator"></a>要求検証コントロール

このサンプルでは、許可される呼び出し元のリストが要求検証に使用されます。 このリストは、スキルの構成ファイルで定義され、作成時に検証コントロール オブジェクトに読み込まれます。

認証構成に "_要求検証コントロール_" を追加することができます。 要求は認証ヘッダーの後に評価されます。 要求を拒否する場合は、検証コードからエラーまたは例外をスローする必要があります。 通常であれば認証される要求を、さまざまな理由で拒否することができます。 次に例を示します。

- スキルが有料サービスの一部である。 データベースに登録されていないユーザーはアクセスできないようにする必要があります。
- スキルが専用のものである。 特定のスキル コンシューマーのみがスキルを呼び出すことができます。

<!--TODO Need a link for more information about claims and claims-based validation.-->

### <a name="c"></a>[C#](#tab/cs)

`ClaimsValidator` クラスから要求検証コントロールを派生させます。 これにより、受信した要求を拒否するための `UnauthorizedAccessException` がスローされます。 構成ファイルの値が空の配列であると、`IConfiguration.Get` メソッドから null が返されるので注意してください。

**EchoSkillBot\Authentication\AllowedCallersClaimsValidator.cs**

[!code-csharp[Claims validator](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Authentication/AllowedCallersClaimsValidator.cs?range=22-52&highlight=24-27)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

受信した要求を拒否するためにエラーをスローする要求検証メソッドを定義します。

**echo-skill-bot/authentication/allowedCallersClaimsValidator.js**

[!code-javascript[Claims validator](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/authentication/allowedCallersClaimsValidator.js?range=6-27&highlight=18-20)]

### <a name="python"></a>[Python](#tab/python)

受信した要求を拒否するためにエラーをスローする要求検証メソッドを定義します。

**echo-skill-bot/authentication/allowed_callers_claims_validator.py**

[!code-python[Claims validator](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/authentication/allowed_callers_claims_validator.py?range=7-28&highlight=17-22)]

---

## <a name="skill-adapter"></a>スキル アダプター

エラーが発生した場合は、スキルのアダプターは、スキルの会話状態をクリアするとともに、スキル コンシューマーに `endOfConversation` アクティビティを送信する必要があります。 アクティビティの _code_ プロパティを使用して、エラーのためにスキルが終了したことを通知します。

### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\SkillAdapterWithErrorHandler.cs**

[!code-csharp[Error handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/SkillAdapterWithErrorHandler.cs?range=20-59&highlight=19-24)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/index.js**

[!code-javascript[Error handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/index.js?range=41-69&highlight=21-28)]

### <a name="python"></a>[Python](#tab/python)

**echo-skill-bot/app.py**

[!code-python[Error handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/app.py?range=38-69&highlight=27-32)]

---

## <a name="service-registration"></a>サービス登録

"_Bot Framework アダプター_" は "_認証構成_" オブジェクト (アダプターの作成時に設定) を使用して、受信した要求の認証ヘッダーを検証します。

次のサンプルでは、要求検証を認証構成に追加し、前のセクションで説明した "_エラー ハンドラー付きスキル アダプター_" を使用しています。

### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\Startup.cs**

[!code-csharp[Configuration](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Startup.cs?range=28-32)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/index.js**

[!code-javascript[configuration](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/index.js?range=34-38)]

<!--C# & JS snippets checked 1/14-->
### <a name="python"></a>[Python](#tab/python)

**app.py**

[!code-python[configuration](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/app.py?range=22-34)]

---

## <a name="skill-manifest"></a>スキル マニフェスト

"_スキル マニフェスト_" は、スキルが実行できるアクティビティ、その入力パラメーターと出力パラメーター、およびスキルのエンドポイントが記述された JSON ファイルです。
マニフェストには、別のボットからスキルにアクセスするために必要な情報が含まれています。

### <a name="c"></a>[C#](#tab/cs)

**EchoSkillBot\wwwroot\manifest\echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/wwwroot/manifest/echoskillbot-manifest-1.0.json)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/manifest/echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/manifest/echoskillbot-manifest-1.0.json)]

### <a name="python"></a>[Python](#tab/python)

**echo_skill_bot/wwwroot/manifest/echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/wwwroot/manifest/echoskillbot-manifest-1.0.json)]

---

"_スキル マニフェスト スキーマ_" は、スキル マニフェストのスキーマが記述された JSON ファイルです。 スキーマの現在のバージョンは [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) です。

## <a name="test-the-skill"></a>スキルのテスト

この時点で、スキルを通常のボットと同様にエミュレーターでテストできます。 ただし、スキルとしてテストするには、[スキル コンシューマーを実装する](skill-implement-consumer.md)必要があります。

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします

1. エコー スキル ボットをお使いのマシンでローカルに実行します。 手順については、README ファイルで [C#](https://aka.ms/skills-simple-bot-to-bot-csharp)、[JavaScript](https://aka.ms/skills-simple-bot-to-bot-js)、または [Python](https://aka.ms/skills-simple-bot-to-bot-python) のサンプルを参照してください。
1. 次に示すように、エミュレーターを使用してボットをテストします。 スキルに "end" または "stop" メッセージを送信すると、応答メッセージに加えて `endOfConversation` アクティビティが送信されることに注意してください。 スキルが終了したことが示す `endOfConversation` アクティビティがスキルから送信されます。

![エコー スキルのテスト](media/skills-simple-skill-test.png)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [スキル コンシューマーを実装する](skill-implement-consumer.md)
