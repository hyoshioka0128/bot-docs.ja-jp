---
title: ボットのトラブルシューティング - Bot Service
description: よく寄せられる技術的な質問を使用して、ボット開発における一般的な問題のトラブルシューティングを行います。
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: 5a4cf92cf7d51cf6d871be8ab632e7e2b6b30303
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80647599"
---
# <a name="troubleshooting-general-problems"></a>一般的な問題のトラブルシューティング
以下のよく寄せられる質問は、一般的なボット開発や運用上の問題のトラブルシューティングに役立ちます。

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>ボットに関する問題をトラブルシューティングするにはどうすればよいですか。

1. [Visual Studio Code](debug-bots-locally-vscode.md) または [Visual Studio](https://docs.microsoft.com/visualstudio/debugger/navigating-through-code-with-the-debugger?view=vs-2017) を使用して、ご利用のボットのソース コードをデバッグします。
1. ボットをクラウドにデプロイする前に、[エミュレーター](bot-service-debug-emulator.md)を使用してテストします。
1. Azure などのクラウド ホスティング プラットフォームにボットをデプロイし、<a href="https://portal.azure.com" target="_blank">Azure portal</a> のボットのダッシュボードで組み込み Web チャット コントロールを使用して、ボットへの接続をテストします。 Azure にデプロイした後でボットに関する問題が発生した場合は、次のブログ記事を参考にしてください:「[Understanding Azure troubleshooting and support (Azure のトラブルシューティングとサポートについて理解する)](https://azure.microsoft.com/blog/understanding-azure-troubleshooting-and-support/)」。
1. [認証][TroubleshootingAuth]を、起こり得る問題から除外します。
1. ボットで使用する予定の Web チャット、Teams、またはその他のチャンネルでボットをテストします。 これは、エンド ツー エンド ユーザー エクスペリエンスを確認するのに役立ちます。
1. ダイレクト ラインや Web チャットなど、追加の認証要件があるチャネルでのボットのテストを検討してください。
1. [ボットをデバッグする](bot-service-debug-bot.md)方法に関する記事、およびそのセクションに示されているデバッグに関するその他の記事をご覧ください。

## <a name="how-can-i-troubleshoot-authentication-issues"></a>認証に関する問題をトラブルシューティングするにはどうすればよいですか。

ボットの認証に関する問題のトラブルシューティングについて詳しくは、Bot Framework 認証の[トラブルシューティング][TroubleshootingAuth]に関する記事をご覧ください。

## <a name="im-using-the-bot-framework-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>Bot Framework SDK for .NET を使用しています。 ボットに関する問題をトラブルシューティングするにはどうすればよいですか。

**例外を探します。**  
Visual Studio 2019 で、 **[デバッグ]**  >  **[Windows]**  >  **[例外設定]** の順に移動します。 **[例外設定]** ウィンドウで、 **[Common Language Runtime Exceptions]\(共通言語ランタイム例外\)** の横にある **[スローされたときに中断]** チェック ボックスをオンにします。 スローされた例外やハンドルされない例外がある場合にも、出力ウィンドウに診断の出力が表示されることがあります。

**呼び出し履歴を確認します。**  
Visual Studio で、[マイ コードのみ](https://msdn.microsoft.com/library/dn457346.aspx)をデバッグするかどうかを選択できます。 完全な呼び出し履歴を調べることで、問題に関する詳細な分析情報が得られる場合があります。

**すべてのダイアログ メソッドの最後が、次のメッセージを処理するプランであることを確認します。**  
すべてのダイアログ ステップは、ウォーターフォールの次のステップにフィードされるか、現在のダイアログを終了してスタックから取り出す必要があります。 ステップが正しく処理されない場合、会話は期待どおりに続きません。 ダイアログの詳細については、[ダイアログ](v4sdk/bot-builder-concept-dialog.md)の概念に関する記事を参照してください。


## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>HTTP 状態コード 429 "要求が多すぎます" のエラーの原因は何ですか。

HTTP 状態コード 429 のエラー応答は、一定時間に発行された要求が多すぎることを示します。 応答の本文には問題の説明が含まれています。また、要求間の最低限必要な間隔が示されている場合があります。 このエラーの発生源の 1 つとして、[ngrok](https://ngrok.com/) ツールが考えられます。 無料プランを利用しており、ngrok の制限に達している場合は、Web サイトの料金と制限のページに移動して、他の[オプション](https://ngrok.com/product#pricing)を確認してください。 

## <a name="why-arent-my-bot-messages-getting-received-by-the-user"></a>ボット メッセージがユーザーによって受信されないのはなぜですか。

応答で生成されるメッセージ アクティビティは正しくアドレス指定されている必要があります。そうでないと、意図した宛先にメッセージが届きません。 ほとんどの場合、これを明示的に処理する必要はありません。メッセージ アクティビティのアドレス指定は SDK によって処理されます。 

アクティビティの正しいアドレス指定とは、適切な "*会話 ID*" の詳細と共に、送信者に関する詳細を含めることを意味します。 ほとんどの場合、メッセージ アクティビティは到着したものに応答して送信されます。 そのため、アドレス指定の詳細は、インバウンド アクティビティから取得できます。 

トレースまたは監査ログを調べると、メッセージが正しくアドレス指定されているかどうかを確認できます。 メッセージが正しくアドレス指定されていない場合は、ボットにブレークポイントを設定し、メッセージに対して ID が設定される場所を確認します。

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>ASP.NET でバックグラウンド タスクを実行するにはどうすればよいですか。 

場合によっては、非同期タスクを開始することができます。このタスクでは数秒間待機してから、コードが実行されてユーザー プロファイルがクリアされるか、会話やダイアログの状態がリセットされます。 これを行う方法の詳細については、「[How to run Background Tasks in ASP.NET](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx)」 (ASP.NET でバックグラウンド タスクを実行する方法) を参照してください。 具体的には、[HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/library/dn636893(v=vs.110).aspx) の使用を検討します。 

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>使用しているボットで最初に受信されたメッセージへの応答に時間がかかります。 時間を短縮するにはどうすればよいですか。

ボットは Web サービスであり、Azure を含む一部のホスティング プラットフォームでは、サービスで一定時間にトラフィックが受信されない場合、そのサービスは自動的にスリープ状態になります。 ご利用のボットでこのようになった場合は、次回のメッセージの受信時に最初からやり直す必要があり、その応答には、既に実行されている場合よりもかなり時間がかかります。

一部のホスティング プラットフォームでは、スリープ状態にならないようにご利用のサービスを構成することができます。 ボットが Azure Bot Service Web Apps でホストされている場合は、[Azure Portal](https://portal.azure.com) でご利用のボットのサービスに移動し、 **[アプリケーション設定]** 、 **[常時接続]** の順に選択します。 このオプションは、すべてではなく、ほとんどのサービス プランで利用できます。

## <a name="how-can-i-guarantee-message-delivery-order"></a>メッセージの配信順序を保証するにはどうすればよいですか。

Bot Framework では、メッセージの順序が可能な限り保持されます。 たとえば、メッセージ A を送信し、その HTTP 操作の完了を待機してから、メッセージ B を送信する別の HTTP 操作を開始する場合などです。SMS や電子メールなどの一部のチャンネルでは、ユーザーのデバイスに対する順序は保証されません。 

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>メッセージ テキストの部分がドロップされるのはなぜですか。

Bot Framework や多くのチャネルでは、[Markdown](https://en.wikipedia.org/wiki/Markdown) での書式設定の場合と同じように、テキストが解釈されます。 Markdown 構文として解釈される可能性のある文字が、ご利用のテキストに含まれているかどうかを確認してください。

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>同じボット サービス エンドポイントで複数のボットをサポートするにはどうすればよいですか。 

この[サンプル](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334)に、正しい `MicrosoftAppCredentials` で `Conversation.Container` を構成し、シンプルな `MultiCredentialProvider` を使用して複数のアプリケーション ID とパスワードを認証する方法が示されています。

## <a name="identifiers"></a>識別子

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>Bot Framework では識別子はどのように動作しますか。

Bot Framework の識別子について詳しくは、Bot Framework の[識別子に関するガイド][BotFrameworkIDGuide]をご覧ください。

## <a name="how-can-i-get-access-to-the-user-id"></a>ユーザー ID にアクセスするにはどうすればよいですか。

Bot Framework チャンネルでは、ユーザーが送信した Activity の `from.Id` フィールドにユーザーの ID が示されます。 SMS および電子メールのメッセージでは、このプロパティで生のユーザー ID が示されます。 一部のチャンネルでは `from.Id` プロパティを隠すため、そのチャンネルのユーザー ID とは異なる、ユーザーに対して一意の ID が含まれます。 既存のアカウントに接続する必要がある場合は、サインイン カードを使用して独自の OAuth フローを実装し、ユーザー ID を独自のサービスのユーザー ID に接続できます。

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>自分の Facebook のユーザー名が表示されなくなったのはなぜですか。

ご利用の Facebook のパスワードを変更しましたか。 変更すると、アクセス トークンが無効になり、<a href="https://portal.azure.com" target="_blank">Azure portal</a> で Facebook Messenger チャネル用のボットの構成設定の更新が必要になります。

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>Kik ボットから、"申し訳ありません。現在、会話することはできません" という応答が返されるのはなぜですか。

Kik の開発中のボットは 50 人のサブスクライバーが利用できます。 50 人の一意のユーザーがご利用のボットと対話した後、そのボットとチャットしようとしている新しいユーザーは、"申し訳ありません。現在、会話することはできません" というメッセージを受け取ります。 詳細については、[Kik のドキュメント](https://botsupport.kik.com/hc/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-)を参照してください。

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>自分のボットから認証済みのサービスを使用するにはどうすればよいですか。

Azure Active Directory の認証については、認証の追加に関する記事 ([V3](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0&tabs=csharp) | [V4](https://docs.microsoft.com/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-4.0&tabs=csharp)) をご覧ください。 

> [!NOTE] 
> 認証およびセキュリティ機能をご自分のボットに追加する場合は、コードで実装するパターンが、ご利用のアプリケーションに適したセキュリティ標準に準拠していることを確認する必要があります。

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>ユーザーの所定の一覧に自分のボットへのアクセスを制限するにはどうすればよいですか。

SMS や電子メールなどの一部のチャネルでは、対象範囲外のアドレスが提供されます。 このような場合は、ユーザーからのメッセージに `from.Id` プロパティの生のユーザー ID が含まれます。

Facebook、Slack などの他のチャネルでは、ボットで事前にユーザーの ID を予測できない方法で、対象範囲内のアドレスまたはテナント アドレスが提供されます。 このような場合は、ログイン リンクまたは共有シークレットを使用してユーザーを認証し、ボットを使用する権限があるかどうかを判断する必要があります。

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>ダイレクト ライン 1.1 の会話がすべてのメッセージの後、やり直されるのはなぜですか。

> [!NOTE]
> 
> このセクションは、最新バージョンの Direct Line プロトコル3.0 には適用されません 

ダイレクト ラインの会話がすべてのメッセージの後、やり直される場合、ダイレクト ライン クライアントによってボットに送信されたメッセージの `from` プロパティが欠落しているか `null` である可能性があります。 ダイレクト ライン クライアントによって、`from` プロパティが欠落しているか `null` であるメッセージが送信されると、ダイレクト ライン サービスで自動的に ID が割り当てられるため、クライアントから送信されたすべてのメッセージは、新しい別のユーザーから送信されたものと見なされます。

これを解決するには、ダイレクト ライン クライアントによって送信される各メッセージの `from` プロパティを、メッセージを送信しているユーザーを一意に表す安定した値に設定します。 たとえば、ユーザーが既に Web ページまたはアプリにサインインしている場合、ユーザーが送信したメッセージの `from` プロパティの値として、その既存のユーザー ID を使用できます。 あるいは、ページの読み込み時またはアプリケーションの読み込み時にランダムなユーザー ID を生成し、その ID を Cookike またはデバイス状態で格納し、ユーザーが送信したメッセージの `from` プロパティの値として、その ID を使用することもできます。

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>ダイレクト ライン 3.0 サービスから HTTP 状態コード 502 "ゲートウェイが不適切です" という応答が返される原因は何ですか。
ご利用のボットに接続しようとしたときに、要求が正常に完了しない場合は、ダイレクト ライン 3.0 から HTTP 状態コード 502 が返されます。 このエラーは、ボットからエラーが返されたか、要求がタイムアウトになったことを示します。ご利用のボットで生成されたエラーの詳細については、<a href="https://portal.azure.com" target="_blank">Azure portal</a> 内のボットのダッシュボードに移動して、影響を受けるチャネルに関する "問題" リンクをクリックしてください。 ご利用のボットに対して Application Insights が構成されている場合は、そこで詳細なエラー情報を見つけることもできます。 

::: moniker range="azure-bot-service-3.0"

## <a name="where-is-conversation-state-stored"></a>会話状態はどこに格納されるのですか。

ユーザー、会話、プライベート会話プロパティ バッグのデータは、コネクタの `IBotState` インターフェイスを使用して格納されます。 各プロパティ バッグは、ボットの ID で範囲指定されます。 ユーザー プロパティ バッグはユーザー ID で、会話プロパティ バッグは会話 ID でキー付けされます。プライベート会話プロパティ バッグはユーザー ID と会話 ID の両方でキー付けされます。 

Bot Framework SDK for .NET または Bot Framework SDK for Node.js を使用してボットを作成する場合、ダイアログ スタックとダイアログ データの両方が、プライベート会話プロパティ バッグにエントリとして自動的に格納されます。 C# 実装ではバイナリ シリアル化が使用され、Node.js 実装では JSON シリアル化が使用されます。

ご利用のデータ センター内にこのデータを格納する場合は、状態サービスのカスタム実装を提供できます。 これは 2 つ以上の方法で行うことができます。

* botbuilder-azure パッケージを使用します。
* REST レイヤーを使用して、カスタム `IBotState` サービスを提供します。
* 言語 (C#、JavaScript、または Python) 層で Builder インターフェイスを使用します。

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>HTTP 状態コード 412 "前提条件が満たされていません" または HTTP 状態コード 409 "競合" のエラーの原因は何ですか。

コネクタの `IBotState` サービスを使用して、ボット データ バッグ (ユーザー ボット データ バッグや会話ボット データ バッグなど。また、プライベート ボット データ バッグにダイアログ スタックの "制御フロー" 状態が含まれている場合は、プライベート ボット データ バッグ) が格納されます。 `IBotState` サービスでのコンカレンシー制御は、ETag を使用してオプティミスティック コンカレンシーによって管理されます。 "読み取り/変更/書き込み" シーケンス時に (単一ボット データ バッグへの同時更新により) 更新の競合が発生した場合は、次のようになります。

* ETag が保持されている場合、HTTP 状態コード 412 "前提条件が満たされていません" のエラーは `IBotState` サービスからスローされます。 これは、Bot Framework SDK for .NET での既定の動作です。
* Etag が保持されていない (つまり、ETag が `\*`に設定されている) 場合、"最後の書き込みが有効" ポリシーが有効になり、"前提条件が満たされていません" エラーを防ぐことはできますが、データ損失のリスクがあります。 これは、Bot Framework SDK for Node.js での既定の動作です。

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>"前提条件が満たされていません" (412) エラーや "競合" (409) エラーを解決するにはどうすればよいですか。

これらのエラーは、ご利用のボットで同じ会話に対する複数のメッセージが一度に処理されていることを示します。 正確に順序付けされたメッセージを必要とするサービスにご利用のボットが接続されている場合は、メッセージが並列処理されないように、会話状態のロックを検討する必要があります。 

Bot Framework SDK for .NET では、メモリ内セマフォで単一の会話の処理をペシミスティックにシリアル化するためのメカニズム (`IScope` を実装するクラス `LocalMutualExclusion`) が提供されます。 会話アドレスで範囲指定された、Redis リースを使用するようにこの実装を拡張できます。

ご利用のボットが外部サービスに接続されていない場合、または同じ会話からのメッセージの並列処理が受け入れられる場合は、このコードを追加して、Bot State API で発生する競合をすべて無視することができます。 これにより、最後の返信で会話状態を設定することができます。

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>State API を使用して格納されたボット データはどのようにバージョン管理するのですか。

> [!IMPORTANT]
> Bot Framework State Service API を運用環境や v4 ボットで使用することは推奨されていません。将来のリリースで完全に非推奨となる可能性があります。 テストが目的の場合はメモリ内ストレージを使用するようにボット コードを更新し、運用ボットの場合はいずれかの **Azure 拡張機能**を使用することをお勧めします。 詳しくは、「[状態データの管理](v4sdk/bot-builder-howto-v4-state.md)」のトピックをご覧ください。

State サービスでは、会話のダイアログの進行状況を保持することができるため、ユーザーは後でボットでの会話に戻ることができます。位置がわからなくなることはありません。 これを保持するため、ボットのコードの変更時に、State API を使用して格納されたボット データ プロパティ バッグが自動的にクリアになることはありません。 変更されたコードが以前のバージョンのコードと互換性があるかどうかに基づいて、ボット データをクリアするかどうかを決定する必要があります。 

* ご利用のボットの開発時に会話のダイアログ スタックと状態を手動でリセットする場合は、`/deleteprofile` コマンドを使用して状態データを削除することができます。 必ず、このコマンドの先頭にスペースを挿入し、チャネルで解釈されないようにしてください。
* ご利用のボットが運用環境にデプロイされたら、ボット データのバージョンを管理し、バージョンをバンプする場合に、関連付けられている状態データがクリアされるようにすることができます。 Bot Framework SDK for Node.js では、ミドルウェアを使用して、このようにすることができます。Bot Framework SDK for .NET では、`IPostToBot` 実装を使用して行うことができます。

> [!NOTE]
> シリアル化形式の変更により、あるいはコードの変更が多すぎるために、ダイアログ スタックが正しく逆シリアル化できない場合は、会話状態がリセットされます。

::: moniker-end

## <a name="why-do-i-get-an-authorization_requestdenied-exception-when-creating-a-bot"></a>ボットを作成するときに Authorization_RequestDenied という例外が発生するのはなぜですか。

Azure Bot Service ボットを作成するためのアクセス許可は、Azure Active Directory (AAD) ポータルを通じて管理されます。 [AAD ポータル](https://aad.portal.azure.com)でアクセス許可が適切に構成されていない場合、ユーザーがボット サービスを作成しようとしたときに **Authorization_RequestDenied** 例外が発生します。

まず、ディレクトリの "ゲスト" であるかどうかを確認します。

1. [Azure Portal](https://portal.azure.com) にサインインします。
2. **[すべてのサービス]** をクリックし、*active* を検索します。
3. **[Azure Active Directory]** を選択します。
4. **[ユーザー]** をクリックします。
5. 一覧からユーザーを見つけて、**ユーザーの種類**が**ゲスト**でないことを確認します。

![Azure Active Directory のユーザーの種類](~/media/azure-active-directory/user_type.png)

**ゲスト**でないことを確認したら、アクティブ ディレクトリ内のユーザーがボット サービスを作成できることを確認するために、ディレクトリ管理者が次の設定を構成する必要があります。

1. [AAD ポータル](https://aad.portal.azure.com)にサインインします。 **[ユーザーとグループ]** に移動して、 **[ユーザー設定]** を選択します。
2. **[アプリの登録]** セクションで、 **[ユーザーはアプリケーションを登録できる]** を **[はい]** に設定します。 これで、ディレクトリ内のユーザーがボット サービスを作成できるようになります。
3. **[外部ユーザー]** セクションで、 **[ゲストのアクセス許可を制限する]** を **[いいえ]** に設定します。 これで、ディレクトリ内のゲスト ユーザーがボット サービスを作成できるようになります。

![Azure Active Directory 管理センター](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>自分のボットを移行できないのはなぜですか。

dev.botframework.com に登録されているボットを Azure に移行する必要があるが、ボットの移行で問題が発生する場合は、ボットが既定のディレクトリ以外のディレクトリに属していることが原因である可能性があります。 以下の手順を試してみてください。

1. ターゲット ディレクトリから、既定のディレクトリのメンバーではない新しいユーザーを (メール アドレスを使用して) 追加し、移行のターゲットとなるサブスクリプションに対する共同作成者ロールをそのユーザーに付与します。

2. [開発者ポータル](https://dev.botframework.com)から、移行する必要があるボットの共同所有者として、ユーザーのメール アドレスを追加します。 その後、サインアウトします。

3. 新しいユーザーとして[開発者ポータル](https://dev.botframework.com)にサインインして、ボットの移行に進みます。

## <a name="where-can-i-get-more-help"></a>詳細情報はどこで入手できますか。

* [Stack Overflow](https://stackoverflow.com/questions/tagged/botframework) で既に回答済みの質問の情報を利用するか、`botframework` タグを使用してご自分の質問を投稿します。 Stack Overflow には、わかりやすいタイトル、完全かつ簡潔な問題の説明、問題を再現するための十分な詳細情報が必要などのガイドラインがあることに注意してください。 機能要求や非常に広範な質問はトピックから外れています。新しいユーザーは、[Stack Overflow ヘルプ センター](https://stackoverflow.com/help/how-to-ask)にアクセスして詳細を確認する必要があります。
* Bot Framework SDK での既知の問題に関する情報を確認する場合や、新しい問題を報告する場合は、GitHub の [BotBuilder に関する問題](https://github.com/Microsoft/BotBuilder/issues)を参照してください。
* [Gitter](https://gitter.im/Microsoft/BotBuilder) の BotBuilder コミュニティ ディスカッションの情報を活用します。



[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md

