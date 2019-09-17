---
ms.openlocfilehash: 28f49af1610f3c154e76d1cfcdb1333f08374f6c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386055"
---
アプリケーションを登録することで、Azure AD を使用してユーザーを認証し、ユーザー リソースへのアクセスを要求できます。 お使いのボットには Azure に登録されたアプリが必要です。このアプリが Bot Framework Service へのボット アクセスを提供し、認証済みメッセージを送受信できるようにします。 Azure CLI を使用してアプリを登録を作成するには、次のコマンドを実行します。

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| オプション   | 説明 |
|:---------|:------------|
| display-name | アプリケーションの表示名。 |
| password | アプリのパスワード。"クライアント シークレット" とも呼ばれます。 パスワードは 16 文字以上でなければなりません。また、大文字または小文字を 1 文字以上、特殊文字を 1 文字以上含める必要があります。|
| available-to-other-tenants| 任意の Azure AD テナントからアプリケーションを使用できるかどうかを示します。 `true` に設定すると、お使いのボットが Azure Bot Service チャネルと連携できるようになります。|

上記のコマンド キーにより JSON と キー `appId` が出力され、ARM デプロイ用にこのキーの値が保存されます。これは、ここで `appId` パラメーターに対して使用されます。 指定されたパスワードは `appSecret` パラメーターで使用されます。

> [!NOTE] 
> 既存のアプリ登録を使用する場合は、次のコマンドを使用できます。
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```
