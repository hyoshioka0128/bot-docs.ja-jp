---
title: タスクの自動化ボットの作成 | Microsoft Docs
description: さらなるユーザーの介入なしでタスクが実行されるボットを設計する方法について説明します。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/13/2018
ms.openlocfilehash: 3bf6bef805e4a86b6e070693660eb5cb20468ffd
ms.sourcegitcommit: f0b22c6286e44578c11c9f15d22b542c199f0024
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404008"
---
# <a name="create-task-automation-bots"></a>タスクの自動化ボットの作成

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

タスクの自動化ボットを使用すると、ユーザーが、特定のタスクまたは一連のタスクを人のサポートなしで完了できます。 この種類のボットは、一般的なアプリや Web サイトとよく似ていることが多く、ユーザーとの通信には、主に、豊富なユーザー コントロールとテキストが使用されます。 ユーザーとの会話を強化するために自然言語を認識する機能を備えている場合もあります。 

## <a name="example-use-case-password-reset"></a>ユース ケースの例: password-reset

タスク ボットの性質をよく理解するために、ユース ケースの例として password-reset について検討します。 

Contoso 社では、パスワードのリセットを求めるヘルプ デスク コールを従業員から 1 日に何件も受け取っています。 Contoso 社は、従業員のパスワードをリセットする単純な繰り返しタスクを自動化し、ヘルプ デスク エージェントがもっと複雑な問題に時間をかけて対処できるようにしたいと考えています。 

Contoso 社の経験豊富な開発者である John は、password-reset タスクを自動化するボットを作成することにします。 John が最初に行うのは、ボットの設計仕様を記述することです。この仕様は、新しいアプリや Web サイトを作成するときと同じように記述します。 

::: moniker range="azure-bot-service-3.0"

### <a name="navigation-model"></a>ナビゲーション モデル

仕様で、ナビテーション モデルを定義します。

![ダイアログの構造](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

ユーザーは、`RootDialog` で開始します。 ユーザーがパスワードのリセットを要求すると、そのユーザーは、  
`ResetPasswordDialog` にリダイレクトされます。 `ResetPasswordDialog` では、ユーザーは、電話番号と誕生日の 2 つの情報を入力するようボットから求められます。 

> [!IMPORTANT]
> この記事で説明するボットの設計は、例示のみを目的としています。 実際のシナリオの password-reset ボットは、より堅牢な ID 認証システムを実装している可能性があります。

### <a name="dialogs"></a>ダイアログ

次に、仕様で各ダイアログ ボックスの外観と機能を記述します。 

#### <a name="root-dialog"></a>ルート ダイアログ

ルート ダイアログには 2 つのオプションがユーザーに示されます。 

1. **パスワードの変更**のシナリオでは、ユーザーが現在のパスワードを知っていて、単純にそのパスワードを変更したいと思っています。
2. **パスワードのリセット**のシナリオでは、ユーザーは自身のパスワードを忘れたか紛失し、新しいパスワードを生成する必要があります。

> [!NOTE]
> この記事では、簡潔にするために、**パスワードのリセット** フローについてのみ説明します。

仕様では、次のスクリーンショットに示すように、ルート ダイアログを記述します。

![ダイアログの構造](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>ResetPassword ダイアログ

ユーザーがルート ダイアログから **[パスワードのリセット]** を選択すると、`ResetPassword` ダイアログが呼び出されます。 
その後、`ResetPassword` ダイアログによって他の 2 つのダイアログが呼び出されます。 
最初に呼び出されるのは `PromptStringRegex` ダイアログで、ユーザーの電話番号が収集されます。 
次に呼び出されるのは `PromptDate` ダイアログです。ここでは、ユーザーの誕生日が収集されます。 

> [!NOTE]
> この例では、John は 2 つの別個のダイアログを使って、ユーザーの電話番号と誕生日を収集するロジックを実装することにしました。 このアプローチは、各ダイアログに必要なコードが簡単になるだけでなく、将来的にこれらのダイアログを別のシナリオで使用できる可能性が高くなります。 

仕様で `ResetPassword` ダイアログを記述します。

![ダイアログの構造](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>PromptStringRegex ダイアログ

`PromptStringRegex` ダイアログでは、ユーザーは電話番号を入力するように求められ、ユーザーが入力した電話番号が、有効な形式と一致するかどうかが検証されます。 
また、ユーザーが無効な入力を繰り返した場合のシナリオも含まれます。 
仕様で `PromptStringRegex` ダイアログを記述します。

![ダイアログの構造](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>プロトタイプ

最後に仕様では、ユーザーがボットと通信して、password-reset タスクが正常に完了する例を提供します。

![ダイアログの構造](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

::: moniker-end 

## <a name="bot-app-or-website"></a>ボットか、アプリか、Web サイトか

タスクの自動化ボットがアプリまたは Web サイトによく似ているのなら、代わりにアプリや Web サイトだけを構築すればよいのではないか、と考える人がいるかもしれません。 シナリオによっては、ボットではなく、アプリや Web サイトを構築する方が完全に合理的である場合があります。 [Bot Framework Direct Line API][directLineAPI] または <a href="https://aka.ms/BotFramework-WebChat" target="_blank">Web チャット コントロール</a>を使用して、お使いのボットをアプリに埋め込むでもこともできます。 ボットをアプリのコンテキスト内に実装すれば、充実したアプリ エクスペリエンスと会話エクスペリエンスの両方が 1 か所で実現するためベストです。 

しかし、多くの場合、アプリまたは Web サイトの構築は、ボットを構築するよりも格段に複雑で、コストもかかります。 アプリや Web サイトでは、多くの場合、複数のクライアントとプラットフォームをサポートする必要があり、パッケージ化や展開が面倒で、時間もかかります。アプリをダウンロードしてインストールするというユーザー エクスペリエンスは必ずしも理想的とはいえません。 これらの理由から、当面の問題については、ほとんどの場合、ボットを使用する方がずっと簡単に解決できるのです。 

さらに、ボットは、簡単に展開および拡張できるという自由さも備えています。 たとえば、開発者は、自然言語および音声機能を password-reset ボットに追加することで、音声通話によるボットへのアクセスを可能にできます。また、テキスト メッセージのサポートを追加することもできます。 会社が建物全体にキオスクをセットアップして、password-reset ボットをそのエクスペリエンスに埋め込むこともできます。

::: moniker range="azure-bot-service-3.0"
<!-- TODO: SimpleTaskAutomation no longer exists
## Sample code

For a complete sample that shows how to implement simple task automation using the Bot Builder SDK for .NET, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.

For a complete sample that shows how to implement simple task automation using the Bot Builder SDK for Node.js, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.
-->

## <a name="additional-resources"></a>その他のリソース

- [ダイアログ](~/dotnet/bot-builder-dotnet-dialogs.md)
- [ダイアログで会話フローを管理する (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [ダイアログで会話フローを管理する (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)

::: moniker-end

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
