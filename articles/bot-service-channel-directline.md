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
ms.openlocfilehash: b5a6296e65ab05cd8a5af24872d31e5ef356dcf7
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441506"
---
# <a name="about-direct-line"></a>Direct Line の概要

Bot Framework Direct Line チャネルにより、お使いのボットを、ご自身のモバイル アプリ、Web ページ、またはその他のアプリケーションに簡単に統合できます。
Direct Line は、次の 3 つのフォームでご利用いただけます。
- Direct Line サービス: ほとんどの開発者に対応する堅牢なグローバル サービス
- Direct Line App Service 拡張機能: セキュリティとパフォーマンスのための専用 Direct Line 機能 (パブリック プレビューで利用可能)
- Direct Line Speech: 高性能音声用に最適化 (GA)

最適な Direct Line サービスを選択するには、それぞれの機能とご自身のソリューションのニーズを評価してください。 これらのサービスは今後シンプルになっていく予定です。

|                            | Direct Line | Direct Line App Service 拡張機能 | Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| 可用性とライセンス    | 一般公開 | パブリック プレビュー、SLA なし  | GA |
| 音声認識およびテキスト読み上げのパフォーマンス | Standard | Standard | 高パフォーマンス |
| 従来の Web ブラウザーのサポート | はい | 一般提供時 | はい |
| Bot Framework SDK のサポート | すべての v3、v4 | v4.63 以降必須 | v4.63 以降必須 |
| クライアント SDK のサポート    | JS、C# | JS、C# | C++、C#、Unity、JS|
| Web チャットとの連携  | はい | はい | いいえ|
| VNet | いいえ | プレビュー | いいえ |


## <a name="additional-resources"></a>その他のリソース
- [ボットを Direct Line に接続する](bot-service-channel-connect-directline.md)
- [ボットを Direct Line Speech に接続する](bot-service-channel-connect-directlinespeech.md)
