---
title: Azure にボットを移行する | Microsoft Docs
description: Azure Portal で旧 Bot Framework Portal からボット サービスにボットを移行する方法について説明します。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 3/22/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c4b058ecc4f6a01d51d3aa7abe8f4f926df4e771
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491358"
---
# <a name="migrate-your-bot-to-azure"></a>ボットを Azure に移行する

[Bot Framework Portal](http://dev.botframework.com) で作成された **Azure Bot Service (プレビュー)** ボットはすべて、Azure の新しい Bot Service に移行する必要があります。 このサービスは 2017 年 12 月に一般公開 (GA) となりました。 

以下にのみ接続されている登録ボットは移行する "*必要がない*" ことにご注意ください: **Teams**、**Skype**、**Cortana**。 たとえば、**Facebook** と **Skype** に接続されている登録ボットは移行する*必要があります*が、**Skype** と **Cortana** に接続されている登録ボットは移行する*必要はありません*。

> [!IMPORTANT]
> Node.js で作成された Functions ボットを移行する前に、**Azure Functions Pack** を利用して **node_modules** モジュールをまとめてパッケージ化する必要があります。 パッケージ化により、移行中のパフォーマンスがよくなり、移行後も Functions ボットがより効率的に実行されます。 モジュールをパッケージ化する方法については、「[Package a Functions bot with Funcpack](#package-a-functions-bot-with-funcpack)」(Funcpack で Functions ボットをパッケージ化する) を参照してください。

ボットは次の手順で移行します。

1. [Bot Framework Portal](http://dev.botframework.com) にサインインし、 **[マイ ボット]** をクリックします。
2. 移行するボットの **[移行]** ボタンをクリックします。
3. **[規約]** を受諾し、 **[移行]** をクリックして移行プロセスを開始するか、 **[キャンセル]** をクリックしてこのアクションを取り消します。

> [!IMPORTANT]
> 移行タスクの進行中は、ページから離れたり、ページを更新したりしないでください。 それを行うと移行タスクが不完全な状態で停止するので、もう一度実行する必要があります。 確認メッセージが表示されるまで待ち、表示されたら移行は正常完了となります。

移行プロセスが正常に完了すると、 **[移行ステータス]** にボットが移行されていることが示されます。問題が発生したときのために、 **[Roll back migration]\(移行のロールバック\)** ボタンが一週間表示されます。

移行したボットの名前をクリックすると、[Azure Portal](https://portal.azure.com) でそのボットが開きます。

## <a name="package-a-functions-bot-with-funcpack"></a>Funcpack で Functions ボットをパッケージ化する

Node.js で作成された Functions ボットは移行前に [Funcpack](https://github.com/Azure/azure-functions-pack) でパッケージ化する必要があります。 プロジェクトは Funcpack でパッケージ化する手順は次のようになります。

1.  コードをローカルにダウンロードしていない場合、[ダウンロード](bot-service-build-download-source-code.md)します。
2.  **packages.json** の npm パッケージをすべて最新版に更新し、`npm install` を実行します。
3.  **messages/index.js** を開き、`module.exports = { default: connector.listen() }` を `module.exports = connector.listen();` に変更します。
4.  npm で `npm install -g azure-functions-pack` を実行し、Funcpack をインストールします。
5.  **node_modules** ディレクトリをパッケージ化するには、`funcpack pack ./` を実行します。
6.  Bot Framework Emulator を使用して Functions ボットを実行することでボットをローカルでテストします。 *funcpack* ボットの実行方法の詳細は[こちら](https://github.com/Azure/azure-functions-pack#how-to-run)にあります。 
7.  コードを Azure にアップロードします。 必ず `.funcpack` ディレクトリをアップロードします。 **node_modules** ディレクトリをアップロードする必要はありません。
8. リモート ボットをテストし、予想どおり応答することを確認します。
9. 上記の手順でボットを移行します。

## <a name="migration-under-the-hood"></a>移行のしくみ

移行するボットの種類によっては、移行のしくみを理解する上で下の一覧が役立つことがあります。

* **Web アプリ ボット**または **Functions ボット**:この種類のボットの場合、古いボットから新しいボットにソース コード プロジェクトがコピーされます。 ボットのストレージ、Application Insights、LUIS など、その他のリソースはそのまま残ります。 そのような場合、新しいボットには、既存リソースの ID/キー/パスワードのコピーが含まれます。 
* **ボット チャネル登録**: この種類のボットの場合、移行プロセスによって新しい**ボット チャネル登録**が作成され、古いボットからエンドポイントがコピーされます。 
* 移行するボットの種類に関係なく、移行プロセスによって既存のボットの状態が変更されることはありません。 そのため、必要であれば、安全にロールバックできます。
* [継続的デプロイ](bot-service-build-continuous-deployment.md)を設定している場合、ソース コントロールが新しいボットに接続されるように、もう一度設定する必要があります。

## <a name="understanding-azure-resources-after-migration"></a>移行後の Azure リソースについて
移行が完了すると、ボットを機能させるために必要なたくさんの Azure リソースが**リソース グループ**に含まれます。 リソースの種類と数は、移行したボットの種類に依存します。 詳細については、後続のセクションを参照してください。

### <a name="registration-bot"></a>登録ボット

これは最も単純なボットです。 Azure のリソース グループには、"ボット チャネル登録" という種類のリソースが 1 つだけ含まれます。 これは Bot Framework 開発者ポータルの以前のボット レコードに相当します。

![Azure のボット チャネル登録ボットの一覧](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Web アプリ ボット
Web アプリ ボット移行により、種類が "Web アプリ ボット" のボット サービス リソースと新しい App Service Web アプリ (下のスクリーンショットで緑) がプロビジョニングされます。 以前の Azure Bot Service (プレビュー) ボットも引き続き表示されます (下のスクリーンショットの赤)。これは削除できます。

![Azure の Web アプリ ボットの一覧](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Functions ボット
Functions ボットの移行により、種類が "Functions ボット" のボット サービス リソースと新しい App Service Functions アプリ (下のスクリーンショットの緑) がプロビジョニングされます。 以前の Azure Bot Service (プレビュー) ボットも引き続き表示されます (下のスクリーンショットの赤)。これは削除できます。

![Azure の Functions ボットの一覧](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>移行をロールバックする

移行中または移行後にボットに問題が発生した場合、**移行をロールバック**できます。 移行は次の手順でロールバックします。

1. [Bot Framework Portal](http://dev.botframework.com) にサインインし、 **[マイ ボット]** をクリックします。
2. ロールバックするボットの **[Roll back migration]\(移行のロールバック\)** ボタンをクリックします。 プロンプトが表示されます。
3. **[Yes, roll back]\(はい、ロールバックします\)** をクリックして続行するか、 **[キャンセル]** をクリックしてロールバックを取り消します。

> [!NOTE]
> ロールバック機能は移行後、一週間利用できます。移行したボットに問題が発生した場合にのみ使用してください。

## <a name="migration-troubleshootingknown-issues"></a>移行のトラブルシューティング/既知の問題
node.js/functions ボットが正常に移行されましたが、応答しません。

* **エンドポイントを確認する**
  * ボット リソースで **[設定]** ブレードに進み、ボット エンドポイントに "コード" クエリ文字列パラメーターとその値があることを確認します。 ない場合、追加する必要があります。
* **kudu のシークレット フォルダーでバックアップを確認する**
  * まれなケースですが、バックアップ シークレット ファイルが競合の原因になっていることがあります。 Kudu の **home\data\Functions\secrets** フォルダーに進み、そこに **host.snapshot** (または **host.backup**) ファイルが見つかった場合、それを削除します。 **host.json** と **messages.json** はいずれも 1 つしか存在しません。 最後に App Service サービスを再起動し、ボットとのチャットを再試行します。

その他の問題については、Azure Support に CRI を提出するか、[GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues) で問題を提出してください。


## <a name="next-steps"></a>次のステップ

これでボットが移行されました。次は、Azure Portal からボットを管理する方法について学習してください。

> [!div class="nextstepaction"]
> [ボットの管理](bot-service-manage-overview.md)
