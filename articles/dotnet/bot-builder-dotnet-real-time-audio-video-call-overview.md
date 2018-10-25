---
title: Skype 用のリアルタイム メディア ボットのビルド | Microsoft Docs
description: Bot Builder SDK for .NET と Bot Builder-RealTimeMediaCalling SDK for .NET を使用し、Skype で音声/動画によるリアルタイム呼び出しを行うボットをビルドする方法について説明します。
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6ceeca9adc9cad9e60a73c1c7c91bea43b97fdd9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997930"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>Skype 用のリアルタイム メディア ボットのビルド

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

ボット用のリアルタイム メディア プラットフォームは、ボットがフレーム単位で音声/動画コンテンツを送受信することを可能にする高度な機能です。 ボットには、音声、動画、画面共有のリアルタイム モダリティに "raw" アクセスできます。 この記事では、音声/動画の呼び出しボットをビルドし、リアルタイム モダリティにアクセスすることの概要を示します。

この記事では、ボットは ASP.NET Web API フレームワークを自己ホストする Web ロールか worker ロールとして Azure Cloud Service で実行されます。

> [!IMPORTANT]
> この記事は序章的な内容になっています。サンプル リアルタイム メディア ボットの完全なコードについては、<a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder-RealTimeMediaCalling</a> GitHub リポジトリにある [Samples] フォルダーを参照してください。

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>リアルタイム メディア ボットをホストするサービスを構成する

リアルタイム メディア プラットフォームを使用するには、次のサービス構成が必要になります。

* ボットはそのサービスの完全修飾ドメイン名 (FQDN) を認識する必要があります。 これは Azure RoleEnvironment API によって提供されません。FQDN はボットの Cloud Service 構成に保管し、サービスの起動中に読み込む必要があります。

* ボット サービスには、一般に認められている証明書機関が発行した証明書を与える必要があります。 証明書の拇印をボットの Cloud Service 構成に保管し、サービスの起動中に読み込む必要があります。

* パブリックの<a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">インスタンス入力エンドポイント</a>をプロビジョニングする必要があります。 これによって、ボットのサービスの仮想マシン (VM) インスタンスごとに一意のパブリック ポートが割り当てられます。 このポートは、Skype 呼び出しクラウドと通信する目的でリアルタイム メディア プラットフォームによって使用されます。
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  呼び出し関連のコールバックや通知のために別のインスタンス入力エンドポイントを作成する際にも役立ちます。 インスタンス入力エンドポイントを使用することで、サービス デプロイに含まれる VM インスタンスのうち、入力エンドポイント呼び出しのためにリアルタイム メディア セッションをホストしているものと同じ VM インスタンスにコールバックと通知が届けられます。
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* 各 VM インスタンスには、インスタンスレベルのパブリック IP アドレス (ILPIP) を与える必要があります。 起動中、各サービス インスタンスに割り当てられている ILPIP をボットが検出する必要があります。 ILPIP を取得し、構成する方法については、<a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a>に関するページを参照してください。
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* サービス インスタンス起動中、スクリプト `MediaPlatformStartupScript.bat` (Nuget パッケージの一部として提供される) は昇格された特権でスタートアップ タスクとして実行する必要があります。 スクリプト実行は、プラットフォームの初期化メソッドが呼び出される前に完了する必要があります。 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>サービス起動時にメディア プラットフォームを初期化する

サービス インスタンスの起動中、リアルタイム メディア プラットフォームを初期化する必要があります。 ボットがそのインスタンスで音声/動画の Skype 呼び出しを受けるには、この初期化を 1 回だけ行う必要があります。 メディア プラットフォームを初期化するには、サービスの FQDN、パブリック ILPIP アドレス、インスタンス入力エンドポイント値、ボットの **Microsoft AppID** など、さまざまな構成を設定する必要があります。

> [!NOTE]
> 自分のボットの **AppID** と **AppPassword** を見つける方法については、「[MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」(MicrosoftAppID と MicrosoftAppPassword) を参照してください。

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>登録して着信要求を受け取る

`CallController` クラスを定義します。 これにより、ボット サービスに Skype 着信が登録され、コールバックと通知が適切な `RealTimeMediaCall` オブジェクトに転送されます。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` によって `IRealTimeMediaCallServiceSettings` が実装され、コールバック イベントと通知イベントの webhook リンクが提供されます。

## <a name="register-for-incoming-events-for-the-call"></a>呼び出しの受信イベントを登録する

`IRealTimeMediaCall` を実装する `RealTimeMediaCall` クラスを定義します。 ボットによって受信される呼び出しごとに、Bot Framework によって `RealTimeMediaCall` のインスタンスが作成されます。 コンストラクターに渡される `IRealTimeMediaCallService` オブジェクトにより、ボットにイベントが登録され、リアルタイム メディア呼び出しに関連付けられているイベントを処理できます。

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>音声/動画ソケットを作成する
ボットが Skype の音声/動画着信を受け取るには、送受信のリアルタイム メディアをサポートする目的で、`AudioSocket` オブジェクトと `VideoSocket` オブジェクトを作成する必要があります。 (ボットが動画に対応する必要がない場合、AudioSocket のみを作成してください。)

ボットでは、それがサポートするモダリティを先に決定し、適切な AudioSocket オブジェクトと VideoSocket オブジェクトを作成する必要があります。 ボットでは、着信を受け取った後、呼び出しに対してサポートしているモダリティを変更できません。

各 AudioSocket と各 VideoSocket に対して、ボットにより、メディア ソケットではメディアの送信と受信の両方をサポートするのか、送信または受信のみをサポートするのかが指定されます。 たとえば、ボットで音声を送受信 ("Sendrecv") するが、動画に関しては送信のみ ("Sendonly") を望むことがあります。 ボットではまた、各メディア ソケットでサポートされるメディア形式を指定する必要があります。 AudioSocket の場合、現在サポートされている形式は "Pcm16K": (符号付き) 16 ビット PCM エンコーディング、16KHz サンプリング レートです。 VideoSocket の場合、送受信のメディア形式が個別に指定されます。 動画の受信では "NV12" 形式のみがサポートされています。送信の場合、いくつかの形式がサポートされています。

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>MediaConfiguration を作成する
メディア ソケットが作成されると、ボットでは次に、メディア ソケットを Skype の音声/動画着信に関連付けるために必要な `MediaConfiguration` オブジェクトを作成する必要があります。

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>音声/動画の着信に応える
Skype の音声/動画着信に応えることをボットに許可する目的で `OnIncomingCallReceived` イベントが呼び出されます。 そのために、ボットによって `MediaConfiguration` オブジェクトを含む `AnswerAppHostedMedia` オブジェクトが作成されます。 ボットにより、この Skype 呼び出しに関連付けられている通知が登録されます。

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>呼び出しの結果
`AnswerAppHostedMedia` アクションが完了すると、`OnAnswerAppHostedMediaCompleted` が発生します。 `AnswerAppHostedMediaOutcomeEvent` の `Outcome` プロパティによって成功または失敗が示されます。 呼び出しを確立できない場合、ボットでは、呼び出しのために作成された AudioSocket オブジェクトと VideoSocket オブジェクトを破棄する必要があります。

## <a name="receive-audio-media"></a>音声メディアを受信する
音声を受信する機能を付けて `AudioSocket` が作成された場合、音声のフレームが受信されるたびに `AudioMediaReceived` イベントが呼び出されます。 音声コンテンツを調達している可能性があるピアに関係なく、ボットでは、毎秒約 50 回このイベントを処理することが予想されます (音声がピアから受信されない場合、コンフォート ノイズ バッファーがローカルで生成されるため)。 音声コンテンツの各パケットが `AudioMediaBuffer` オブジェクトで届けられます。 このオブジェクトには、復号された音声コンテンツを含む、ネイティブでヒープを割り当てたメモリ バッファーのポインターが含まれます。 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

イベント ハンドラーはすぐに戻る必要があります。 アプリケーションでは、非同期で処理するように `AudioMediaBuffer` を待ち行列に入れることをお勧めします。 `OnAudioMediaReceived` イベントはリアルタイム メディア プラットフォームによってシリアル化されます (つまり、現在のイベントが戻るまで次のイベントは発生しません)。 `AudioMediaBuffer` が使用されたら、基礎となっているアンマネージド メモリをメディア プラットフォームで回収できるように、アプリケーションでバッファーの Dispose メソッドを呼び出す必要があります。 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> バッファーへのアクセスが完了するまで、ボットでバッファーの Dispose メソッドを呼び出すことはできません。

## <a name="receive-video-media"></a>動画メディアを受信する
動画の受信は、毎秒バッファー数がフレーム レートの値に依存することを除き、音声の受信と同じです。 `VideoMediaBuffer` には `VideoFormat` プロパティと `OriginalVideoFormat` プロパティがあります。 `OriginalVideoFormat` は、調達されたときのバッファーの元の形式を表します。 `IVideoSocket.VideoMediaReceived` イベント ハンドラー経由で動画バッファーを受信したときにのみ利用できます。 送信前にバッファーのサイズが変更された場合、`OriginalVideoFormat` プロパティには元の Width と Height が与えられ、`VideoFormat` にはサイズ変更後の現在の Width と Height が与えられます。 `OriginalVideoFormat` の Width プロパティと Height プロパティが `VideoFormat` プロパティと異なる場合、`VideoMediaReceived` イベントで発生した `VideoMediaBuffer` のコンシューマーにより `OriginalVideoFormat` のサイズに合わせてバッファーのサイズが変更される必要があります。 現在のところ、受信には NV12 形式のみがサポートされています。

## <a name="send-audio-media"></a>オーディオ メディアの送信
メディアを送信するように `AudioSocket` が構成されている場合、ボットでは、ステータス変更の送信に関する通知を取得する目的で、`AudioSocket` で `AudioSendStatusChanged` イベント ハンドラーを登録する必要があります。 ボットで音声の送信を開始するのは、送信ステータスが Active に変わった後に限られます。

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

音声メディアを送信するには、ネイティブでヒープを割り当てたバッファー内に PCM 音声コンテンツが含まれていることが前提となります。 ボットの派生元は `AudioMediaBuffer` 抽象クラスにする必要があります。 メディアは AudioSocket の `Send` メソッドにより非同期で送信され、送信が完了したとき、プラットフォームによって `AudioMediaBuffer` で `Dispose` が呼び出されます。 `Send` が戻ったとき、ボットでアンマネージド リソースを解放しないでください (あるいは、ボットをプール アロケーターに戻さないでください)。 `Dispose` が呼び出されるのを待つ必要があります。

## <a name="send-video-media"></a>動画メディアを送信する
動画メディアの送信は音声メディアの送信と似ています。 ボットでは、`VideoSendStatusChanged` イベントを登録し、`MediaSendStatus` が `Active` になるのを待つ必要があります。 `Dispose` メソッドが呼び出されるまで、ボットでバッファーのアンマネージド リソースを解放したり、回収したりすることはできません。 RGB24、NV12、Yuy2 が色の形式としてサポートされています。

動画の送信では複数の `VideoFormat` に対応していますが、現在推奨されている `VideoFormat` では `VideoSendStatusChanged` イベント経由でボットとの通信が行われます。 動画の送信に推奨される `VideoFormat` は、ネットワークの帯域幅状態やその動画ウィンドウのサイズを変更するピア クライアントに起因して変わることがあります。

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

各 `VideoMediaBuffer` には、その特定のバッファーの動画コンテンツの形式を示す VideoFormat プロパティがあります。 `VideoFormat` プロパティは `VideoSendStatusChanged` イベントで指示される `PreferredVideoSourceFormat` プロパティに一致している必要はありませんが、動画フレームのサイズ変更で CPU サイクルが無駄に使用されることを回避するために、指示された `PreferredVideoSourceFormat` を使用することを強くお勧めします。

## <a name="call-notifications"></a>呼び出し通知
### <a name="call-state-changes"></a>呼び出し状態の変化
ボットでは、`RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` の `NotificationSubscriptions` で `NotificationType.CallStateChange` をサブスクライブすることで呼び出し状態の変化の通知を取得できます。

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>名簿の更新
ボットが会議に追加される場合、`RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` の `NotificationSubscriptions` で `NotificationType.RosterUpdate` をサブスクライブすることで会議の名簿を待ち受けることができます。 `RosterUpdateNotification` には、会議の全参加者の詳細が含まれます。 ボットでは、参加者が送信する動画を含む通知を待ったり、その `Subscribe` オブジェクトの 1 つで参加者の動画を `VideoSocket` したりできます。

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>通話を終わらせる

### <a name="handle-call-termination-from-the-caller"></a>呼び出し元からの通話終了を処理する
ピアツーピアの会話でユーザーがボットへの通話を切ると、あるいは会議からボットが削除されると、ボットに呼び出し状態の変化通知が届き、CallState が Terminated になります。 ボットでは、それによって作成されたメディア ソケットをすべて破棄する必要があります。

### <a name="terminate-the-call-from-the-bot"></a>ボットからの呼び出しを終了する
ボットでは、`IRealTimeMediaCallService` で `EndCall` を呼び出すことで通話を終了できます。

### <a name="handle-call-clean-up-by-the-bot-framework"></a>Bot Framework で呼び出しのクリーンアップを処理する
エラー状態では (たとえば、妥当な時間内に `AnswerAppHostedMediaOutcomeEvent` が届かない)、Bot Framework によって呼び出しが終了されることがあります。 ボットでは `OnCallCleanup` イベントを登録し、メディア ソケットを破棄する必要があります。

