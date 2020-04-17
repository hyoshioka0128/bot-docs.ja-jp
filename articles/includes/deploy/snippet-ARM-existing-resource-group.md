---
ms.openlocfilehash: 27c6079030e6a1da73646c6c325a78614960fa07
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80499852"
---
この手順では、ボットのデプロイ ステージを設定するボット アプリケーション サービスを作成します。 既存のリソース グループを使用する場合は、既存のアプリ サービス プランを使用するか、新しいものを作成できます。 両方のオプションの手順を以下に示します。

結果として得られる JSON 出力から、次の手順で**登録サブスクリプション ID** の値として使用するために、**ID** フィールドの値をコピーします。

> [!NOTE]
> この手順は完了するまでに数分かかることがあります。

**オプション 1: 既存の App Service プラン**

この場合、既存の App Service プランが使用されますが、Web アプリと Bot Channels Registration は新しく作成されます。

次のコマンドにより、ボットの ID と表示名が設定されます。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 ボットの表示名は変更可能です。

```cmd
az group deployment create --resource-group "<name-of-resource-group>" --template-file "<path-to-template-with-preexisting-rg.json>" --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" newWebAppName="<bot-app-service-name>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

**オプション 2: 新しい App Service プラン**

この場合は、App Service プラン、Web アプリ、および Bot Channels Registration が作成されます。

```cmd
az group deployment create --resource-group "<name-of-resource-group>" --template-file "<path-to-template-with-preexisting-rg.json>" --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" newWebAppName="<bot-app-service-name>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | ボット チャンネル登録に使用する表示名。 既定値は `botId` パラメーターの値です。|
| resource-group | Azure リソース グループの名前。 |
| template-file | ARM テンプレートへのパス。 通常、`template-with-preexisting-rg.json` ファイルはプロジェクトの `deploymentTemplates` フォルダーにあります。 これは、既存のテンプレート ファイルのパスです。 パスは、絶対パス、または現在のディレクトリに対する相対パスのどちらでもかまいません。 すべてのボット テンプレートにより、ARM テンプレート ファイルが生成されます。|
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | key=value のペアの一覧として指定されるデプロイ パラメーター。 次のパラメーター値を入力します。

- `appId` - 前の手順で生成された "*アプリ ID*" の値。
- `appSecret` - 前の手順で指定したパスワード。
- `botId` - 作成するボット チャンネル登録リソースの名前。 名前はグローバルに一意である必要があります。 これは、不変のボット ID として使用されます。 また、変更可能な既定の表示名としても使用されます。
- `newWebAppName` - ボット アプリケーション サービスの名前。
- `newAppServicePlanName` - 作成するアプリケーション サービス プラン リソースの名前。
- `newAppServicePlanLocation` - アプリケーション サービス プランの場所。