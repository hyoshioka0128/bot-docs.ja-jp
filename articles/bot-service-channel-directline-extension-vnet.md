---
title: VNET 内で Direct Line App Service 拡張機能を使用する
titleSuffix: Bot Service
description: VNET 内で Direct Line App Service 拡張機能を使用する
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: a37ae6f3a9a9cd43c28d353745f0da9065a7886b
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757754"
---
## <a name="use-direct-line-app-service-extension-within-a-vnet"></a>VNET 内で Direct Line App Service 拡張機能を使用する

この記事では、Azure Virtual Network (VNET) で Direct Line App Service 拡張機能を使用する方法について説明します。

### <a name="create-an-app-service-environment-and-other-azure-resources"></a>App Service Environment とその他の Azure リソースを作成する

1. Direct Line App Service 拡張機能は、**Azure App Service Environment** 内でホストされているものも含め、すべての **Azure App Service** で利用できます。 Azure App Service Environment は分離を提供し、VNET 内での作業に最適です。
    - 外部 App Service Environment の作成手順については、「[外部 App Service Environment の作成](https://docs.microsoft.com/en-us/azure/app-service/environment/create-external-ase)」の記事を参照してください。
    - 内部 App Service Environment の作成手順については、「[App Service Environment で内部ロード バランサーを作成して使用する](https://docs.microsoft.com/en-us/azure/app-service/environment/create-ilb-ase)」の記事を参照してください。
1. App Service Environment を作成したら、その中に App Service プランを追加する必要があります。そこでボットをデプロイできます (そして、それにより Direct Line App Service 拡張機能を実行できます)。 これを行うには、次の手順を実行します。
    - [https://resources.azure.com](https://portal.azure.com/ ) に移動します
    - 新しい "App Service プラン" のリソースを作成します。
    - [リージョン] で、目的の App Service Environment を選択します
    - App Service プランの作成を完了します

### <a name="configure-the-vnet-network-security-groups-nsg"></a>VNET ネットワーク セキュリティ グループ (NSG) を構成する

1. Direct Line App Service 拡張機能には、HTTP 要求を発行できるように、送信接続が必要です。 これは、App Service Environment のサブネットに関連付けられている VNET NSG の送信規則として構成できます。 必要な規則は次のとおりです。

|source|Any|
|---|---|
|発信元ポート|*|
|宛先|IP アドレス|
|送信先 IP アドレス|52.155.168.246、13.83.242.172|
|宛先ポート範囲|443|
|Protocol|Any|
|Action|Allow|


![Direct Line 拡張機能のアーキテクチャ](./media/channels/direct-line-extension-vnet.png)

>[!NOTE]
> 指定された IP アドレスは、プレビューのために明示的に指定されています。 今年後半に Azure Bot Service 用のサービス タグへの移行が予定されており、それにより、この構成は変更されます。

### <a name="configure-your-bots-app-service"></a>ボットの App Service を構成する

プレビューのためには、Direct Line App Service 拡張機能が Azure と通信する方法を変更する必要があります。 これを行うには、ポータルまたは `applicationsettings.json` ファイルを使用して、新しい **App Service アプリケーション設定**をアプリケーションに追加します。

- プロパティ:DirectLineExtensionABSEndpoint
- 値: https://dlase.botframework.com/v3/extension

>[!NOTE]
> これは、Direct Line App Service 拡張機能のプレビューにのみ必要です。
