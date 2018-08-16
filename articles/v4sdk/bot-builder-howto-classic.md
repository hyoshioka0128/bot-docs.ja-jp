---
title: SDK V4 で.NET SDK V3 のボットを実行する方法 | Microsoft Docs
description: 従来の NuGet パッケージを使用して、ボットを 3.x から 4.0 に変換する方法について説明します。
keywords: 移行, クラシック ボット, v3 の変換, v3 から v4
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a808076e4865a181802b85cfc24ce342dbf23cba
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301491"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>SDK 4.0 で.NET SDK V3 のボットを実行する方法

**Microsoft.Bot.Builder.Classic** NuGet パッケージを使用すると、Microsoft Bot Framework のバージョン 3.x からバージョン 4.0 へのボットの移行が容易になります。

**注:** このプロセスは、**ASP.NET Web アプリケーション (.NET Framework)** ボットでのみ機能します。 **ASP.NET Core Web アプリケーション** ボットでは機能しません。

## <a name="the-process"></a>プロセス

プロセスは比較的単純です。

- **Microsoft.Bot.Builder.Classic** NuGet パッケージをプロジェクトに追加します。
    - **Autofac** NuGet パッケージを更新することが必要な場合もあります。
- **Microsoft.Bot.Builder.Classic** 名前空間を更新します。
- 4.0 の **ITurnContext** ベースの **Conversation.SendAsync()** を使用してダイアログを呼び出します。

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>Microsoft.Bot.Builder.Classic NuGet パッケージを追加する

**Microsoft.Bot.Builder.Classic** NuGet パッケージを追加するには、**[NuGet パッケージの管理]** を使用してパッケージを追加します。

### <a name="update-the-namespaces"></a>名前空間を更新する

名前空間を更新するには、見つかった次の `using` ステートメントをすべて削除します。

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

次の `using` ステートメントを代わりに追加します。

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

`[BotAuthentication]` 行がある場合は、この行を削除するか、コメントにします。

### <a name="invoke-your-3x-dialog"></a>3.x ダイアログを呼び出す

`Conversation.SendAsync` を使用している 3.x ダイアログを呼び出すには、**Activity** ではなく、4.0 の **ITurnContext** を受け取る必要があります。

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

**ITurnContext** オブジェクトがなくても、**Activity** オブジェクトがあれば、次のように **ITurnContext** オブジェクトを取得できます。

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>アセンブリの競合を修正する

従来の NuGet パッケージで作成されたボットには、アセンブリとの競合があります。 これは、ビルド後に Visual Studio の [エラー一覧] ウィンドウに表示されます。

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>"Warning: Found conflicts between different versions of the same dependent assembly"\(警告: 同じ依存アセンブリの異なるバージョン間で競合が見つかりました\) と表示された場合

**"Found conflicts between different versions of the same dependent assembly"\(同じ依存アセンブリの異なるバージョン間で競合が見つかりました\)** というテキストで始まる警告が表示された場合は、次の手順を実行します。

- 警告メッセージをダブルクリックします。 [アプリケーション構成ファイル内でバインド リダイレクト レコードを追加してこれらの競合を修正しますか?] とたずねるダイアログ ボックスが表示されます。
- [はい] をクリックします。
- プロジェクトを再度ビルドします。

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>"Error: Missing Method Exception on startup"\(エラー: 起動時にメソッドが見つからない例外が発生しました\) と表示された場合

.NET Standard には、古い .NET 4.6 プロジェクトを取得し、4.6.1 にアップグレードした後、そのプロジェクトで .NET Standard ライブラリを使用しようとしたときに発生するように思われるバグがあります。 基本的に、動的なスワップ アウトを試みる 2 つの異なる System.Net.Http アセンブリがあります。対処法として、System.Net.Http の Web.config にバインド リダイレクトを追加します。 

このエラーが発生した場合は、Web.config ファイルに次のコードを追加します。

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

この問題の詳細については、「[System.Net.Http v4.2.0.0 being copied/loaded from MSBuild tooling #25773 (MSBuild ツールからコピーされる/読み込まれる System.Net.Http v4.2.0.0 #25773)](https://github.com/dotnet/corefx/issues/25773)」をご覧ください。

## <a name="sample-of-a-converted-bot"></a>変換後のボットのサンプル

既に変換されたボットについては、4.0 で動作するように変換された 3.x ボットを示す [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic) サンプルを参照してください。

## <a name="limitations"></a>制限事項
Microsoft.Bot.Builder.Classic は.NET 4.61 ライブラリ限定であり、.NET Core では動作しません。
