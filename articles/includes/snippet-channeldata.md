# <a name="implement-channel-specific-functionality"></a>チャネル固有の機能の実装

一部のチャネルには、[メッセージのテキストと添付ファイル](../dotnet/bot-builder-dotnet-create-messages.md)だけでは実装できない機能が用意されています。 チャネル固有の機能を実装するには、`Activity` オブジェクトの `ChannelData` プロパティを使用して、ネイティブのメタデータをチャネルに渡します。 たとえば、お使いのボットで `ChannelData` プロパティを使用して、ステッカーを送信するように Telegram に指示したり、電子メールを送信するように Office365 に指示したりできます。

この記事では、メッセージ アクティビティの `ChannelData` プロパティを使用して、このチャネル固有の機能を実装する方法について説明します。

| チャネル | 機能 |
|----|----|
| 電子メール | 本文、件名、および重要度のメタデータを含む電子メールを送受信します |
| Slack | 完全に忠実な Slack メッセージを送信します |
| Facebook | Facebook 通知をネイティブで送信します |
| Telegram | 音声メモ、ステッカーの共有など、Telegram 固有のアクションを実行します |
| Kik | ネイティブ Kik メッセージを送受信します | 

> [!NOTE]
> `Activity` オブジェクトの `ChannelData` プロパティの値は JSON オブジェクトです。 そのため、この記事の例では、さまざまなシナリオにおいて有効な `channelData` JSON プロパティの書式を示します。 .NET を使用して JSON オブジェクトを作成するには、`JObject` (.NET) クラスを使用します。 

## <a name="create-a-custom-email-message"></a>カスタム電子メール メッセージを作成する

電子メール メッセージを作成するには、`Activity` オブジェクトの `ChannelData` プロパティを、次のプロパティを含む JSON オブジェクトに設定します。 

| プロパティ | 説明 |
|----|----|
| htmlBody | 電子メール メッセージの本文を指定する HTML ドキュメント。 サポートされている HTML の要素と属性については、チャネルのドキュメントを参照してください。 |
| importance | 電子メールの重要度レベル。 有効な値は、**high**、**normal**、および **low** です。 既定値は **normal** です。 |
| subject | 電子メールの件名です。 フィールドの要件については、チャネルのドキュメントを参照してください。 |

> [!NOTE]
> ボットがユーザーから電子メール チャネル経由で受信したメッセージには、上記で説明した JSON オブジェクトが設定された `ChannelData` プロパティが含まれている可能性があります。

次のスニペットは、カスタム電子メール メッセージ用の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "htmlBody" : "<html><body style=\"font-family: Calibri; font-size: 11pt;\">This is the email body!</body></html>",
    "subject":"This is the email subject",
    "importance":"high"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>完全に忠実な Slack メッセージを作成する

完全に忠実な Slack メッセージを作成するには、`Activity` オブジェクトの `ChannelData` プロパティを、<a href="https://api.slack.com/docs/messages" target="_blank">Slack のメッセージ</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack の添付ファイル</a>、<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack のボタン</a>を指定する JSON オブジェクトに設定します。 

> [!NOTE]
> Slack メッセージでボタンをサポートするには、Slack チャネルに[お使いのボットを接続](../bot-service-manage-channels.md)するときに、**インタラクティブ メッセージ**を有効にする必要があります。

このスニペットは、カスタム Slack メッセージ用の `channelData` プロパティの例を示しています。

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

ユーザーが Slack メッセージ内のボタンをクリックすると、お使いのボットに応答メッセージが届きます。このメッセージの `ChannelData` プロパティには `payload` JSON オブジェクトが設定されています。 `payload` オブジェクトによって、元のメッセージのコンテンツが指定され、クリックされたボタンと、そのボタンをクリックしたユーザーが特定されます。 

このスニペットは、ユーザーが Slack メッセージ内のボタンをクリックしたときに、ボットに届くメッセージ内の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

お使いのボットでは[通常の方法](../dotnet/bot-builder-dotnet-connector.md#create-reply)でこのメッセージに返信するか、その応答を、`payload` オブジェクトの `response_url` プロパティで指定されているエンドポイントに直接投稿できます。
応答を `response_url` に投稿するタイミングと方法については、<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack ボタン</a>に関するページをご覧ください。 

動的なボタンの作成には、次のコードを使用できます。
```cs
private async Task DemoButtonsAsync(IDialogContext context)
        {
            var reply = context.MakeMessage();

            string s = @"{
                ""text"": ""Would you like to play a game ? "",
                ""attachments"": [
                    {
                        ""text"": ""Choose a game to play!"",
                        ""fallback"": ""You are unable to choose a game"",
                        ""callback_id"": ""wopr_game"",
                        ""color"": ""#3AA3E3"",
                        ""attachment_type"": ""default"",
                        ""actions"": [
                            {
                                ""name"": ""game"",
                                ""text"": ""Chess"",
                                ""type"": ""button"",
                                ""value"": ""chess""
                            },
                            {
                                ""name"": ""game"",
                                ""text"": ""Falken's Maze"",
                                ""type"": ""button"",
                                ""value"": ""maze""
                            },
                            {
                                ""name"": ""game"",
                                ""text"": ""Thermonuclear War"",
                                ""style"": ""danger"",
                                ""type"": ""button"",
                                ""value"": ""war"",
                                ""confirm"": {
                                    ""title"": ""Are you sure?"",
                                    ""text"": ""Wouldn't you prefer a good game of chess?"",
                                    ""ok_text"": ""Yes"",
                                    ""dismiss_text"": ""No""
                                }
                            }
                        ]
                    }
                ]
            }";

            reply.Text = null;
            reply.ChannelData = JObject.Parse(s);
            await context.PostAsync(reply);
            context.Wait(MessageReceivedAsync);
        }
```

対話型メニューを作成するには、次のコードを使用します。
```cs
private async Task DemoMenuAsync(IDialogContext context)
        {
            var reply = context.MakeMessage();

            string s = @"{
                ""text"": ""Would you like to play a game ? "",
                ""response_type"": ""in_channel"",
                ""attachments"": [
                    {
                        ""text"": ""Choose a game to play"",
                        ""fallback"": ""If you could read this message, you'd be choosing something fun to do right now."",
                        ""color"": ""#3AA3E3"",
                        ""attachment_type"": ""default"",
                        ""callback_id"": ""game_selection"",
                        ""actions"": [
                            {
                                ""name"": ""games_list"",
                                ""text"": ""Pick a game..."",
                                ""type"": ""select"",
                                ""options"": [
                                    {
                                        ""text"": ""Hearts"",
                                        ""value"": ""menu_id_hearts""
                                    },
                                    {
                                        ""text"": ""Bridge"",
                                        ""value"": ""menu_id_bridge""
                                    },
                                    {
                                        ""text"": ""Checkers"",
                                        ""value"": ""menu_id_checkers""
                                    },
                                    {
                                        ""text"": ""Chess"",
                                        ""value"": ""menu_id_chess""
                                    },
                                    {
                                        ""text"": ""Poker"",
                                        ""value"": ""menu_id_poker""
                                    },
                                    {
                                        ""text"": ""Falken's Maze"",
                                        ""value"": ""menu_id_maze""
                                    },
                                    {
                                        ""text"": ""Global Thermonuclear War"",
                                        ""value"": ""menu_id_war""
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }";

            reply.Text = null;
            reply.ChannelData = JObject.Parse(s);
            await context.PostAsync(reply);
            context.Wait(MessageReceivedAsync);
        }
```

## <a name="create-a-facebook-notification"></a>Facebook 通知を作成する

Facebook 通知を作成するには、`Activity` オブジェクトの `ChannelData` プロパティを JSON オブジェクトに設定し、次のプロパティを指定します。 

| プロパティ | 説明 |
|----|----|
| notification_type | 通知の種類 (例: **REGULAR**、**SILENT_PUSH**、**NO_PUSH**)。
| attachment | 画像、動画、またはその他のマルチメディアの種類を指定する添付ファイル、または受信確認などのテンプレート化された添付ファイル。 |

> [!NOTE]
> `notification_type` プロパティおよび `attachment` プロパティの形式とコンテンツの詳細については、<a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">Facebook API のドキュメント</a>を参照してください。 

このスニペットは、Facebook 受信確認添付ファイル用の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Telegram メッセージを作成する

音声メモやステッカーの共有など、Telegram 固有のアクションが実装されているメッセージを作成するには、`Activity` オブジェクトの `ChannelData` プロパティを JSON オブジェクトに設定し、次のプロパティを指定します。 

| プロパティ | 説明 |
|----|----|
| method | 呼び出す Telegram Bot API メソッド。 |
| parameters | 指定されたメソッドのパラメーター。 |

次の Telegram メソッドがサポートされています。 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

これらの Telegram メソッドとそのパラメーターの詳細については、<a href="https://core.telegram.org/bots/api#available-methods" target="_blank">Telegram Bot API のドキュメント</a>を参照してください。

> [!NOTE]
> <ul><li><code>chat_id</code> パラメーターは、すべての Telegram メソッドに共通です。 <code>chat_id</code> をパラメーターとして指定しない場合は、フレームワークによって ID が自動的に指定されます。</li>
> <li>ファイル コンテンツをインラインで渡すのではなく、以下の例で示すように、URL およびメディアの種類を使用してファイルを指定します。</li>
> <li>お使いのボットで受信した、Telegram チャネルからの各メッセージの <code>ChannelData</code>プロパティに、お使いのボットによって以前送信されたメッセージが指定されます。</li></ul>

このスニペットは、1 つの Telegram メソッドを指定する `channelData` プロパティの例を示しています。

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

このスニペットは、Telegram メソッドの配列を指定する `channelData` プロパティの例を示しています。

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>ネイティブ Kik メッセージを作成する

ネイティブ Kik メッセージを作成するには、`Activity` オブジェクトの `ChannelData` プロパティを JSON オブジェクトに設定し、次のプロパティを指定します。 

| プロパティ | 説明 |
|----|----|
| 最大配信数 | Kik メッセージの配列。 Kik メッセージ形式の詳細については、<a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik メッセージ形式</a>に関するページをご覧ください。 |

このスニペットは、ネイティブ Kik メッセージ用の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```
 
## <a name="additional-resources"></a>その他のリソース

- [アクティビティの概要](../dotnet/bot-builder-dotnet-activities.md)
- [メッセージの作成](../dotnet/bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity クラス</a>
