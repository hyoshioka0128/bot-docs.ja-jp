---
ms.openlocfilehash: c664749bc6ad63d1b0f60e001b2603898e58a9ea
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "70386010"
---
ボットをローカルで作成してテストしたら、Azure にデプロイできます。 コマンド プロンプトを開いて Azure portal にログインします。

```cmd
az login
```
ブラウザー ウィンドウが開いて、サインインできます。

> [!NOTE]
> US Gov などの Azure 以外のクラウドにボットをデプロイする場合は、`az login` の前に `az cloud set --name <name-of-cloud>` を実行する必要があります。この場合、&lt;name-ofcloud> は、`AzureUSGovernment` などの登録済みクラウドの名前になります。 パブリック クラウドに戻る場合は、`az cloud set --name AzureCloud` を実行できます。 
