---
title: ボットのセキュリティ保護 | Microsoft Docs
description: HTTPS と Bot Framework 認証を使用して、ボットをセキュリティで保護する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8741dee2724d5e3c1e650b86957aabec5716da59
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496642"
---
# <a name="secure-your-bot"></a>ボットのセキュリティ保護

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector サービスを使用して、さまざまな通信チャネル (Skype、SMS、電子メールなど) にボットを接続できます。 この記事では、HTTPS と Bot Framework 認証を使用して、ボットをセキュリティで保護する方法について説明します。

## <a name="use-https-and-bot-framework-authentication"></a>HTTPS と Bot Framework 認証を使用する

Bot Framework [Connector](bot-builder-dotnet-concepts.md#connector) だけがボットのエンドポイントにアクセスできるようにするには、HTTPS のみを使用するようにボットのエンドポイントを構成し、ボットを[登録](~/bot-service-quickstart-registration.md)してアプリ ID とパスワードを取得することによって、Bot Framework 認証を有効にします。 

## <a name="configure-authentication-for-your-bot"></a>ボットの認証を構成する

ボットの web.config ファイルにボットのアプリ ID とパスワードを指定します。 

> [!NOTE]
> 自分のボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

次に、Bot Framework SDK for .NET を使用してボットを作成するときに、`[BotAuthentication]` 属性を使用して認証資格情報を指定します。 

web.config ファイルに保存されている認証資格情報を使用するには、パラメーターを指定せずに `[BotAuthentication]` を指定します。

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

認証資格情報に他の値を使用するには、`[BotAuthentication]` 属性を指定し、それらの値を渡します。

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>その他のリソース

- [Bot Framework SDK for .NET](bot-builder-dotnet-overview.md)
- [Bot Builder SDK for .NET の主要概念](bot-builder-dotnet-concepts.md)
- [Bot Framework へのボットの登録](~/bot-service-quickstart-registration.md)
