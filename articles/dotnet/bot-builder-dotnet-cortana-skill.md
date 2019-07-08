---
title: .NET を使用して Cortana スキルを構築する | Microsoft Docs
description: Bot Framework SDK for .NET で Cortana スキルを構築するための主要概念について説明します。
keywords: Bot Framework, Cortana スキル, 音声, .NET, SDK, 主な概念, 主要概念
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9dd84e9e5e39e1e1b801e08fbee101dbfa8b0c49
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405680"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Cortana スキルを使用した音声認識ボットの作成

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-cortana-skill.md)


Bot Framework SDK for .NET で音声認識ボットを作成できます。そのためには、これを Cortana スキルとして Cortana チャネルに接続します。 


> [!TIP]
> スキルとは何か、何ができるかについては、「[Cortana Skills Kit][CortanaGetStarted]」 (Cortana スキル キット) を参照してください。

Bot Framework を使用して Cortana スキルを作成する場合、Cortana 固有の知識はほとんど必要なく、ボットの作成が主な部分を構成します。 これまでに他のボットを作成したことがあるかもしれませんが、主な相違点の 1 つは、Cortana には視覚と音声の両方のコンポーネントがあることです。 視覚コンポーネントとしては、Cortana にはカードなどのコンテンツを表示するためのキャンバス領域があります。 音声コンポーネントとしては、ボットのメッセージにテキストまたは SSML を指定すると、それが Cortana によってユーザーに向けて読み上げられるので、ボットが話せるようになります。 

> [!NOTE]
> Cortana は多種多様なデバイスで利用できます。 画面があるものもあれば、スタンドアロン スピーカーのように画面がないものもあります。 どちらのシナリオにも対応できるようにボットを作成する必要があります。 デバイス情報を確認する方法については、[Cortana 固有のエンティティ][CortanaSpecificEntities]に関するページを参照してください。

## <a name="adding-speech-to-your-bot"></a>ボットに音声を追加する

ボットからの音声メッセージは、音声合成マークアップ言語 (SSML) として表されます。 Bot Framework SDK を使用すると、ボットが表示する内容だけでなく、話す内容も制御できるよう、応答に SSML を含めることができます。  また、ボットがユーザーの入力を受け入れるか、期待しているか、無視しているかを指定して、Cortana のマイクの状態を制御することもできます。

`IMessageActivity` オブジェクトの `Speak` プロパティを設定して、Cortana が話すメッセージを指定します。 プレーン テキストを指定すると、Cortana はその単語の発音方法を決定します。 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

ピッチ、トーン、強調を細かく制御するには、`Speak` プロパティを [音声合成マークアップ言語 (SSML)](http://www.w3.org/TR/speech-synthesis/) 形式で設定します。  

次のコード例では、"text" という単語を中くらいの強調で話すことを指定しています。
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


**InputHint** プロパティは、ボットが入力を期待しているかどうかを Cortana に示すために役立ちます。 既定値は、プロンプトの場合は **ExpectingInput**、他の種類の応答の場合は **AcceptingInput** です。


| 値 | 説明 |
|------|------|
| **AcceptingInput** | ボットは受動的に入力を受け入れる準備ができていますが、応答を待機しているわけではありません。 ユーザーがマイク ボタンを押したままにすると、Cortana はユーザーからの入力を受け入れます。|
| **ExpectingInput** | ボットがユーザーからの応答をアクティブに必要としていることを示します。 Cortana はユーザーがマイクに話すのをリッスンします。  |
| **IgnoringInput** | Cortana は入力を無視します。 ボットが要求をアクティブに処理しており、要求が完了するまでユーザーの入力を無視する場合に、このヒントを送信できます。  |

<!-- TODO: tip about time limit and batching -->

以下の例は、ユーザー入力が必要であることを Cortana に知らせる方法を示しています。 マイクは、起動した状態のままになります。
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>Cortana でカードを表示する

音声応答に加え、Cortana ではカードの添付ファイルも表示できます。 Cortana は、次のリッチ カードをサポートしています。

| カードの種類 | 説明 |
|----|----|
| [HeroCard][heroCard] | 通常 1 つの大きなイメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。 |
| [ThumbnailCard][thumbnailCard] | 通常 1 つのサムネイル イメージ、1 つまたは複数のボタン、およびテキストが含まれるカード。 |
| [ReceiptCard][receiptCard] | ボットからユーザーに領収書を提供できるようにするカード。 通常は、領収書に含める項目の一覧、税金と合計の情報、およびその他のテキストが含まれます。 |
| [SignInCard][signinCard] | ボットでユーザーのサインインを要求できるようにするカード。 通常は、テキストと、ユーザーがクリックしてサインイン プロセスを開始できる 1 つ以上のボタンが含まれます。 |


これらのカードが Cortana の内部でどのように表示されるかについては、[カード設計のベスト プラクティス][CardDesign]に関するページを参照してください。 ボットでリッチ カードを使用する方法の例については、「[Add rich card attachments to messages](bot-builder-dotnet-add-rich-card-attachments.md)」(メッセージにリッチ カードの添付ファイルを追加する) を参照してください。 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>サンプル:RollerSkill
以下のセクションで取り上げるコードは、サイコロを振る Cortana のサンプル スキルのものです。 [BotBuilder-Samples リポジトリ](https://github.com/Microsoft/BotBuilder-Samples/)から、ボットのコード全体をダウンロードしてください。

その[呼び出し名][InvocationNameGuidelines] to Cortana. For the roller skill, after you [add the bot to the Cortana channel][CortanaChannel] を言ってスキルを呼び出し、それを Cortana スキルとして登録した後、Cortana に "Ask Roller" (Roller に頼んで) または "Ask Roller to roll dice" (サイコロを振るよう Roller に頼んで) と話しかけると、これを呼び出せます。

### <a name="explore-the-code"></a>コードを調べる



適切なダイアログを呼び出すには、`RootDispatchDialog.cs` で定義されているアクティビティ ハンドラーでユーザー入力を照合する正規表現を使用します。 たとえば、次の例のハンドラーは、ユーザーが「I'd like to roll some dice」(いくつかサイコロを振りたい) などと話した場合にトリガーされます。 類義語が正規表現に含まれているため、似た発話でダイアログがトリガーされます。
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

`CreateGameDialog` ダイアログでは、ボットでプレイするカスタム ゲームを設定します。 また、`PromptDialog` を使用して、何面のサイコロにするか、そしてサイコロをいくつ振るかをユーザーに尋ねます。 プロンプトの初期化に使用される `PromptOptions`オブジェクトには、音声バージョンのプロンプトの`speak` プロパティが含まれています。

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

`PlayGameDialog` は、`HeroCard` で表示し、さらに `Speak` メソッドを使用して話す音声メッセージを構築することで結果をレンダリングします。

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>次の手順

ボットをローカルで実行する場合、またはクラウドにデプロイする場合、ボットを Cortana から呼び出せます。 Cortana スキルを試すために必要な手順については、「[Test a Cortana skill](../bot-service-debug-cortana-skill.md)」(Cortana スキルのテスト) を参照してください。


## <a name="additional-resources"></a>その他のリソース
* [Cortana スキル キット][CortanaGetStarted]
* [メッセージへの音声の追加](bot-builder-dotnet-text-to-speech.md)
* [SSML リファレンス][SSMLRef]
* [Cortana の音声設計のベスト プラクティス][VoiceDesign]
* [Cortana のカード設計のベスト プラクティス][CardDesign]
* [Cortana デベロッパー センター][CortanaDevCenter]
* [Cortana のテストとデバッグのベスト プラクティス][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET リファレンス</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
[CortanaDevCenter]: https://developer.microsoft.com/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines  
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 


