---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360884"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| オプション | 説明 |
|:---|:---|
| --name | リソース グループの一意名。 名前にスペースやアンダースコアを含めないでください。 |
| --location | リソース グループの作成に使用する地理的な場所。 たとえば、`eastus`、`westus`、`westus2` などです。 場所の一覧を取得するには `az account list-locations` を使用します。 |