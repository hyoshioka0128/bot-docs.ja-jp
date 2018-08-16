---
title: ボットの管理 | Microsoft Docs
description: ボット サービス ポータルを通じてボットを管理する方法について説明します。
keywords: Azure portal, ボット管理, Web チャットでのテスト, MicrosoftAppID, MicrosoftAppPassword, アプリケーション設定
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: f64dddedc0215e1277a9570201b14e53fb031d9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302265"
---
# <a name="manage-a-bot"></a>ボットの管理

このトピックでは、Azure portal を使用してボットを管理する方法について説明します。

## <a name="bot-settings-overview"></a>ボットの設定の概要

![ボットの設定の概要](~/media/azure-manage-a-bot/overview.png)

**[概要]** ブレードでは、ボットに関する概要情報を確認できます。 たとえば、ボットの**サブスクリプション ID**、**価格レベル**、**メッセージング エンドポイント**などが表示されます。

## <a name="bot-management"></a>ボットの管理

 ボットの管理オプションのほとんどは、**[BOT MANAGEMENT]\(ボットの管理\)** セクションにあります。 ボットの管理に役立つオプションの一覧を次に示します。

![ボットの管理](~/media/azure-manage-a-bot/bot-management.png)

| オプション |  説明 |
| ---- | ---- |
| **ビルド** | [ビルド] タブには、ボットを変更するためのオプションが用意されています。 このオプションは、**登録のみのボット**では使用できません。 |
| **Test in Web Chat (Web チャットでのテスト)** | 統合 Web チャット コントロールを使用すると、ボットを簡単にテストできます。 |
| **Analytics** | ボットに対する分析が有効になっている場合、Application Insights によって収集されたボットの分析データを表示できます。 |
| **Channels** | ボットがユーザーとの通信に使用するチャネルを構成します。 |
| **設定** | 表示名、分析、メッセージング エンドポイントなどのさまざまなボット プロファイル設定を管理します。 |
| **Speech priming (音声認識の準備)** | LUIS アプリと Bing Speech サービスとの接続を管理します。 |
| **Bot Service pricing (Bot Service の価格)** | ボット サービスの価格レベルを管理します。 |

## <a name="app-service-settings"></a>アプリ サービスの設定

![アプリ サービスの設定](~/media/azure-manage-a-bot/app-service-settings.png)

**[アプリケーションの設定]** ブレードには、ボットの環境、ID、Application Insights キー、Microsoft アプリ ID、および Microsoft アプリ パスワードなど、ボットに関する詳細情報が含まれています。

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID と MicrosoftAppPassword

**MicrosoftAppID** と **MicrosoftAppPassword** は、**[アプリケーションの設定]** ブレードで確認できます。

![Microsoft AppID とパスワード](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> **[Bot Channels Registration]\(ボット チャネル登録\)** ボット サービスには *MicrosoftAppID* が付属しますが、この種類のサービスに関連付けられているアプリ サービスがないため、*MicrosoftAppPassword* を確認するための **[アプリケーションの設定]** ブレードは表示されません。 パスワードを取得するには、最初にパスワードの生成を行う必要があります。 **[Bot Channels Registration]\(ボット チャネル登録\)** のパスワードを生成するには、「[Bot Channels Registration password (ボット チャネル登録のパスワード)](bot-service-quickstart-registration.md#bot-channels-registration-password)」を参照してください

## <a name="next-steps"></a>次の手順
Azure portal の [ボット サービス] ブレードを探索した後は、オンライン コード エディターを使用してボットをカスタマイズする方法について学習しましょう。
> [!div class="nextstepaction"]
> [オンライン コード エディターの使用](bot-service-build-online-code-editor.md)
