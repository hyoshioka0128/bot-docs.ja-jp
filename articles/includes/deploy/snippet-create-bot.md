---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360876"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| オプション | 説明 |
|:---|:---|
| --name | Azure でのボットのデプロイに使用される一意の名前。 ローカルのボットと同じ名前にすることができます。 名前にスペースやアンダースコアを含めないでください。 |
| --location | ボット サービス リソースを作成するために使用される地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 |
| --lang | ボットの作成に使用する言語: `Csharp` または `Node`。既定値は `Csharp` です。 |
| --resource-group | ボットを作成するリソース グループの名前。 `az configure --defaults group=<name>` を使用して、既定のグループを構成できます。 |