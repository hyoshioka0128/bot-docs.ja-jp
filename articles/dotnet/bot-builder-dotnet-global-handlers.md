---
title: グローバル メッセージ ハンドラーの実装 | Microsoft Docs
description: Bot Framework SDK for .NET を使用して、特定のキーワードを含むユーザー入力をボットがリッスンして処理する方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 86964bc39a95a23f397af649cfac6e2784dd588a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224367"
---
# <a name="implement-global-message-handlers"></a>グローバル メッセージ ハンドラーの実装

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>ユーザー入力内のキーワードをリッスンする

次のチュートリアルでは、Bot Framework SDK for .NET を使用してグローバル メッセージ ハンドラーを実装する方法を示します。

最初に、`Global.asax.cs` を使用して `GlobalMessageHandlersBotModule` を登録します。これは次のように実装されます。 この例で、モジュールは、設定変更要求の管理用 (`SettingsScorable`) とキャンセル要求の管理用 (`CancelScoreable`) の 2 つのスコアラブルを登録します。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable` には、トリガーを定義する `PrepareAsync` メソッドが含まれています。メッセージ テキストが "cancel" の場合は、このスコアラブルがトリガーされます。

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

"cancel" 要求が受信されると、`CancelScoreable` 内の `PostAsync` メソッドによってダイアログ スタックがリセットされます。 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

"change settings" 要求が受信されると、`SettingsScorable` 内の `PostAsync` メソッドによって `SettingsDialog` が呼び出され (要求がそのダイアログに渡され)、`SettingsDialog` がダイアログ スタックの最上部に追加されて会話の管理下に置かれます。

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>サンプル コード

Bot Framework SDK for .NET を使用してグローバル メッセージ ハンドラーを実装する方法を示す完全なサンプルについては、GitHub の「<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Global Message Handlers sample (グローバル メッセージ ハンドラーのサンプル)</a>」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- [会話フローの設計と制御](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">Bot Framework SDK for .NET リファレンス</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">グローバル メッセージ ハンドラーのサンプル (GitHub)</a>
