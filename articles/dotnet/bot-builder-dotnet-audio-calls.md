---
title: Skype で音声通話を行う | Microsoft Docs
description: Bot Builder SDK for .NET を使用して、Skype で音声通話を行う方法について説明します。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dd40123ef641d3cef80525d466fd85ecab80654a
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904940"
---
# <a name="conduct-audio-calls-with-skype"></a>Skype で音声通話を行う

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

音声通話をサポートするボットのアーキテクチャは、一般的なボットのアーキテクチャとよく似ています。 次のコード サンプルは、Bot Builder SDK for .NET を使用して Skype での音声通話に対するサポートを有効にする方法を示しています。 

## <a name="enable-support-for-audio-calls"></a>音声通話のサポートの有効化

ボットが音声通話をサポートできるようにするには、`CallingController` を定義します。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> 音声通話をサポートする `CallingController` に加え、ボットにメッセージをサポートするための `MessagesController` を含めることもできます。 両方のオプションを提供することにより、ユーザーは好みの方法でボットとやり取りすることができます。 <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>電話に応答して

`ProcessIncomingCallAsync` タスクは、ユーザーが Skype からこのボットへの呼び出しを開始するたびに実行されます。
コンストラクターは、`incomingCallEvent` のための事前定義されたハンドラーがある `IVRBot` クラスを登録します。

ワークフロー内の最初のアクションは、ボットが着信に応答するか拒否するかを決定する必要があります。 このワークフローでは、ボットに着信に応答してからウェルカム メッセージを再生するように指示します。 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>ボットが応答した後

ボットが通話に応答すると、ワークフロー内で指定された後続のアクションが **Skype Bot Platform for Calling** にプロンプトの再生、音声の録音、音声の認識、ダイヤル パッドからの数字の収集を指示します。 ワークフローの最後のアクションは、通話を終了することです。 

このコード サンプルでは、ウェルカム メッセージの完了後、メニューを設定するハンドラーを定義します。

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

`CreateIvrOptions` メソッドは、ユーザーに提示されるメニューを定義します。

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

`RecognitionOption` クラスは、音声応答だけでなく対応するデュアルトーン多重周波数 (DTMF) バリエーションの両方を定義します。 DTMF により、ユーザーは話す代わりに、キーパッド上の対応する数字を入力して応答できます。

`OnRecognizeCompleted` メソッドはユーザーの選択を処理し、入力パラメーター `recognizeOutcomeEvent` にはユーザーの選択値が含まれます。

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>自然言語のサポート
自然言語の応答をサポートするようにボットを設計することもできます。 **Bing Speech API** は、ボットがユーザーの音声応答の単語を認識できるようにします。

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>サンプル コード

Bot Builder SDK for .NET を使用して Skype で音声通話をサポートする方法を示す完全なサンプルについては、GitHub の「<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot sample (Skype Calling Bot のサンプル)</a>」を参照してください。

## <a name="additional-resources"></a>その他のリソース

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET リファレンス</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot のサンプル (GitHub)</a>
