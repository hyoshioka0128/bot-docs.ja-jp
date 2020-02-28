---
title: 状態データの管理 - Bot Service
description: Bot State サービスを使用して、状態データを格納および取得する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: 1dc6fa1a9223e61daaa52cf475095fc08e0356d7
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77519961"
---
# <a name="manage-state-data"></a>状態データの管理

ボットは通常、ストレージを使用して、会話内のユーザーの場所、またはボットとの関係に関連する詳細を追跡します。 Bot Framework SDK では、ボット開発者のためにユーザーおよび会話状態を自動的に管理します。 

当初、Bot Framework には、このデータを格納するための状態サービスが付属していましたが、最新のボット (および Bot Framework SDK の最近のすべてのリリース) では、この一元管理サービスではなく、ボット開発者によって直接制御されているストレージを使用します。 

一元的な Bot Framework State サービスは 2018 年 3 月 30 日時点で廃止されています。 詳細については、「[お知らせ: Bot Framework State サービスは廃止されました - 知っておくべきこと](https://blog.botframework.com/2018/04/02/reminder-the-bot-framework-state-service-has-been-retired-what-you-need-to-know/)」を参照してください。
