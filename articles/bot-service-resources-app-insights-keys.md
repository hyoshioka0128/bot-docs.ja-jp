---
title: Application Insights キー | Microsoft Docs
description: ボットにテレメトリを追加するための Application Insights キーを取得する方法について説明します。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 07fb6e9630996a61932da99b0575d43f4604141e
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389431"
---
# <a name="application-insights-keys"></a>Application Insights キー

Azure **Application Insights** は、Microsoft Azure リソースにアプリケーションに関するデータを表示します。 ボットにテレメトリを追加するには、Azure サブスクリプションとボット用に作成された Application Insights リソースが必要です。 このリソースから、ボットを構成するための次の 3 つのキーを取得できます。

1. インストルメンテーション キー
2. アプリケーション ID
3. API キー

このトピックでは、これらの Application Insights キーを作成する方法について説明します。

> [!NOTE]
> ボットの作成または登録プロセス中に、**Application Insights** を "*オン*" または "*オフ*" にするオプションがありました。 "*オン*" を選択した場合、ボットには必要なすべての Application Insights キーが既にあります。 ただし、"*オフ*" を選択した場合は、このトピックの手順に従って、これらのキーを手動で作成できます。

## <a name="instrumentation-key"></a>インストルメンテーション キー

インストルメンテーション キーを取得するには、次の手順を実行します。
1. [Azure portal](http://portal.azure.com) の [モニター] セクションで、新しい **Application Insights** リソースを作成します (または既存のリソースを使用します)。
![Application Insights の一覧を示すポータルの画面キャプチャ](~/media/portal-app-insights-add-new.png)

2. Application Insights リソースの一覧で、先ほど作成した Application Insight リソースをクリックします。

3. **[Overview]** をクリックします。

4. **[要点]** ブロックを展開し、**[インストルメンテーション キー]** を見つけます。 
![インストルメンテーション キーを示すポータルの画面キャプチャ](~/media/portal-app-insights-instrumentation-key.png)

5. **インストルメンテーション キー**をコピーし、ボットの設定の **[Application Insights Instrumentation Key]\(Application Insights インストルメンテーション キー\)** フィールドに貼り付けます。

## <a name="application-id"></a>アプリケーション ID

アプリケーション ID を取得するには、次の手順を実行します。
1. Application Insights リソースから、**[API アクセス]** をクリックします。

2. **アプリケーション ID** をコピーし、ボットの設定の **[Application Insights Application ID]\(Application Insights アプリケーション ID\)** フィールドに貼り付けます。 
![アプリケーション ID を示すポータルの画面キャプチャ](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>API キー

API キーを取得するには、次の手順を実行します。
1. Application Insights リソースから、**[API アクセス]** をクリックします。

2. **[API キーの作成]** をクリックします。

3. 簡単な説明を入力し、**[テレメトリを読み取る]** をオンにして、**[キーの生成]** をクリックします。
![アプリケーション ID と API キーを示すポータルの画面キャプチャ](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > この **API キー**が再度表示されることはないため、このキーをコピーして保存してください。 このキーを紛失した場合は、新しいキーを作成する必要があります。

4. API キーを、ボットの設定の **[Application Insights API キー]** フィールドにコピーします。

## <a name="additional-resources"></a>その他のリソース
これらのフィールドをボットの設定に接続する方法の詳細については、「[Enable analytics (分析を有効にする)](~/bot-service-manage-analytics.md#enable-analytics)」をご覧ください。