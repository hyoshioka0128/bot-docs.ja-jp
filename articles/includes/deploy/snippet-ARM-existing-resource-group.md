---
ms.openlocfilehash: 029275eae6f7b0b4448613ede898575eb491a56f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386071"
---
既存のリソース グループを使用する場合は、既存の App Service プランを使用するか、新しい App Service プランを作成できます。 両方のオプションの手順を以下に示します。 

**オプション 1:既存の App Service プラン** 

この場合、既存の App Service プランが使用されますが、Web アプリと Bot Channels Registration は新しく作成されます。 

> [!NOTE]
> 次のコマンドにより、ボットの ID と表示名が設定されます。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 ボットの表示名は変更可能です。

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**オプション 2:新しい App Service プラン**

この場合は、App Service プラン、Web アプリ、および Bot Channels Registration が作成されます。 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | デプロイのフレンドリ名。 |
| resource-group | Azure リソース グループの名前。 |
| template-file | ARM テンプレートへのパス。 プロジェクトの `deploymentTemplates` フォルダーで指定された `template-with-preexisting-rg.json` ファイルを使用できます。 |
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | デプロイ パラメーターの値を提供します。 `az ad app create` コマンドを実行して取得した `appId` 値。 `appSecret` は、前の手順で指定したパスワードです。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 これは、変更可能なボットの表示名を構成するときにも使用されます。 `newWebAppName` は、作成する Web アプリの名前です。 `newAppServicePlanName` は App Service プランの名前です。 `newAppServicePlanLocation` は、App Service プランの場所です。 |
