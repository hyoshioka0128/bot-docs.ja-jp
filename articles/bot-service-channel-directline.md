---
title: Direct Line チャネルに概要
titleSuffix: Bot Service
description: Direct Line チャネルの機能
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/01/2019
ms.author: kamrani
ms.openlocfilehash: cba508d0895b2e2657abd37073bc5c348c1eef44
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2019
ms.locfileid: "73441608"
---
# <a name="about-direct-line"></a>Direct Line の概要

Bot Framework Direct Line チャネルにより、お使いのボットを、ご自身のモバイル アプリ、Web ページ、またはその他のアプリケーションに簡単に統合できます。
Direct Line は、次の 3 つのフォームでご利用いただけます。
- Direct Line サービス: ほとんどの開発者に対応する堅牢なグローバル サービス
- Direct Line App Service 拡張機能: セキュリティとパフォーマンスのための専用 Direct Line 機能 (2019 年 5 月からプライベート プレビューで利用可能)
- Direct Line Speech: 高性能音声用に最適化 (2019 年 5 月からプライベート プレビューで利用可能)

最適な Direct Line サービスを選択するには、それぞれの機能とご自身のソリューションのニーズを評価してください。 これらのサービスは今後シンプルになっていく予定です。

|                            | Direct Line | Direct Line App Service 拡張機能 | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| 可用性とライセンス    | 一般公開 | プライベート プレビュー、SLA なし  | プライベート プレビュー、SLA なし |
| 音声認識およびテキスト読み上げのパフォーマンス | Standard | Standard | 高性能 |
| 統合 OAuth           | はい | はい | いいえ |
| 統合テレメトリ       | はい | はい | いいえ |
| 従来の Web ブラウザーのサポート | はい | いいえ | いいえ |
| Bot Framework SDK のサポート | すべての v3、v4 | v4.5 以上が必要 | v4.5 以上が必要 |
| クライアント SDK のサポート    | JS、C# | JS、C# | C++、C#、Unity |
| Web チャットとの連携  | はい | はい | いいえ|
| 会話履歴 API | はい | はい| いいえ|
| VNet | いいえ | プレビュー* | いいえ |

_* Direct Line App Service 拡張機能は VNET で使用できますが、送信呼び出しの制限はまだ許可されていません。_

## <a name="addtional-resources"></a>その他のリソース
- [ボットを Direct Line に接続する](bot-service-channel-connect-directline.md)
- [ボットを Direct Line Speech に接続する](bot-service-channel-connect-directlinespeech.md)
