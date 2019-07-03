---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252669"
---
コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```

ブラウザー ウィンドウが開いて、サインインできます。

### <a name="set-the-subscription"></a>サブスクリプションを設定する

使用する既定のサブスクリプションを設定します。

```cmd
az account set --subscription "<azure-subscription>"
```

ボットのデプロイに使用するサブスクリプションが不明な場合は、`az account list` コマンドを使用して、お使いのアカウントの `subscriptions` の一覧を表示できます。

ボットのフォルダーに移動します。

```cmd
cd <local-bot-folder>
```