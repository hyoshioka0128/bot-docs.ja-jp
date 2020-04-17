---
ms.openlocfilehash: 4cee4c59fc7baa4d9aa18b573f7aebce8e99d1bb
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77479290"
---
この手順では、ボットのデプロイ ステージを設定するボット アプリケーション サービスを作成します。 ARM テンプレート、新しいサービス プラン、新しいリソース グループを使用します。

結果として得られる JSON 出力から、次の手順で**登録サブスクリプション ID** の値として使用するために、**ID** フィールドの数値をコピーします。

> [!NOTE]
> この手順は完了するまでに数分かかることがあります。

```cmd
az deployment create --template-file "<path-to-template-with-new-rg.json" --location <region-location-name> --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" botSku=F0 newAppServicePlanName="<new-service-plan-name>" newWebAppName="<bot-app-service-name>" groupName="<new-group-name>" groupLocation="<region-location-name>" newAppServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | ボット チャンネル登録に使用する表示名。 既定値は `botId` パラメーターの値です。|
| template-file | ARM テンプレートへのパス。 通常、`template-with-new-rg.json` ファイルはボット プロジェクトの `deploymentTemplates` フォルダーにあります。 これは、既存のテンプレート ファイルのパスです。 パスは、絶対パス、または現在のディレクトリに対する相対パスのどちらでもかまいません。 すべてのボット テンプレートにより、ARM テンプレート ファイルが生成されます。|
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | key=value のペアの一覧として指定されるデプロイ パラメーター。 次のパラメーター値を入力します。

- `appId` - 前の手順で生成された "*アプリ ID*" の値。
- `appSecret` - 前の手順で指定したパスワード。
- `botId` - 作成するボット チャンネル登録リソースの名前。 名前はグローバルに一意である必要があります。 これは、不変のボット ID として使用されます。 また、変更可能な既定の表示名としても使用されます。
- `botSku` - 価格レベル。F0 (無料) または S1 (Standard) を指定できます。
- `newAppServicePlanName` - 新しいアプリケーション サービス プランの名前。
- `newWebAppName` - ボット アプリケーション サービスの名前。
- `groupName` - 新しいリソース グループの名前。
- `groupLocation` - Azure リソース グループの場所。
- `newAppServicePlanLocation` - アプリケーション サービス プランの場所。