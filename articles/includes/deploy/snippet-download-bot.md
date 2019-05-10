---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035728"
---
現在のプロジェクト ディレクトリの外部にある一時ディレクトリを使用します。 

このコマンドで、save-path 以下にサブディレクトリが作成されます。ただし、指定するパスは既に存在している必要があります。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| オプション | 説明 |
|:---|:---|
| --name | Azure 内のボットの名前。 |
| --resource-group | ボットが属するリソース グループの名前。 |
| --save-path | ボット コードをダウンロードする先の既存のディレクトリ。 |