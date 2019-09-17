---
ms.openlocfilehash: fd279af81e0c94ac22f54429cb8ae330e5d60661
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385998"
---
Azure で新しいリソース グループを作成し、ARM テンプレートを使用して、そこで指定されたリソースを作成します。 この場合は、App Service プラン、Web アプリ、Bot Channels Registration が提供されます。

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| オプション   | 説明 |
|:---------|:------------|
| name | デプロイのフレンドリ名。 |
| template-file | ARM テンプレートへのパス。 プロジェクトの `deploymentTemplates` フォルダーに用意されている `template-with-new-rg.json` ファイルを使用できます。 |
| location |場所。 値のソース: `az account list-locations` `az configure --defaults location=<location>` を使用して、既定の場所を構成できます。 |
| parameters | デプロイ パラメーターの値を提供します。 `az ad app create` コマンドを実行して取得した `appId` 値。 `appSecret` は、前の手順で指定したパスワードです。 `botId` パラメーターはグローバルに一意である必要があり、不変のボット ID として使用されます。 これは、変更可能なボットの表示名を構成するときにも使用されます。 `botSku` は価格レベルです。F0 (無料) または S1 (Standard) を指定できます。 `newAppServicePlanName` は App Service プランの名前です。 `newWebAppName` は、作成する Web アプリの名前です。 `groupName` は、作成する Azure リソース グループの名前です。 `groupLocation` は、Azure リソース グループの場所です。 `newAppServicePlanLocation` は、App Service プランの場所です。 |
