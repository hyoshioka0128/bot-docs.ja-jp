---
title: 同じ .NET Framework プロジェクト内での既存のボットの移行 | Microsoft Docs
description: 同じプロジェクトを使用して、既存の v3 ボットを取得し、それを v4 SDK に移行します。
keywords: bot の移行, formflow, ダイアログ, v3 ボット
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8bcb2e545cea640f74a37cac20f16b288c690956
ms.sourcegitcommit: fd60ad0ff51b92fa6495b016e136eaf333413512
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/06/2019
ms.locfileid: "55764189"
---
# <a name="migrate-a-bot-within-the-same-net-framework-project"></a>同じ .NET Framework プロジェクト内でのボットの移行

Bot Framework SDK v4 は、SDK v3 と同じ基になる REST API に基づいています。 ただし、SDK v4 は、開発者がより柔軟にボットを制御できるように、以前のバージョンの SDK をリファクタリングしたものです。 この SDK の主な変更点は次のとおりです。<!--TODO: Replace with a snippet summary of changes that includes a link to the concept topic.-->

- 状態は、状態管理オブジェクトとプロパティ アクセサーを使用して管理されます。
- ターン ハンドラーの設定とターン ハンドラーへのアクティビティの受け渡しが変更されました。
- scoreable は存在しなくなりました。 ダイアログに制御を渡す前に、ターン ハンドラー内で "グローバル" コマンドの有無を確認できます。
- 以前のバージョンのものとは大きく異なる新しい Dialogs ライブラリ。 コンポーネント ダイアログ、ウォーターフォール ダイアログ、v4 用の FormFlow ダイアログのコミュニティ実装を使用して、古いダイアログを新しいダイアログ システムに変換する必要があります。

この記事では、v3 の [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) を取り上げ、これを v4 ボットに変換します。 <!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->プロジェクトの種類は変換せずにボットを変換するので、.NET Framework プロジェクトであることに変わりはありません。 この変換は次の手順に分けられます。

1. NuGet パッケージを更新してインストールする
1. Global.asax.cs ファイルを更新する
1. MessagesController クラスを更新する
1. ダイアログを変換する

## <a name="update-and-install-nuget-packages"></a>NuGet パッケージを更新してインストールする

1. **Microsoft.Bot.Builder.Azure** を最新の安定バージョンに更新します。

    **Microsoft.Bot.Builder** パッケージと **Microsoft.Bot.Connector** は依存関係であるため、これらのパッケージも更新されます。

1. **Microsoft.Bot.Builder.History** パッケージを削除します。 これは v4 SDK には含まれていません。
1. **Autofac.WebApi2** を追加します。

    ASP.NET の依存関係挿入に役立つため、これを使用します。

1. **Bot.Builder.Community.Dialogs.Formflow** を追加します。

    これは、v3 の FormFlow 定義ファイルから v4 のダイアログを構築するためのコミュニティ ライブラリです。 その依存関係の 1 つとして、**Microsoft.Bot.Builder.Dialogs** があるので、これも自動的にインストールされます。

この時点でビルドすると、コンパイラ エラーが発生します。 これらは無視してかまいません。 変換が完了すると、作業コードが作成されます。

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>Global.asax.cs ファイルを更新する

スキャフォールディングの一部が変更されたので、v4 では[状態管理](/articles/v4sdk/bot-builder-concept-state.md)インフラストラクチャの一部を自分で設定する必要があります。 たとえば、v4 ではボット アダプターを使用して認証を処理し、アクティビティをボット コードに転送します。また、状態プロパティを事前に宣言しておきます。

`DialogState` の状態プロパティを作成します。これは、v4 のダイアログのサポートに必要になります。 依存関係挿入を使用して、必要な情報をコントローラーとボット コードに取得します。

**Global.asax.cs** 内で、次の手順を実行します。

1. `using` ステートメントを更新します。
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. **Application_Start** メソッドから次の行を削除します。
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    次の行を挿入します。
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. 参照されなくなった **RegisterBotModules** メソッドを削除します。
1. **BotConfig.UpdateConversationContainer** メソッドを、この **BotConfig.Register** メソッドに置き換えます。このメソッドを使用して、依存関係挿入をサポートするために必要なオブジェクトを登録します。
    > [!NOTE]
    > このボットでは、_ユーザー_状態も_個人的な会話_状態も使用していません。 これらを含む行は、ここではコメント アウトされています。
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>MessagesController クラスを更新する

v4 では、このクラスでターン ハンドラーを呼び出すので、多くの変更を行う必要があります。 ターン ハンドラー自体を除き、このほとんどは定型句と考えることができます。 **Controllers\MessagesController.cs** ファイル内で、次の手順を実行します。

1. `using` ステートメントを更新します。
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. クラスから `[BotAuthentication]` 属性を削除します。 v4 では、ボットのアダプターが認証を処理します。
1. 次のフィールドを追加します。 **ConversationState** は会話を対象とする状態を管理し、**IStatePropertyAccessor\<DialogState>** は v4 のダイアログをサポートするために必要となります。
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. 次の処理を実行するコンストラクターを追加します。
    - インスタンス フィールドを初期化します。
    - ASP.NET の依存関係挿入を使用して、パラメーター値を取得します (これをサポートするために、**Global.asax.cs** でこれらのクラスのインスタンスを登録しました)。
    - ダイアログ セットを作成して初期化します。ダイアログ セットからダイアログ コンテキストを作成できます (v4 ではこれを明示的に行う必要があります)。
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. **Post** メソッドの本体を置き換えます。 このメソッド内でアダプターを作成し、それを使用してメッセージ ループ (ターン ハンドラー) を呼び出します。 各ターンの最後に `SaveChangesAsync` を使用して、ボットが行った状態の変更を保存します。

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. ボットの[ターン ハンドラー](/articles/v4sdk/bot-builder-basics.md#the-activity-processing-stack) コードを含む **OnTurnAsync** メソッドを追加します。
    > [!NOTE]
    > v4 には scoreable は存在しません。 アクティブ ダイアログを続行する前に、ボットのターン ハンドラーでユーザーからの `cancel` メッセージの有無を確認します。
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. "_メッセージ_" アクティビティだけを処理するため、**HandleSystemMessage** メソッドは削除できます。

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>CancelScorable クラスと GlobalMessageHandlersBotModule クラスを削除する

v4 には scorable が存在せず、`cancel` メッセージに対応するようにターン ハンドラーを更新したので、**CancelScorable** (**Dialogs\CancelScorable.cs** 内) クラスと **GlobalMessageHandlersBotModule** クラスを削除できます。

## <a name="convert-your-dialogs"></a>ダイアログを変換する

v4 SDK に移行するために、元のダイアログに多くの変更を加えます。 現時点では、コンパイラ エラーは気にしないでください。 これらは変換が完了したら解決します。
元のコードを必要以上に変更しないようにするために、移行の完了後もコンパイラの警告がいくつか残ります。

すべてのダイアログは、v3 の `IDialog<object>` インターフェイスを実装するのではなく、`ComponentDialog` から派生します。

このボットには、変換する必要があるダイアログが 4 つあります。

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | オプションを提示し、他のダイアログを開始します。 |
| [InstallAppDialog](#update-the-install-app-dialog) | コンピューターにアプリをインストールする要求を処理します。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | ローカル コンピューターの管理者権限の要求を処理します。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | パスワードをリセットする要求を処理します。 |

これらは入力を収集しますが、コンピューター上でこれらの操作を実行するわけではありません。

### <a name="make-solution-wide-dialog-changes"></a>ソリューション全体のダイアログを変更する

1. ソリューション全体で、出現するすべての `IDialog<object>` を `ComponentDialog` に置き換えます。
1. ソリューション全体で、出現するすべての `IDialogContext` を `DialogContext` に置き換えます。
1. 各ダイアログ クラスで、`[Serializable]` 属性を削除します。

ダイアログ内の制御フローとメッセージングは同じ方法で処理されなくなったので、各ダイアログを変換するときにこれを変更する必要があります。

| Operation | v3 コード | v4 コード |
| :--- | :--- | :--- |
| ダイアログの開始を処理する | `IDialog.StartAsync` を実装する | これをウォーターフォール ダイアログの最初のステップにするか、`Dialog.BeginDialogAsync` を実装する |
| ダイアログの継続を処理する | `IDialogContext.Wait` を呼び出す | ウォーターフォール ダイアログにステップを追加するか、`Dialog.ContinueDialogAsync` を実装する |
| ユーザーにメッセージを送信する | `IDialogContext.PostAsync` を呼び出す | `ITurnContext.SendActivityAsync` を呼び出す |
| 子ダイアログを開始する | `IDialogContext.Call` を呼び出す | `DialogContext.BeginDialogAsync` を呼び出す |
| 現在のダイアログが完了したことを通知する | `IDialogContext.Done` を呼び出す | `DialogContext.EndDialogAsync` を呼び出す |
| ユーザーの入力を取得する | `IAwaitable<IMessageActivity>` パラメーターを使用する | ウォーターフォール内からプロンプトを使用するか、`ITurnContext.Activity` を使用する |

v4 コードに関する注意事項を次に示します。

- 現在のターン コンテキストを取得するには、`DialogContext.Context` プロパティを使用します。
- ウォーターフォール ステップには、`DialogContext` から派生した `WaterfallStepContext` パラメーターがあります。
- 具象ダイアログ クラスおよびプロンプト クラスはすべて抽象 `Dialog` クラスから派生します。
- コンポーネント ダイアログを作成するときに ID を割り当てます。 ダイアログ セットの各ダイアログには、そのセット内で一意の ID を割り当てる必要があります。
- 変換後のダイアログを v3 に近いものにするために、拡張メソッド `PostAsync` と `Done` を実装します。 より多くの変換プロセスを軽減するために、拡張メソッドをさらに追加することも、移行後のコードを v4 に近いものにしておくために、これをスキップすることもできます。

### <a name="update-the-root-dialog"></a>ルート ダイアログを更新する

このボットでは、ルート ダイアログがユーザーに一連のオプションからの選択を要求し、その選択に基づいて子ダイアログを開始します。 会話の有効期間中、これがループします。

- メイン フローはウォーターフォール ダイアログとして設定できます。これは、v4 SDK の新しい概念です。 固定された一連のステップが順番に実行されます。 詳細については、「[連続して行われる会話フローの実装](/articles/v4sdk/bot-builder-dialog-manage-conversation-flow)」をご覧ください。
- プロンプトは、プロンプト クラスで処理されるようになりました。プロンプト クラスは、入力を要求し、最小限の処理と検証を実行して値を返す短い子ダイアログです。 詳細については、[ダイアログ プロンプトを使用したユーザー入力の収集](/articles/v4sdk/bot-builder-prompts.md)に関するページをご覧ください。

**Dialogs/RootDialog.cs** ファイルで、次の手順を実行します。

1. `using` ステートメントを更新します。
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. `HelpdeskOptions` オプションを文字列のリストから選択肢のリストに変換する必要があります。 これは選択プロンプトで使用されます。選択プロンプトは、(リスト内で) 選択された番号、選択された値、または選択肢のいずれかの同義語を有効な入力として受け入れます。
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. コンストラクターを追加します。 このコードは、次の処理を実行します。
   - ダイアログの各インスタンスの作成時に ID を割り当てます。 ダイアログ ID は、ダイアログの追加先となるダイアログ セットに含まれます。 ボットには、**MessageController** クラスで初期化されたダイアログ セットがあることを思い出してください。 各 `ComponentDialog` には、一連の独自のダイアログ ID が割り当てられた独自の内部ダイアログ セットがあります。
   - 選択プロンプトなど、他のダイアログを子ダイアログとして追加します。 ここでは、各ダイアログ ID にクラス名を使用します。
   - 3 ステップのウォーターフォール ダイアログを定義します。 このダイアログはこの後すぐに実装します。
     - ダイアログでは、まず、ユーザーに実行するタスクを選択するよう求めます。
     - 次に、その選択肢に関連付けられている子ダイアログを開始します。
     - 最後に、ダイアログ自体を再開します。
   - ウォーターフォールの各ステップはデリゲートです。次にそれらを実装します。可能であれば、元のダイアログから既存のコードを取得します。
   - コンポーネント ダイアログを開始すると、その _initial dialog_ が開始されます。 既定では、これはコンポーネント ダイアログに追加された最初の子ダイアログです。 初期ダイアログとして別の子を割り当てるには、コンポーネントの `InitialDialogId` プロパティを手動で設定します。
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. **StartAsync** メソッドは削除できます。 コンポーネント ダイアログが開始されると、"_初期_" ダイアログが自動的に開始されます。 ここでは、コンストラクターで定義したウォーターフォール ダイアログがこれに該当します。 このダイアログも、その最初のステップで自動的に開始されます。
1. **MessageReceivedAsync** メソッドと **ShowOptions** メソッドを削除し、それらをウォーターフォールの最初のステップに置き換えます。 この 2 つのメソッドは、ユーザーにあいさつし、利用可能なオプションのいずれかを選択するよう求めていました。
   - ここでは、選択リストが表示されます。あいさつメッセージとエラー メッセージは、選択プロンプトの呼び出しでオプションとして提供されます。
   - 選択プロンプトが完了すると、ウォーターフォールは次のステップに進むため、ダイアログで次に呼び出すメソッドを指定する必要はありません。
   - 選択プロンプトは、有効な入力を受け取るか、ダイアログ スタック全体がキャンセルされるまでループします。
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. **OnOptionSelected** をウォーターフォールの 2 番目のステップに置き換えることができます。 置換後も、ユーザーの入力に基づいて子ダイアログを開始します。
   - 選択プロンプトは `FoundChoice` 値を返します。 これは、ステップ コンテキストの `Result` プロパティに示されます。 ダイアログ スタックでは、すべての戻り値をオブジェクトとして扱います。 戻り値がダイアログのいずれかから返されている場合、そのオブジェクトがどの種類の値であるかがわかります。 各プロンプトの種類で返される値の一覧については、[プロンプトの種類](/articles/v4sdk/bot-builder-concept-dialog.md#prompt-types)に関するセクションをご覧ください。
   - 選択プロンプトは例外をスローしないため、try-catch ブロックを削除できます。
   - このメソッドが常に適切な値を返すように、フォールスルーを追加する必要があります。 このコードへの到達は回避する必要がありますが、到達しても、ダイアログを "正常に失敗にする" ことができます。
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. 最後に、古い **ResumeAfterOptionDialog** メソッドをウォーターフォールの最後のステップに置き換えます。
    - 元のダイアログで行っていたように、ダイアログを終了してチケット番号を返すのではなく、スタック上で元のインスタンスをそれ自体の新しいインスタンスに置き換えることでウォーターフォールを再開します。 これが可能なのは、元のアプリでは常に戻り値 (チケット番号) が無視され、ルート ダイアログが再開されていたためです。
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>アプリのインストール ダイアログを更新する

アプリのインストール ダイアログでは、いくつかの論理タスクを実行します。これを 4 ステップのウォーターフォール ダイアログとして設定します。 既存のコードをウォーターフォール ステップにどのように組み込むかは、ダイアログごとの論理的な作業です。 ステップごとに、そのコードが含まれていた元のメソッドに留意します。

1. ユーザーに検索文字列を要求します。
1. 一致候補をデータベースに照会します。
   - 一致候補が 1 つの場合は、これを選択して続行します。
   - 一致候補が複数ある場合は、ユーザーにいずれかを選択するよう求めます。
   - 一致候補がない場合、ダイアログは終了します。
1. ユーザーにアプリをインストールするコンピューター名の入力を求めます。
1. 情報をデータベースに書き込み、確認メッセージを送信します。

**Dialogs/InstallAppDialog.cs** ファイルで、次の手順を実行します。

1. `using` ステートメントを更新します。
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. プロンプトとダイアログに使用する ID の定数を定義します。 これにより、使用する文字列が一箇所で定義されるため、ダイアログ コードを管理しやすくなります。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. ダイアログの状態の追跡に使用するキーの定数を定義します。
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。 今回は、`InitialDialogId` プロパティを明示的に設定します。つまり、メインのウォーターフォール ダイアログが、セットに追加する最初のダイアログである必要はありません。 たとえば、最初にプロンプトを追加する方がよい場合は、実行時の問題を引き起こさずに、これを行うことが可能になります。
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。
    - 状態を自分で管理する必要があるので、ダイアログの状態でアプリのインストール オブジェクトを追跡します。
    - ユーザーに入力を求めるメッセージは、プロンプトの呼び出しでのオプションになります。
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. **appNameAsync** と **multipleAppsAsync** をウォーターフォールの 2 番目のステップに置き換えることができます。
    - ユーザーの最後のメッセージを単に確認するのではなく、プロンプトの結果を取得します。
    - データベース クエリと if ステートメントは、**appNameAsync** の場合と同様に構成されています。 if ステートメントの各ブロック内のコードは、v4 ダイアログで動作するように更新されました。
        - 一致候補が 1 つの場合は、ダイアログの状態を更新し、次のステップに進みます。
        - 一致候補が複数ある場合は、選択プロンプトを使用して、オプションの一覧から選択するようユーザーに求めます。 つまり、**multipleAppsAsync** は削除できます。
        - 一致候補がない場合は、このダイアログを終了し、ルート ダイアログに null を返します。
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. **appNameAsync** では、クエリの解決後にユーザーにコンピューター名の入力も求めていました。 ウォーターフォールの次のステップに、ロジックのこの部分を取り込みます。
    - 繰り返しますが、v4 では状態を自分で管理する必要があります。 ここで唯一注意が必要なことは、前のステップで 2 つの異なるロジック分岐を通過してこのステップに到達できることです。
    - 今回は別のオプションを指定して、以前と同じテキスト プロンプトを使ってユーザーにコンピューター名の入力を求めます。
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. **machineNameAsync** のロジックは、ウォーターフォールの最後のステップにまとめられています。
    - テキスト プロンプトの結果からコンピューター名を取得し、ダイアログの状態を更新します。
    - サポート コードは別のプロジェクトにあるため、データベースを更新する呼び出しを削除します。
    - その後、成功メッセージをユーザーに送信し、ダイアログを終了します。
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. データベースをシミュレートするために、データベースではなく静的リストを照会するように **getAppsAsync** を更新しました。
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>ローカル管理者ダイアログを更新する

v3 では、このダイアログでユーザーにあいさつし、FormFlow ダイアログを開始して、結果をデータベースに保存していました。 これは 2 ステップのウォーターフォールに簡単に変換できます。

1. `using` ステートメントを更新します。 このダイアログには、v3 の FormFlow ダイアログが含まれています。 v4 では、FormFlow コミュニティ ライブラリを使用できます。
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 結果はダイアログの状態で提供されるので、`LocalAdmin` のインスタンス プロパティを削除できます。
1. ダイアログに使用する ID の定数を定義します。 コミュニティ ライブラリでは、作成されるダイアログ ID は、常にダイアログで生成されるクラスの名前に設定されます。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。 FormFlow ダイアログも同じ方法で作成されます。 コンストラクター内でコンポーネントのダイアログ セットに追加するだけです。
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。 コンストラクター内で FormFlow を既に作成しているので、他の 2 つのステートメントはこれに変換されます。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. **ResumeAfterLocalAdminFormDialog** をウォーターフォールの 2 番目のステップに置き換えることができます。 インスタンス プロパティからではなく、ステップ コンテキストから戻り値を取得する必要があります。
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. FormFlow でインスタンス プロパティを更新しない点を除き、**BuildLocalAdminForm** の変更はほとんどありません。
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>パスワードのリセット ダイアログを更新する

v3 では、このダイアログでユーザーにあいさつし、パス コードでユーザーを承認して、失敗にするかまたは FormFlow ダイアログを開始した後、パスワードをリセットしていました。 これもウォーターフォールに適切に変換できます。

1. `using` ステートメントを更新します。 このダイアログには、v3 の FormFlow ダイアログが含まれています。 v4 では、FormFlow コミュニティ ライブラリを使用できます。
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. ダイアログに使用する ID の定数を定義します。 コミュニティ ライブラリでは、作成されるダイアログ ID は、常にダイアログで生成されるクラスの名前に設定されます。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. コンストラクターを追加し、コンポーネントのダイアログ セットを初期化します。 FormFlow ダイアログも同じ方法で作成されます。 コンストラクター内でコンポーネントのダイアログ セットに追加するだけです。
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. **StartAsync** をウォーターフォールの最初のステップに置き換えることができます。 ここでは、コンストラクター内で FormFlow を既に作成しました。 それ以外は、v3 の呼び出しから v4 の対応する呼び出しへの変換だけを行い、同じロジックを維持します。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** は主に演習として残してあります。 元のコードはコメント アウトされています。このメソッドは true を返すだけです。 また、メール アドレスは、元のボットで使用されていなかったので削除することもできます。
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** に変更はありません。
1. **ResumeAfterLocalAdminFormDialog** をウォーターフォールの 2 番目のステップに置き換えることができます。戻り値はステップ コンテキストから取得します。 元のダイアログで何にも使用されていなかった電子メール アドレスを削除しました。また、データベースに照会する代わりにダミーの結果を用意しました。 v3 の呼び出しから v4 の対応する呼び出しへの変換だけを行い、同じロジックを維持します。
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>必要に応じてモデルを更新する

FormFlow ライブラリを参照する一部のモデルで、`using` ステートメントを更新する必要があります。

1. `LocalAdminPrompt` で、これらを次のように変更します。
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. `ResetPasswordPrompt` で、これらを次のように変更します。
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>Web.config の更新

**MicrosoftAppId** と **MicrosoftAppPassword** の構成キーをコメント アウトします。 これにより、これらの値をエミュレーターに提供しなくても、ローカルでボットをデバッグできるようになります。

## <a name="run-and-test-your-bot-in-the-emulator"></a>ボットを実行してエミュレーターでテストする

この時点で、ボットをローカルの IIS で実行し、エミュレーターを使用してボットに接続できます。

1. IIS でボットを実行します。
1. エミュレーターを起動し、ボットのエンドポイント (例: **http://localhost:3978/api/messages**) に接続します。
    - 初めてボットを実行する場合は、**[File]\(ファイル\) > [New Bot]\(新しいボット\)** をクリックして、画面の指示に従います。 それ以外の場合は、**[File]\(ファイル\) > [Open Bot]\(ボットを開く\)** をクリックして既存のボットを開きます。
    - 構成のポート設定を再確認します。 たとえば、ブラウザーで `http://localhost:3979/` にアクセスしてボットを開いている場合は、エミュレーターでボットのエンドポイントを `http://localhost:3979/api/messages` に設定します。
1. 4 つのダイアログはすべて機能するはずです。ウォーターフォール ステップにブレークポイントを設定して、これらのポイントでダイアログ コンテキストとダイアログの状態を確認できます。

## <a name="additional-resources"></a>その他のリソース

v4 の概念トピック

- [ボットのしくみ](../bot-builder-basics.md)
- [状態の管理](../bot-builder-concept-state.md)
- [ダイアログ ライブラリ](../bot-builder-concept-dialog.md)

v4 の使用方法のトピック

- [テキスト メッセージを送受信する](../bot-builder-howto-send-messages.md)
- [ユーザーと会話データを保存する](../bot-builder-howto-v4-state.md)
- [連続して行われる会話フローの実装](../bot-builder-dialog-manage-conversation-flow.md)
- [ダイアログ プロンプトを使用してユーザー入力を収集する](../bot-builder-prompts.md)

<!-- TODO:
- The conceptual piece
- The migration to a .NET Core project
-->
