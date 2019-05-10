---
ms.openlocfilehash: 8e5677fe59dd9edad6ac1da9d029e5f7c08bf179
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563965"
---
## <a name="next-steps"></a>次の手順
ご利用になるボッドをクラウドにデプロイし、Bot Framework Emulator を使用したボットのテストによってデプロイが正常に行われたことを確認したら、ボット発行プロセスでの次の手順は、ご利用になるボッドを Bot Framework に既に登録しているかどうかによって異なります。

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>ご利用のボッドを Bot Framework に既に登録している場合:

1. <a href="https://dev.botframework.com" target="_blank">Bot Framework Portal</a> に戻り、[ボットの [設定] データを更新](~/bot-service-manage-settings.md)して、ボット用の **[Messaging endpoint]\(メッセージング エンドポイント\)** を指定します。

2. [1 つまたは複数のチャネルで実行するボットを構成する](~/bot-service-manage-channels.md)

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>ご利用のボッドを Bot Framework にまだ登録していない場合:

1. [ご利用のボットを Bot Framework に登録する](~/bot-service-quickstart-registration.md)

2. デプロイされたアプリケーションの構成設定内で Microsoft アプリ ID と Microsoft アプリ パスワードの値を更新して、登録プロセス中に自動的に生成された **appID** 値と **password** 値を指定します。 ボットの **AppID** と **AppPassword** を見つけるには、[MicrosoftAppID と MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) を確認します。

3. [1 つまたは複数のチャネルで実行するボットを構成する](~/bot-service-manage-channels.md)