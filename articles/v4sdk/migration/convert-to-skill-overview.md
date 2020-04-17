---
title: スキルへの v3 ボットの変換の概要 - Bot Service
description: v3 ボットをスキルに変換して v4 ボットから使用する方法の概要について説明します。
keywords: ボットの移行
author: clearab
ms.author: anclear
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/28/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2483efea11905837132676f59d15076620d955b5
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80117777"
---
# <a name="convert-a-v3-bot-to-a-skill"></a>v3 ボットをスキルに変換する

シナリオによっては v3 ボットから v4 ボットにすぐに移行することは有益でないが、v4 SDK で利用可能な追加機能を利用したい場合があります。 このような場合、v3 ボットをスキルに変換し、v4 SDK に基づくスキル コンシューマー ボットを作成して、v3 ボットにメッセージを渡すことは理にかなっている場合があります。 スキルとスキル コンシューマーの詳細については、[スキルの概要に関する記事](../skills-conceptual.md)を参照してください。

より複雑なボットの場合、この手法によって段階的な移行も可能です。 スキル コンシューマー ボットで処理するメッセージと、スキルに渡すものを制御できます。 これにより、機能を段階的に新しいボットに移行できるようになり、すべての機能が移行された後、最終的にスキルを削除できます。

## <a name="whats-required"></a>必要なこと

### <a name="upgrade-your-v3-bot-sdk-to-the-most-current-3x-version"></a>v3 ボット SDK を最新の 3.x バージョンにアップグレードする

ボットをスキルに変換するために必要なフックが、.NET SDK のバージョン 3.30.2 と JavaScript SDK のバージョン 3.30.0 に追加されました。

### <a name="convert-your-v3-bot-to-a-skill"></a>v3 ボットをスキルに変換する

SDK バージョンをアップグレードしたら、スキル コンシューマー ボットに対するメッセージの送受信を処理するロジックをボットに追加する必要があります。

### <a name="create-a-v4-skill-consumer-bot"></a>v4 スキル コンシューマー ボットを作成する

次に、スキル コンシューマーとして機能するボットを作成し、スキルにルーティングするメッセージを決定するロジックを追加する必要があります。 この新しいボットは、顧客の対話の相手になります。これに、v4 SDK の拡張された機能の上に構築された付加的な機能を追加します。

### <a name="connect-your-users-to-the-new-skill-consumer-bot"></a>ユーザーを新しいスキル コンシューマー ボットに接続する

最後に、v3 ボットを、作成した新しい v4 ボットに置き換える必要があります。

## <a name="get-started"></a>はじめに

使い始める準備はできていますか。

- [.NET v3 ボットをスキルに変換する](net-v3-as-skill.md)
- [Javascript v3 ボットをスキルに変換する](javascript-v3-as-skill.md)
