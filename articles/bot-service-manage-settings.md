---
title: ボット設定の構成 | Microsoft Docs
description: Azure Portal を使用してボットのさまざまなオプションを構成する方法について説明します。
keywords: ボット設定の構成, 表示名, アイコン, Application Insights, 設定ブレード
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 640e5675adce627ed14d732e529e9a86ec8b195f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996689"
---
# <a name="configure-bot-settings"></a>ボット設定の構成

ボット設定 (表示名、アイコン、Application Insights など) は、**[設定]** ブレードで表示および変更できます。

![ボット設定ブレード](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

以下に示すのは、**[設定]** ブレードのフィールドの一覧です。

| フィールド | 説明 |
| :---  | :---        |
| アイコン | 各チャネルでボットを視覚的に識別したり、Skype、Cortana、およびその他のサービスでアイコンとして使用するためのカスタム アイコンです。 このアイコンは PNG 形式である必要があり、サイズは 30 K 以下である必要があります。 この値はいつでも変更できます。 |
| 表示名 | チャネルやディレクトリでのボットの名前です。 この値はいつでも変更できます。 文字数は最大 35 文字です。 |
| ボット ハンドル | ボットの一意識別子です。 この値は、Bot Service を使ってボットを作成した後には変更できません。 |
| メッセージング エンドポイント | ボットと通信するためのエンドポイントです。 |
| Microsoft アプリ ID | ボットの一意識別子です。 この値は変更できません。 **[管理]** リンクをクリックすると、新しいパスワードを生成できます。 |
| Application Insights インストルメンテーション キー | ボット テレメトリの一意キーです。 このボットのボット テレメトリを受信する場合は、このフィールドに Azure Application Insights キーをコピーします。 この値は省略可能です。 Azure Portal で作成されたボットに対しては、このキーが自動生成されます。 このフィールドについて詳しくは、「[Application Insights キー](~/bot-service-resources-app-insights-keys.md)」をご覧ください。 |
| Application Insights API キー | ボット分析の一意キーです。 ボットに関する分析をダッシュ ボードに表示する場合は、このフィールドに Azure Application Insights API キーをコピーします。 この値は省略可能です。 このフィールドについて詳しくは、「[Application Insights キー](~/bot-service-resources-app-insights-keys.md)」をご覧ください。 |
| Application Insights アプリケーション ID | ボット分析の一意キーです。 ボットに関する分析をダッシュ ボードに表示する場合は、このフィールドに Azure Insights アプリケーション ID キーをコピーします。 この値は省略可能です。 Azure Portal で作成されたボットに対しては、このキーが自動生成されます。 このフィールドについて詳しくは、「[Application Insights キー](~/bot-service-resources-app-insights-keys.md)」をご覧ください。 |

> [!NOTE]
> ボットの設定を変更したら、クリックして、ブレードの上部にある **[保存]** ボタンをクリックして、新しいボット設定を保存します。

## <a name="next-steps"></a>次の手順
ここでは、ボット サービスの設定を構成する方法について説明しました。次は、音声認識の標準を構成する方法について学習しましょう。
> [!div class="nextstepaction"]
> [音声認識の標準](bot-service-manage-speech-priming.md)