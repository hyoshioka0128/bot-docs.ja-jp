---
title: ボットを GroupMe に接続する - Bot Service
description: ボットの GroupMe への接続を構成する方法について説明します。
keywords: ボット チャネル, GroupMe, GroupMe の作成, 資格情報
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: ae2f2e289d6c0b4e90041fc2ad60452b1996d53d
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "75791981"
---
# <a name="connect-a-bot-to-groupme"></a>ボットを GroupMe に接続する

GroupMe グループ メッセージング アプリを使用して、複数のユーザーとコミュニケーションを行うボットを構成できます。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>GroupMe アカウントにサインアップする

GroupMe アカウントを持っていない場合は、[新しいアカウントにサインアップ](https://web.groupme.com/signup)してください。

## <a name="create-a-groupme-application"></a>GroupMe アプリケーションを作成する

ボット用の [GroupMe アプリケーションを作成](https://dev.groupme.com/applications/new)します。

次のコールバック URL を使用します。`https://groupme.botframework.com/Home/Login`

![アプリを作成する](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>資格情報を収集する

1. **[リダイレクト URL]** フィールドで、**client_id =** の後ろの値をコピー します。
2. **[アクセス トークン]** 値をコピーします。

![クライアント ID とアクセス トークンのコピー](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>資格情報を送信する

1. dev.botframework.com で、先ほどコピーした **client_id** 値を **[クライアント ID]** フィールドに貼り付けます。
2. **[アクセス トークン]** 値を **[アクセス トークン]** フィールドにコピーします。
2. **[保存]** をクリックします。

![資格情報を入力する](~/media/channels/GM-StepClientIDToken.png)
