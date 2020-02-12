---
title: スキル コンシューマーを実装する | Microsoft Docs
description: Bot Framework SDK を使用してスキル コンシューマーを実装する方法について説明します。
keywords: スキル
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/22/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5dc2dc7327f739c8aee5fecd799aa562a6a98035
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2020
ms.locfileid: "76753736"
---
# <a name="implement-a-skill-consumer"></a>スキル コンシューマーを実装する

[!INCLUDE[applies-to](../includes/applies-to.md)]

スキルを使用して別のボットを拡張することができます。
"_スキル_" は別のボットに対して一連のタスクを実行できるボットであり、マニフェストを使用してそのインターフェイスが記述されます。
"_ルート ボット_" は、1 つ以上のスキルを呼び出すことができるユーザー向けのボットです。 ルート ボットは "_スキル コンシューマー_" の一種です。

- スキル コンシューマーにアクセスできるスキルまたはユーザーは、要求検証を使用して管理できます。
- スキル コンシューマーは複数のスキルを使用できます。
- スキルのソース コードにアクセスできない開発者は、スキル マニフェストの情報を使用してスキル コンシューマーを設計できます。

この記事では、エコー スキルを使用してユーザーの入力をエコーするスキル コンシューマーを実装する方法について説明します。 スキル マニフェストのサンプルおよびエコー スキルの実装に関する情報については、「[スキルを実装する](skill-implement-skill.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

- [ボットの基本](bot-builder-basics.md)、[スキル ボットのしくみ](skills-conceptual.md)、および[スキルの実装方法](skill-implement-skill.md)に関する知識。
- Azure サブスクリプション。 お持ちでない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。
- **skills simple bot-to-bot** サンプルのコピー ([**C#** ](https://aka.ms/skills-simple-bot-to-bot-csharp)、[**JavaScript**](https://aka.ms/skills-simple-bot-to-bot-js)、または [**Python**](https://aka.ms/skills-simple-bot-to-bot-python))。

## <a name="about-this-sample"></a>このサンプルについて

**skills simple bot-to-bot** サンプルには、次の 2 つのボットのプロジェクトが含まれています。

- スキルを実装する "_エコー スキル ボット_"。
- スキルを使用するルート ボットを実装する "_単純なルート ボット_"。

この記事で取り上げるルート ボットには、ボット オブジェクトとアダプター オブジェクトにサポート ロジックが組み込まれ、かつアクティビティをスキルとやり取りするためのオブジェクトが含まれています。 チェックの内容は次のとおりです

- スキル クライアント。スキルにアクティビティを送信するために使用されます。
- スキル ハンドラー。スキルからアクティビティを受信するために使用されます。
- スキル会話 ID ファクトリ。ユーザー/ルート間の会話リファレンスの参照とルート/スキル間の会話リファレンスの間で変換を行うために、スキル クライアントとスキル ハンドラーで使用されます。

### <a name="ctabcs"></a>[C#](#tab/cs)

![スキル コンシューマーのクラス ダイアグラム](./media/skills-simple-root-cs.png)

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![スキル コンシューマーのクラス ダイアグラム](./media/skills-simple-root-js.png)

### <a name="pythontabpython"></a>[Python](#tab/python)

![スキル コンシューマーのクラス ダイアグラム](./media/skills-simple-root-python-2.png)

---

エコー スキル ボットについては、「[スキルを実装する](skill-implement-skill.md)」を参照してください。

## <a name="resources"></a>リソース

ボット間認証を行うには、参加するボットのそれぞれに有効なアプリ ID とパスワードが必要です。

スキルとスキル コンシューマーの両方を Azure に登録します。 Bot Channels Registration を使用できます。 詳細については、「[ボットを Azure Bot Service に登録する](../bot-service-quickstart-registration.md)」を参照してください。

## <a name="application-configuration"></a>アプリケーションの構成

1. ルート ボットのアプリ ID とパスワードを追加します。
1. スキルがスキル コンシューマーに返信するときの宛先となるエンドポイント URL を追加します。
1. スキル コンシューマーが使用するスキルごとにエントリを追加します。 各エントリには次のものが含まれます。
   - スキル コンシューマーが各スキルを識別するために使用する ID。
   - スキルのアプリ ID。
   - スキルのメッセージング エンドポイント。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\appsettings.json**

ルート ボットのアプリ ID とパスワードを appsettings.json ファイルに追加します。 また、エコー スキル ボットのアプリ ID を `BotFrameworkSkills` 配列に追加します。

[!code-csharp[configuration file](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/appsettings.json)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**echo-skill-bot/.env**

ルート ボットのアプリ ID とパスワードを .env ファイルに追加します。 また、エコー スキル ボットのアプリ ID を追加します。

[!code-javascript[configuration file](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/.env)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple_root_bot/config.py**

ルート ボットのアプリ ID とパスワードを .env ファイルに追加します。 また、エコー スキル ボットのアプリ ID を追加します。

[!code-python[configuration file](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/config.py?range=16-29)]

---

## <a name="skills-configuration"></a>スキルの構成

このサンプルでは、構成ファイル内の各スキルの情報を、"_スキル_" オブジェクトのコレクションに読み込みます。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\SkillsConfiguration.cs**

[!code-csharp[skills configuration](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/SkillsConfiguration.cs?range=14-38)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/skillsConfiguration.js**

[!code-javascript[skills configuration](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/skillsConfiguration.js?range=7-33)]


### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/config.py**

[!code-python[skills configuration](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/config.py?range=32-36)]

---

## <a name="conversation-id-factory"></a>会話 ID ファクトリ

会話 ID ファクトリは、スキルで使用する会話 ID を作成し、スキルの会話 ID から元のユーザー会話 ID を復旧できるようにします。

このサンプルの会話 ID ファクトリは、次に示す単純なシナリオをサポートしています。

- 1 つの特定のスキルを使用するようにルート ボットが設計されている。
- ルート ボットとスキルの間で一度にアクティブになる会話は 1 つだけである。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\SkillConversationIdFactory.cs**

[!code-csharp[Conversation ID factory](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/SkillConversationIdFactory.cs?range=17-40)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/skillConversationIdFactory.js**

[!code-javascript[Conversation ID factory](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/skillConversationIdFactory.js?range=10-29)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/skill_conversation_id_factory.py**

[!code-python[Conversation ID factory](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/skill_conversation_id_factory.py?range=9-47)]

---

より複雑なシナリオをサポートするには、次のように会話 ID ファクトリを設計します。

- _create skill conversation ID_ メソッドで、適切なスキル会話 ID を取得または生成します。
- _get conversation reference_ メソッドで、正しいユーザー会話を取得します。

## <a name="skill-client-and-skill-handler"></a>スキル クライアントとスキル ハンドラー

スキル コンシューマーは、スキル クライアントを使用してアクティビティをスキルに転送します。
クライアントは、このためにスキルの構成情報と会話 ID ファクトリを使用します。

スキル コンシューマーは、スキル ハンドラーを使用してスキルからアクティビティを受信します。
ハンドラーは、このために会話 ID ファクトリ、認証構成、および資格情報プロバイダーを使用するほか、ルート ボットのアダプターとアクティビティ ハンドラーへの依存関係を持ちます。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Startup.cs**

[!code-csharp[skill client and handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Startup.cs?range=42-43)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**

[!code-javascript[skill client](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=107-108)]

[!code-javascript[skill handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=132)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[skill client](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=58)]

[!code-python[skill handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=120-122)]

---

スキルからの HTTP トラフィックは、スキル コンシューマーがスキルにアドバタイズするサービス URL エンドポイントに送られます。 トラフィックをスキル ハンドラーに転送するには、言語固有のエンドポイント ハンドラーを使用します。

既定のスキル ハンドラーは次のことを行います。

- 認証構成オブジェクトを使用して、ボット間認証と要求検証の両方を実行します。
- 会話 ID ファクトリを使用して、コンシューマー/スキル間の会話を、元のルート/ユーザー間の会話に変換します。
- プロアクティブ メッセージを生成して、スキル コンシューマーがルート/ユーザー間のターン コンテキストを再確立し、アクティビティをユーザーに転送できるようにします。

## <a name="activity-handler-logic"></a>アクティビティ ハンドラーのロジック

注目すべき点として、スキル コンシューマーのロジックでは次の処理を行う必要があります。

- アクティブなスキルの有無を記憶し、必要に応じてそれらのスキルにアクティビティに転送します。
- スキルへの転送が必要な要求をユーザーが行ったことを確認し、スキルを開始します。
- アクティブなスキルからの `endOfConversation` アクティビティを探し、その完了を確認します。
- 必要に応じて、未完了のスキルをユーザーまたはスキル コンシューマーがキャンセルできるようにするロジックを追加します。
- 応答がスキル コンシューマーの別のインスタンスに返される可能性があるため、スキルを呼び出す前に状態を保存します。 (負荷分散など)

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Bots\RootBot.cs**

ルート ボットには、会話状態、スキル情報、スキル クライアント、および全般的な構成への依存関係があります。 これらのオブジェクトは、ASP.NET から依存関係の挿入を通じて提供されます。
またルート ボットでは、アクティブなスキルを追跡するための会話状態プロパティ アクセサーが定義されます。

[!code-csharp[Root bot dependencies](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=21-55)]

次のサンプルでは、アクティビティをスキルに転送するためのヘルパー メソッドが使用されています。 このメソッドは、スキルを呼び出す前に会話状態を保存し、HTTP 要求が成功したかどうかを確認します。

[!code-csharp[Send to skill](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=125-139)]

ルート ボットには、ユーザーからのメッセージとスキルからの `endOfConversation` アクティビティを処理するロジックが組み込まれています。

[!code-csharp[message/end-of-conversation handlers](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=57-112)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/rootBot.js**

ルート ボットには、会話状態、スキル情報、およびスキル クライアントへの依存関係があります。
またルート ボットでは、アクティブなスキルを追跡するための会話状態プロパティ アクセサーが定義されます。

[!code-javascript[Root bot dependencies](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=7-30)]

次のサンプルでは、アクティビティをスキルに転送するためのヘルパー メソッドが使用されています。 このメソッドは、スキルを呼び出す前に会話状態を保存し、HTTP 要求が成功したかどうかを確認します。

[!code-javascript[Send to skill](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=108-120)]

ルート ボットには、ユーザーからのメッセージとスキルからの `endOfConversation` アクティビティを処理するロジックが組み込まれています。

[!code-javascript[message/end-of-conversation handlers](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=33-85)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/bots/root_bot.py**

ルート ボットには、会話状態、スキル情報、スキル クライアント、および全般的な構成への依存関係があります。
またルート ボットでは、アクティブなスキルを追跡するための会話状態プロパティ アクセサーが定義されます。

[!code-python[Root bot dependencies](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=23-37)]

次のサンプルでは、アクティビティをスキルに転送するためのヘルパー メソッドが使用されています。 このメソッドは、スキルを呼び出す前に会話状態を保存し、HTTP 要求が成功したかどうかを確認します。

[!code-python[Send to skill](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=104-117)]

ルート ボットには、ユーザーからのメッセージとスキルからの `endOfConversation` アクティビティを処理するロジックが組み込まれています。

[!code-python[Handled activities](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=39-93)]

---


## <a name="on-turn-error-handler"></a>オン ターン エラー ハンドラー

エラーが発生すると、アダプターは会話状態をクリアしてユーザーとの会話をリセットし、エラー状態を解消します。

スキル コンシューマーで会話状態をクリアする前に、アクティブなスキルに "_会話終了_" アクティビティを送信することをお勧めします。 これにより、スキル コンシューマーが会話を解放する前に、コンシューマ/スキル間の会話に関連するリソースをスキルが解放できるようになります。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\AdapterWithErrorHandler.cs**

このサンプルでは、ターン エラー ロジックがいくつかのヘルパー メソッドに分けられています。

[!code-csharp[On turn error](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/AdapterWithErrorHandler.cs?range=40-120)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**


[!code-javascript[On turn error](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=34-87)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**app.py**

[!code-python[On turn error](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=62-115)]

---

## <a name="skills-endpoint"></a>スキル エンドポイント

ボットでは、受信したスキル アクティビティをルート ボットのスキル ハンドラーに転送するエンドポイントが定義されます。


### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Controllers\SkillController.cs**

[!code-csharp[skill endpoint](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Controllers/SkillController.cs?range=15-23)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**


[!code-javascript[skill endpoint](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=133-134)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[skill endpoint](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=144)]

---

## <a name="service-registration"></a>サービス登録

要求検証を含む認証構成オブジェクトを、すべての追加オブジェクトと共に組み込みます。

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Startup.cs**

[!code-csharp[services](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Startup.cs?range=22-53)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**

[!code-javascript[services](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=27-31)]

[!code-javascript[services](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=95-134)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[services](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=35-58)]

[!code-python[services](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=118-144)]

---

## <a name="test-the-root-bot"></a>ルート ボットのテスト

スキル コンシューマーは通常のボットと同様にエミュレーターでテストできますが、スキルとスキル コンシューマーの両方のボットを同時に実行する必要があります。
スキルを構成する方法については、「[スキルを実装する](skill-implement-skill.md)」を参照してください。

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします

1. エコー スキル ボットと単純なルート ボットをお使いのマシンでローカルに実行します。 手順については、README ファイルで [C#](https://aka.ms/skills-simple-bot-to-bot-csharp)、[JavaScript](https://aka.ms/skills-simple-bot-to-bot-js)、または [Python](https://aka.ms/skills-simple-bot-to-bot-python) のサンプルを参照してください。
1. 次に示すように、エミュレーターを使用してボットをテストします。 スキルに `end` または `stop` メッセージを送信すると、スキルからルート ボットに対し、応答メッセージに加えて `endOfConversation` アクティビティが送信されることに注意してください。 `endOfConversation` アクティビティの _code_ プロパティにより、スキルが正常に完了したことが示されます。

![スキル コンシューマーのテスト](media/skills-simple-consumer-test.png)

## <a name="additional-information"></a>関連情報

より複雑なルート ボットを実装する場合に検討すべき事項を次に示します。

### <a name="to-allow-the-user-to-cancel-a-multi-step-skill"></a>ユーザーが複数ステップのスキルをキャンセルできるようにする

ルート ボットでは、ユーザーのメッセージをアクティブなスキルに転送する前に確認する必要があります。 ユーザーが現在のプロセスのキャンセルを求めた場合、ルート ボットはメッセージを転送する代わりに `endOfConversation` アクティビティをスキルに送信できます。

### <a name="to-exchange-data-between-the-root-and-skill-bots"></a>ルート ボットとスキル ボットの間でデータをやり取りする

スキル コンシューマーでは、スキルにパラメーターを送信するために、スキルに送信するメッセージに _value_ プロパティを設定できます。 スキルから戻り値を受信するために、スキル コンシューマーでは、スキルから `endOfConversation` アクティビティが送信されたときに _value_ プロパティを確認する必要があります。

### <a name="to-use-multiple-skills"></a>複数のスキルを使用する

- アクティブなスキルがある場合、ルート ボットはそのスキルを特定し、ユーザーのメッセージを適切なスキルに転送する必要があります。
- アクティブなスキルがない場合、ルート ボットは、ボットの状態とユーザーの入力を基に、開始するスキル (存在する場合) を特定する必要があります。
- ユーザーが複数の同時スキル間を切り替えられるようにする場合は、ルート ボットで、ユーザーのメッセージを転送する前にユーザーが対話しようとしているアクティブなスキルを特定する必要があります。

<!--
## Next steps

TBD: Claims validation? Skill manifest?

> [!div class="nextstepaction"]
> [Add claims validation](skill-add-claims-validation.md)
-->
