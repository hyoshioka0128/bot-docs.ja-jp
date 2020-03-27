---
title: .NET v3 ボットをスキルに変換する
description: 既存の .NET v3 ボットをスキルに変換し、.NET v4 ボットからそれらを使用する方法について説明します。
keywords: .NET, ボットの移行, スキル, v3 ボット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2cf0398986203e131cf456344440cef3c2b90955
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117767"
---
# <a name="convert-a-net-v3-bot-to-a-skill"></a>.NET v3 ボットをスキルに変換する

この記事では、3 つのサンプル .NET v3 ボットをスキルに変換し、これらのスキルにアクセスできる v4 スキル コンシューマーを作成する方法について説明します。
JavaScript v3 ボットをスキルに変換するには、「[JavaScript v3 ボットをスキルに変換する](javascript-v3-as-skill.md)」を参照してください。
.NET ボットを v3 から v4 に移行するには、「[.NET v3 ボットを .NET Framework v4 ボットに移行する](conversion-framework.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

- Visual Studio 2019。
- .NET Core 3.1。
- .NET Framework 4.6.1 以降。
- Azure サブスクリプション。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。
- 変換する v3 .NET サンプル ボットのコピー: エコー ボット、[**PizzaBot**](https://aka.ms/v3-cs-pizza-bot)、および [**SimpleSandwichBot**](https://aka.ms/v3-cs-simple-sandwich-bot)。
- v4 .NET サンプル スキル コンシューマーのコピー: [**SimpleRootBot**](https://aka.ms/cs-simple-root-bot)。

## <a name="about-the-bots"></a>ボットについて

この記事で、各 v3 ボットは、スキルとして機能するように更新されます。 変換されたボットをスキルとしてテストできるように、v4 のスキル コンシューマーが含まれています。

- **EchoBot** は、受信したメッセージをエコー バックします。 これは、スキルとして、"end" (終了) または "stop" (停止) メッセージを受信したときに "_完了_" します。
  変換するボットは、v3 Bot Builder Echo Bot プロジェクト テンプレートに基づいています。
- **PizzaBot** は、ユーザーにピザの注文を段階的に説明します。 これは、スキルとして、完了時にユーザーの注文を親に送り返します。
- **SimpleSandwichBot** は、ユーザーにサンドイッチの注文を段階的に説明します。 これは、スキルとして、完了時にユーザーの注文を親に送り返します。

また、v4 スキル コンシューマー (**SimpleRootBot**) では、スキルを利用する方法を示し、それらをテストすることができます。

スキル コンシューマーを使用してスキルをテストするには、4 つのボットすべてを同時に実行する必要があります。 ボットは、Bot Framework Emulator を使用し、ボットごとに異なるローカル ポートを使って、ローカルでテストすることができます。

## <a name="create-azure-resources-for-the-bots"></a>ボット用の Azure リソースを作成する

ボット間認証を行うには、参加するボットのそれぞれに有効なアプリ ID とパスワードが必要です。

1. 必要に応じて、これらのボットのボット チャンネル登録を作成します。
1. それぞれのアプリ ID とパスワードを記録します。

## <a name="conversion-process"></a>変換処理

既存のボットをスキル ボットに変換するには、以降のセクションで説明するように、いくつかの手順を実行するだけです。 詳細については、「[スキルについて](../skills-conceptual.md)」を参照してください。

- ボットの `web.config` ファイルを更新して、ボットのアプリ ID とパスワードを設定し、"_許可される呼び出し元_" (AllowedCallers) プロパティを追加します。
- 要求検証を追加します。 これにより、ユーザーまたはルート ボットだけがスキルにアクセスできるように、スキルへのアクセスが制限されます。 既定およびカスタムの要求検証の詳細については、「[関連情報](#additional-information)」セクションを参照してください。
- ルート ボットからの `endOfConversation` アクティビティを処理するようにボットのメッセージ コントローラーを変更します。
- スキルが完了したときに `endOfConversation` アクティビティを返すようにボットのコードを変更します。
- スキルは、完了したときに会話状態がある場合やリソースを保持している場合は必ず、会話状態をクリアし、リソースを解放する必要があります。
- 必要に応じて、マニフェスト ファイルを追加します。
  スキル コンシューマーは必ずしもスキル コードにアクセスできるわけではないため、スキル マニフェストを使用して、スキルが受信および生成できるアクティビティ、スキルの入力パラメーターと出力パラメーター、およびスキルのエンドポイントを記述します。
  現在のマニフェスト スキーマは [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) です。

## <a name="convert-the-echo-bot"></a>エコー ボットを変換する

1. v3 **Bot Builder Echo Bot** プロジェクト テンプレートから新しいプロジェクトを作成し、ポート 3979 を使用するように設定します。

   1. プロジェクトを作成します。
   1. プロジェクトのプロパティを開きます。
   1. **[Web]** カテゴリを選択し、 **[Project Url]\(プロジェクト URL\)** を `http://localhost:3979/` に設定します。
   1. 変更を保存し、[プロパティ] タブを閉じます。

1. 構成ファイルに、エコー ボットのアプリ ID とパスワードを追加します。 また、アプリの設定で `EchoBotAllowedCallers` プロパティを追加し、単純なルート ボットのアプリ ID をその値に追加します。

   **V3EchoBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Web.config?range=11-16)]

1. カスタム要求検証コントロールと、サポートする認証構成クラスを追加します。

   **V3EchoBot\\Authentication\\CustomAllowedCallersClaimsValidator.cs**

   これは、カスタム要求検証を実装し、検証が失敗すると `UnauthorizedAccessException` をスローします。

   [!code-csharp[claims validator](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Authentication/CustomAllowedCallersClaimsValidator.cs?range=4-72&highlight=45,63,66)]

   **V3EchoBot\\Authentication\\CustomSkillAuthenticationConfiguration.cs**

   これは、許可される呼び出し元の情報を構成ファイルから読み込み、要求の検証に `CustomAllowedCallersClaimsValidator` を使用します。

   [!code-csharp[allowed callers](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Authentication/CustomSkillAuthenticationConfiguration.cs?range=4-20&highlight=12-14)]

1. `MessagesController` クラスを更新します。

   **V3EchoBot\\Controllers\\MessagesController.cs**

   using ステートメントを更新します。

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=4-15)]

   クラス属性を `BotAuthentication` から `SkillBotAuthentication` に更新します。 カスタム要求検証プロバイダーを使用するには、省略可能な `AuthenticationConfigurationProviderType` パラメーターを使用します。

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=19-21)]

   `HandleSystemMessage` メソッドで、`endOfConversation` メッセージを処理する条件を追加します。 これにより、スキル コンシューマーから会話が終了されると、スキルは状態をクリアして、リソースを解放することができます。

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=45-65)]

1. スキルが、ユーザーから "end" (終了) または "stop" (停止) メッセージを受信したときに、会話が完了しているというフラグを設定できるようにボットのコードを変更します。 また、スキルは、会話を終了するときに状態をクリアして、リソースを解放する必要があります。

   **V3EchoBot\\Dialogs\\RootDialog.cs**

   [!code-csharp[message received](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Dialogs/RootDialog.cs?range=21-41&highlight=5-13)]

1. このマニフェストをエコー ボットに使用します。 エンドポイントのアプリ ID をボットのアプリ ID に設定します。

   **V3EchoBot\\wwwroot\\echo-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/wwwroot/echo-bot-manifest.json?highlight=22)]

   スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="convert-the-pizza-bot"></a>ピザ ボットを変換する

1. PizzaBot プロジェクトのコピーを開き、ポート 3980 を使用するように設定します。

   1. プロジェクトのプロパティを開きます。
   1. **[Web]** カテゴリを選択し、 **[Project Url]\(プロジェクト URL\)** を `http://localhost:3980/` に設定します。
   1. 変更を保存し、[プロパティ] タブを閉じます。

1. 構成ファイルに、ピザ ボットのアプリ ID とパスワードを追加します。 また、アプリの設定で `AllowedCallers` プロパティを追加し、単純なルート ボットのアプリ ID をその値に追加します。

   **V3PizzaBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Web.config?range=11-16)]

1. 以下を行うためのヘルパー メソッドを持つ `ConversationHelper` クラスを追加します。
   - スキルが終了したときに `endOfConversation` アクティビティを送信する。 これにより、アクティビティの `Value` プロパティで注文情報が返され、会話が終了した理由を反映するように `Code` プロパティを設定できます。
   - 会話状態をクリアし、関連付けられているリソースをすべて解放します。

   **V3PizzaBot\\ConversationHelper.cs**

   [!code-csharp[conversation helper](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/ConversationHelper.cs?range=4-79)]

1. `MessagesController` クラスを更新します。

   **V3PizzaBot\\Controllers\\MessagesController.cs**

   using ステートメントを更新します。

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=4-16)]

   クラス属性を `BotAuthentication` から `SkillBotAuthentication` に更新します。 このボットは、既定の要求検証コントロールを使用します。

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=20-21)]

   `Post` メソッドで、ユーザーがスキル内から注文プロセスをキャンセルできるように `message` アクティビティ条件を変更します。 また、スキル コンシューマーから会話が終了されたときに、スキルが状態をクリアしてリソースを解放できるように、`endOfConversation` アクティビティ条件を追加します。

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=69-87)]

1. ボットのコードを変更します。

   **V3PizzaBot\\PizzaOrderDialog.cs**

   using ステートメントを更新します。

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=4-14)]

   ウェルカム メッセージを追加します。 これにより、ルート ボットがピザ ボットをスキルとして呼び出すときに、ユーザーはどのような処理が行われているか知ることができます。

   [!code-csharp[start form](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=29-35)]

   ユーザーが注文をキャンセルまたは完了したときに、会話が完了したというフラグをスキルが設定できるように、ボットのコードを変更します。 また、スキルは、会話を終了するときに状態をクリアして、リソースを解放する必要があります。

   [!code-csharp[form complete](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=76-104)]

1. このマニフェストをピザ ボットに使用します。 エンドポイントのアプリ ID をボットのアプリ ID に設定します。

   **V3PizzaBot\\wwwroot\\pizza-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/wwwroot/pizza-bot-manifest.json?highlight=22)]

   スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="convert-the-sandwich-bot"></a>サンドイッチ ボットを変換する

1. SimpleSandwichBot プロジェクトのコピーを開き、ポート 3981 を使用するように設定します。

   1. プロジェクトのプロパティを開きます。
   1. **[Web]** カテゴリを選択し、 **[Project Url]\(プロジェクト URL\)** を `http://localhost:3981/` に設定します。
   1. 変更を保存し、[プロパティ] タブを閉じます。

1. 構成ファイルに、ピザ ボットのアプリ ID とパスワードを追加します。 また、アプリの設定で `AllowedCallers` プロパティを追加し、単純なルート ボットのアプリ ID をその値に追加します。

   **V3SimpleSandwichBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Web.config?range=11-16)]

1. 以下を行うためのヘルパー メソッドを持つ `ConversationHelper` クラスを追加します。
   - スキルが終了したときに `endOfConversation` アクティビティを送信する。 これにより、アクティビティの `Value` プロパティで注文情報を返し、会話が終了した理由を反映するように `Code` プロパティを設定できます。
   - 会話状態をクリアし、関連付けられているリソースをすべて解放します。

   **V3SimpleSandwichBot\\ConversationHelper.cs**

   [!code-csharp[conversation helper](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/ConversationHelper.cs?range=4-79)]

1. `MessagesController` クラスを更新します。

   **V3SimpleSandwichBot\\Controllers\\MessagesController.cs**

   using ステートメントを更新します。

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=4-17)]

   クラス属性を `BotAuthentication` から `SkillBotAuthentication` に更新します。 このボットは、既定の要求検証コントロールを使用します。

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=21-22)]

   `Post` メソッドで、ユーザーがスキル内から注文プロセスをキャンセルできるように `message` アクティビティ条件を変更します。 また、スキル コンシューマーから会話が終了されたときに、スキルが状態をクリアしてリソースを解放できるように、`endOfConversation` アクティビティ条件を追加します。

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=42-60)]

1. サンドイッチ フォームを変更します。

   **V3SimpleSandwichBot\\Sandwich.cs**

   using ステートメントを更新します。

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Sandwich.cs?range=4-8)]

   会話が完了したというフラグをスキルが設定できるように、フォームの `BuildForm` メソッドを変更します。

   [!code-csharp[form complete](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Sandwich.cs?range=47-54&highlight=6)]

1. このマニフェストをサンドイッチ ボットに使用します。 エンドポイントのアプリ ID をボットのアプリ ID に設定します。

   **V3SimpleSandwichBot\\wwwroot\\sandwich-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/wwwroot/sandwich-bot-manifest.json?highlight=22)]

   スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="create-the-v4-root-bot"></a>v4 ルート ボットを作成する

単純なルート ボットでは 3 つのスキルが使用されます。ユーザーはこれを使用して、変換手順が計画どおりに機能したことを確認できます。 このボットは、ポート 3978 でローカルに実行されます。

1. 構成ファイルに、ルート ボットのアプリ ID とパスワードを追加します。 v3 のスキルごとに、スキルのアプリ ID を追加します。

   **V4SimpleRootBot\\appsettings.json**

   [!code-json[configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V4SimpleRootBot/appsettings.json?highlight=2-3,8,13,18)]

## <a name="test-the-root-bot"></a>ルート ボットのテスト

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします。

1. 4 つのボットすべてを自分のマシン上でローカルにビルドして実行します。
1. エミュレーターを使用してルート ボットを接続します。
1. スキルとスキル コンシューマーをテストします。

## <a name="additional-information"></a>関連情報

### <a name="bot-to-bot-authentication"></a>ボット間認証

ルートとスキルは HTTP を介して通信します。 このフレームワークでは、ベアラー トークンとボット アプリケーション ID を使用して各ボットの ID が検証されます。 これは、認証構成オブジェクトを使用して、受信した要求の認証ヘッダーを検証します。 認証構成に要求検証コントロールを追加できます。 要求は認証ヘッダーの後に評価されます。 要求を拒否する場合は、検証コードからエラーまたは例外をスローする必要があります。

既定の要求検証コントロールは、ボットの構成ファイルから `AllowedCallers` アプリケーション設定を読み取ります。 この設定には、スキルの呼び出しが許可されているボットのアプリケーション ID のコンマ区切りリスト、またはすべてのボットにスキルの呼び出しを許可する "*" が含まれている必要があります。

カスタム要求検証コントロールを実装するには、`AuthenticationConfiguration` と `ClaimsValidator` から派生するクラスを実装し、`SkillBotAuthentication` 属性で派生した認証構成を参照します。 「[エコー ボットを変換する](#convert-the-echo-bot)」セクションの手順 3 と 4 に、要求検証クラスの例があります。
