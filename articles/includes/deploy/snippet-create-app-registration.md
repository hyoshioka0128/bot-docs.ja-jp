---
ms.openlocfilehash: a0d110e4975fd3fed943080c5bf4bc71739e877b
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479289"
---
この手順では、Azure Active Directory アプリケーションを作成します。これにより、次のことが可能になります。

- ユーザーが"*Web チャット*" などの一連のチャンネルを介してボットと対話する。
- "*OAuth 接続設定*" の定義により、ユーザーを認証し、ユーザーに代わって保護されたリソースにアクセスするためにボットが使用する "*トークン*" を作成する。

Azure Active Directory アプリケーションを作成するには、次のコマンドを実行します。

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| オプション   | 説明 |
|:---------|:------------|
| display-name | アプリケーションの表示名。 これは、Azure portal の一般的なリソースの一覧と、それが属しているリソース グループに表示されます。|
| パスワード | アプリケーションのパスワード。**クライアント シークレット**とも呼ばれます。 これは、このリソース用に作成するパスワードです。 これは 16 文字以上でなければなりません。また、大文字または小文字を 1 文字以上、特殊文字を 1 文字以上含める必要があります。|
| available-to-other-tenants| 任意の Azure AD テナントからアプリケーションを使用できることを示します。 これを設定すると、お使いのボットが Azure Bot Service チャンネルと連携できるようになります。|

上記のコマンドにより、キー `appId` を含む JSON が出力されます。それをコピーして保存してください。
この `appId` と、ARM デプロイ手順で入力したパスワードを使用して、`appId` と `appSecret` パラメーターにそれぞれ値を割り当てます。
