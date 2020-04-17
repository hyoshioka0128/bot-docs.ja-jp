---
title: Javascript v3 ボットをスキルに変換する
description: 既存の JavaScript v3 ボットをスキルに変換し、JavaScript v4 ボットからそれらを使用する方法について説明します。
keywords: JavaScript, ボットの移行, スキル, v3 ボット
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6b29a60d757725e65ae716301e736a66d508332a
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80117807"
---
# <a name="convert-a-javascript-v3-bot-to-a-skill"></a>Javascript v3 ボットをスキルに変換する

この記事では、2 つのサンプル JavaScript v3 ボットをスキルに変換し、これらのスキルにアクセスできる v4 スキル コンシューマーを作成する方法について説明します。
.NET v3 ボットをスキルに変換するには、「[.NET v3 ボットをスキルに変換する](net-v3-as-skill.md)」を参照してください。
JavaScript ボットを v3 から v4 に移行するには、「[JavaScript v3 ボットを v4 ボットに移行する](conversion-javascript.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

- Visual Studio Code。
- Node.js.
- Azure サブスクリプション。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。
- スキルをテストするには、Bot Framework Emulator とボットのローカル コピーが必要です。
  - V3 JavaScript エコー スキル: [**Skills/v3-skill-bot**](https://aka.ms/v3-js-echo-skill)
  - v3 JavaScript 予約スキル: [**Skills/v3-booking-bot-skill**](https://aka.ms/v3-js-booking-skill)
  - v4 JavaScript サンプル スキル コンシューマー: [**Skills/v4-root-bot**](https://aka.ms/js-simple-root-bot)

## <a name="about-the-bots"></a>ボットについて

この記事では、v3 ボットをそれぞれスキルとして機能するように作成します。 変換されたボットをスキルとしてテストできるように、v4 のスキル コンシューマーが含まれています。

- **v3-skill-bot** は、受信したメッセージをエコー バックします。 これは、スキルとして、"end" (終了) または "stop" (停止) メッセージを受信したときに "_完了_" します。 変換するボットは、最小 v3 ボットに基づいています。
- **v3-booking-bot-skill** を使用すると、ユーザーはフライトやホテルを予約できます。 これは、スキルとして、収集された情報を、完了時に親に送り返します。

また、v4 スキル コンシューマー (**v4-root-bot**) では、スキルを利用する方法を示し、それらをテストすることができます。

スキル コンシューマーを使用してスキルをテストするには、3 つのボットをすべて同時に実行する必要があります。 ボットは、Bot Framework Emulator を使用し、ボットごとに異なるローカル ポートを使ってローカルでテストできます。

## <a name="create-azure-resources-for-the-bots"></a>ボット用の Azure リソースを作成する

ボット間認証を行うには、参加するボットのそれぞれに有効なアプリ ID とパスワードが必要です。

1. 必要に応じて、これらのボットのボットチャンネル登録を作成します。
1. それぞれのアプリ ID とパスワードを記録します。

## <a name="conversion-process"></a>変換処理

既存のボットをスキル ボットに変換するには、以降のセクションで説明するように、いくつかの手順を実行するだけです。 詳細については、「[スキルについて](../skills-conceptual.md)」を参照してください。

- ボットの `.env` ファイルを更新して、ボットのアプリ ID とパスワードを設定し、"_ルート ボットのアプリ ID_" プロパティを追加します。
- 要求検証を追加します。 これにより、ユーザーまたはルート ボットだけがスキルにアクセスできるように、スキルへのアクセスが制限されます。 既定およびカスタムの要求検証の詳細については、「[関連情報](#additional-information)」セクションを参照してください。
- ルート ボットからの `endOfConversation` アクティビティを処理するようにボットのメッセージ コントローラーを変更します。
- スキルが完了したときに `endOfConversation` アクティビティを返すようにボットのコードを変更します。
- スキルは、完了したときに会話状態がある場合やリソースを保持している場合は必ず、会話状態をクリアし、リソースを解放する必要があります。
- 必要に応じて、マニフェスト ファイルを追加します。
  スキル コンシューマーは必ずしもスキル コードにアクセスできるわけではないため、スキル マニフェストを使用して、スキルが受信および生成できるアクティビティ、スキルの入力パラメーターと出力パラメーター、およびスキルのエンドポイントを記述します。
  現在のマニフェスト スキーマは [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) です。

## <a name="convert-the-echo-bot"></a>エコー ボットを変換する

基本のスキルに変換された v3 エコー ボットの例については、[Skills/v3-skill-bot](https://aka.ms/v3-js-echo-skill) に関するページを参照してください。

1. 単純な JavaScript v3 ボット プロジェクトを作成し、必要なモジュールをインポートします。

   **v3-skill-bot/app.js**

   [!code-javascript[require statements](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=5-7)]

1. ポート 3979 でローカルに実行されるようにボットを設定します。

   **v3-skill-bot/app.js**

   [!code-javascript[Setup server and port](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=9-13)]

1. 構成ファイルに、エコー ボットのアプリ ID とパスワードを追加します。 さらに、単純なルート ボットのアプリ ID を値として持つ `ROOT_BOT_APP_ID` プロパティを追加します。

   **v3-skill-bot/.env**

   [!code[.env file](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/.env)]

1. ボット用のチャット コネクタを作成します。 これは、既定の認証構成を使用します。 `enableSkills` を `true` に設定して、ボットをスキルとして使用できるようにします。 `allowedCallers` は、このスキルを使用することが許可されているボットのアプリ ID の配列です。 この配列の最初の値が "*" の場合は、すべてのボットがこのスキルを使用できます。

   **v3-skill-bot/app.js**

   [!code-javascript[chat connector](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=24-30&highlight=5-6)]

1. ユーザーがスキルを終了することを選択したときに `endOfConversation` アクティビティを送信するようにメッセージ ハンドラーを更新します。

   **v3-skill-bot/app.js**

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=35-46&highlight=6)]

1. このボットは会話状態を維持せず、会話のためのリソースを作成しないため、ボットは、スキル コンシューマーから受け取る `endOfConversation` アクティビティを処理する必要はありません。

1. このマニフェストをエコー ボットに使用します。 エンドポイントのアプリ ID をボットのアプリ ID に設定します。

   **v3-skill-bot/manifest/v3-skill-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/manifest/v3-skill-bot-manifest.json?highlight=22)]

   スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="convert-the-booking-bot"></a>予約ボットを変換する

基本のスキルに変換された v3 予約ボットの例については、[Skills/v3-booking-bot-skill](https://aka.ms/v3-js-booking-skill) に関するページを参照してください。
変換前のボットは、v3 [core-MultiDialogs](https://aka.ms/v3-js-core-multidialogs) サンプルに似ていました。

1. 必要なモジュールをインポートします。

   **v3-booking-bot-skill/app.js**

   [!code-javascript[require statements](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=2-7)]

1. ポート 3980 でローカルに実行するようにボットを設定します。

   **v3-booking-bot-skill/app.js**

   [!code-javascript[Setup server and port](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=9-13)]

1. 構成ファイルに、予約ボットのアプリ ID とパスワードを追加します。 さらに、単純なルート ボットのアプリ ID を値として持つ `ROOT_BOT_APP_ID` プロパティを追加します。

   **v3-booking-bot-skill/.env**

   [!code[.env file](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/.env)]

1. ボット用のチャット コネクタを作成します。 これは、カスタム認証構成を使用します。 `enableSkills` を `true` に設定して、ボットをスキルとして使用できるようにします。 `authConfiguration` には、認証と要求検証に使用するカスタム認証構成オブジェクトが含まれています。

   **v3-booking-bot-skill/app.js**

   [!code-javascript[chat connector](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=18-24&highlight=5-6)]

   **v3-booking-bot-skill/allowedCallersClaimsValidator.js**

   これは、カスタム要求検証を実装し、検証が失敗するとエラーをスローします。

   [!code-javascript[custom claims validation](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/allowedCallersClaimsValidator.js?range=4-47&highlight=22,25,39,41)]

1. スキルが終了したときに `endOfConversation` アクティビティを送信するようにメッセージ ハンドラーを更新します。 `session.endConversation()` は `endOfConversation` アクティビティを送信することに加え、会話状態をクリアすることに注意してください。

   **v3-booking-bot-skill/app.js**

   `endOfConversation` アクティビティの `code` と `value` プロパティを設定し、会話状態をクリアするヘルパー関数を実装します。
   ボットが会話の他のリソースを管理していた場合は、それらもここで解放します。

   [!code-javascript[endConversation function](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=122-134&highlight=12)]

   ユーザーがプロセスを完了したら、ヘルパー メソッドを使用してスキルを終了し、ユーザーが収集したデータを返します。

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=39-40)]

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=72-77&highlight=4)]

   ユーザーが代わりにプロセスを早期に終了した場合でも、ヘルパー メソッドは呼び出されます。

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=86-101&highlight=9-10)]

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=107-108)]

1. スキルがスキル コンシューマーによって取り消された場合、コンシューマーは `endOfConversation` アクティビティを送信します。 このアクティビティを処理し、会話に関連付けられているすべてのリソースを解放します。

   **v3-booking-bot-skill/app.js**

   [!code-javascript[endConversation function](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=110-115)]

1. このマニフェストを予約ボットに使用します。 エンドポイントのアプリ ID をボットのアプリ ID に設定します。

   **v3-booking-bot-skill/manifest/v3-booking-bot-skill-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/manifest/v3-booking-bot-skill-manifest.json?highlight=22)]

   スキル マニフェスト スキーマについては、[skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json) を参照してください。

## <a name="create-the-v4-root-bot"></a>v4 ルート ボットを作成する

単純なルート ボットでは 2 つのスキルが使用されます。ユーザーはこれを使用して、変換手順が計画どおりに機能したことを確認できます。 このボットは、ポート 3978 でローカルに実行されます。

1. 構成ファイルに、ルート ボットのアプリ ID とパスワードを追加します。 v3 のスキルごとに、スキルのアプリ ID を追加します。

   **v4-root-bot/.env**

   [!code-json[configuration](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v4-root-bot/.env?highlight=2-3,6,10)]

## <a name="test-the-root-bot"></a>ルート ボットのテスト

最新の [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) をダウンロードしてインストールします。

1. 3 つのすべてのボットをマシン上でローカルに起動します。
1. エミュレーターを使用してルート ボットを接続します。
1. スキルとスキル コンシューマーをテストします。

## <a name="additional-information"></a>関連情報

### <a name="bot-to-bot-authentication"></a>ボット間認証

ルートとスキルは HTTP を介して通信します。 このフレームワークでは、ベアラー トークンとボット アプリケーション ID を使用して各ボットの ID が検証されます。 これは、認証構成オブジェクトを使用して、受信した要求の認証ヘッダーを検証します。 認証構成に要求検証コントロールを追加できます。 要求は認証ヘッダーの後に評価されます。 要求を拒否する場合は、検証コードからエラーまたは例外をスローする必要があります。

チャット コネクタを作成するときに、設定パラメーターに `allowedCallers` または `authConfiguration` のいずれかのプロパティを含めて、ボット間認証を有効にします。

チャット コネクタの既定の要求検証コントロールでは、`allowedCallers` プロパティが使用されます。 その値は、スキルの呼び出しが許可されているボットのアプリケーション ID の配列である必要があります。 すべてのボットがスキルを呼び出せるようにするには、最初の要素を "*" に設定します。

カスタム要求検証関数を使用するには、`authConfiguration` フィールドを検証関数に設定します。 この関数は、要求オブジェクトの配列を受け取り、検証が失敗するとエラーをスローします。 「[予約ボットを変換する](#convert-the-booking-bot)」セクションの手順 4 に、要求検証コントロールの例があります。
