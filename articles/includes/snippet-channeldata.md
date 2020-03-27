---
ms.openlocfilehash: 392eb8bb197105739222c37f695cfd785515b323
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117836"
---
一部のチャネルには、メッセージのテキストと添付ファイルを使用するだけでは実装できない機能が用意されています。 チャネル固有の機能を実装するには、アクティビティ オブジェクトの "_チャネル データ_" プロパティを使用して、ネイティブのメタデータをチャネルに渡します。 たとえば、お使いのボットでチャネル データ プロパティを使用して、ステッカーを送信するように Telegram に指示したり、電子メールを送信するように Office365 に指示したりできます。

この記事では、メッセージ アクティビティのチャネル データ プロパティを使用して、このチャネル固有の機能を実装する方法について説明します。

| チャネル  | 機能                                                                  |
| -------- | ------------------------------------------------------------------------------ |
| Email    | 本文、件名、および重要度のメタデータを含む電子メールを送受信します |
| Slack    | 完全に忠実な Slack メッセージを送信します                                              |
| Facebook | Facebook 通知をネイティブで送信します                                           |
| Telegram | 音声メモ、ステッカーの共有など、Telegram 固有のアクションを実行します   |
| Kik      | ネイティブ Kik メッセージを送受信します                                           |

> [!NOTE]
> アクティビティ オブジェクトのチャネル データ プロパティの値は JSON オブジェクトです。
> そのため、この記事の例では、さまざまなシナリオにおいて有効な `channelData` JSON プロパティの書式を示します。
> .NET を使用して JSON オブジェクトを作成するには、`JObject` (.NET) クラスを使用します。

## <a name="create-a-custom-email-message"></a>カスタム電子メール メッセージを作成する

電子メール メッセージを作成するには、アクティビティ オブジェクトのチャネル データ プロパティを、以下のプロパティを含む JSON オブジェクトに設定します。

| プロパティ      | 説明                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| bccRecipients | メッセージの BCC (ブラインド カーボン コピー) フィールドに追加する、セミコロン (;) 区切りの電子メール アドレスの文字列。                                                   |
| ccRecipients  | メッセージの CC (カーボン コピー) フィールドに追加する、セミコロン (;) 区切りの電子メール アドレスの文字列。                                                          |
| htmlBody      | 電子メール メッセージの本文を指定する HTML ドキュメント。 サポートされている HTML の要素と属性については、チャネルのドキュメントを参照してください。 |
| importance    | 電子メールの重要度レベル。 有効な値は、**high**、**normal**、および **low** です。 既定値は **normal** です。                                           |
| subject       | 電子メールの件名です。 フィールドの要件については、チャネルのドキュメントを参照してください。                                                               |
| toRecipients  | メッセージの宛先フィールドに追加する、セミコロン (;) 区切りの電子メール アドレスの文字列。                                                                        |

> [!NOTE]
> ボットがユーザーから電子メール チャネル経由で受信したメッセージには、上記で説明した JSON オブジェクトが設定されたチャネル データ プロパティが含まれている可能性があります。

次のスニペットは、カスタム電子メール メッセージ用の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>完全に忠実な Slack メッセージを作成する

完全に忠実な Slack メッセージを作成するには、アクティビティ オブジェクトのチャネル データ プロパティを、<a href="https://api.slack.com/docs/messages" target="_blank">Slack のメッセージ</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack の添付ファイル</a>、<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack のボタン</a>を指定する JSON オブジェクトに設定します。

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

ユーザーが Slack メッセージ内のボタンをクリックすると、お使いのボットに応答メッセージが届きます。このメッセージのチャネル データ プロパティには `payload` JSON オブジェクトが設定されています。
`payload` オブジェクトによって、元のメッセージのコンテンツが指定され、クリックされたボタンと、そのボタンをクリックしたユーザーが特定されます。

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

お使いのボットでは通常の方法でこのメッセージに返信するか、その応答を、`payload` オブジェクトの `response_url` プロパティで指定されているエンドポイントに直接投稿できます。
応答を `response_url` に投稿するタイミングと方法については、<a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack ボタン</a>に関するページをご覧ください。

次の JSON を使用して動的なボタンを作成できます。

```json
{
    "text": "Would you like to play a game ? ",
    "attachments": [
        {
            "text": "Choose a game to play!",
            "fallback": "You are unable to choose a game",
            "callback_id": "wopr_game",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "game",
                    "text": "Chess",
                    "type": "button",
                    "value": "chess"
                },
                {
                    "name": "game",
                    "text": "Falken's Maze",
                    "type": "button",
                    "value": "maze"
                },
                {
                    "name": "game",
                    "text": "Thermonuclear War",
                    "style": "danger",
                    "type": "button",
                    "value": "war",
                    "confirm": {
                        "title": "Are you sure?",
                        "text": "Wouldn't you prefer a good game of chess?",
                        "ok_text": "Yes",
                        "dismiss_text": "No"
                    }
                }
            ]
        }
    ]
}
```

対話型メニューを作成するには、次の JSON を使用します。

```json
{
    "text": "Would you like to play a game ? ",
    "response_type": "in_channel",
    "attachments": [
        {
            "text": "Choose a game to play",
            "fallback": "If you could read this message, you'd be choosing something fun to do right now.",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "callback_id": "game_selection",
            "actions": [
                {
                    "name": "games_list",
                    "text": "Pick a game...",
                    "type": "select",
                    "options": [
                        {
                            "text": "Hearts",
                            "value": "menu_id_hearts"
                        },
                        {
                            "text": "Bridge",
                            "value": "menu_id_bridge"
                        },
                        {
                            "text": "Checkers",
                            "value": "menu_id_checkers"
                        },
                        {
                            "text": "Chess",
                            "value": "menu_id_chess"
                        },
                        {
                            "text": "Poker",
                            "value": "menu_id_poker"
                        },
                        {
                            "text": "Falken's Maze",
                            "value": "menu_id_maze"
                        },
                        {
                            "text": "Global Thermonuclear War",
                            "value": "menu_id_war"
                        }
                    ]
                }
            ]
        }
    ]
}
```

## <a name="create-a-facebook-notification"></a>Facebook 通知を作成する

Facebook 通知を作成するには、アクティビティ オブジェクトのチャネル データ プロパティを JSON オブジェクトに設定し、以下のプロパティを指定します。

| プロパティ          | 説明                                                                                                          |
| ----------------- | -------------------------------------------------------------------------------------------------------------------- |
| notification_type | 通知の種類 (例: **REGULAR**、**SILENT_PUSH**、**NO_PUSH**)。                                          |
| attachment        | 画像、動画、またはその他のマルチメディアの種類を指定する添付ファイル、または受信確認などのテンプレート化された添付ファイル。 |

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

音声メモやステッカーの共有など、Telegram 固有のアクションが実装されているメッセージを作成するには、アクティビティ オブジェクトのチャネル データ プロパティを JSON オブジェクトに設定し、次のプロパティを指定します。 

| プロパティ   | 説明                             |
| ---------- | --------------------------------------- |
| method     | 呼び出す Telegram Bot API メソッド。    |
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

Telegram メソッドが実装されると、ボットはチャネル データ プロパティに JSON オブジェクトが設定されている応答メッセージを受信します。 この応答オブジェクトは、元のメッセージの内容 (`update_id` と最大で 1 つの省略可能なパラメーターを含む) を示します。 着信応答の受信の詳細については、「<a href="https://core.telegram.org/bots/api#getting-updates" target="_blank">更新プログラムの取得</a>」を参照してください。

このスニペットは、投票が作成されたときにボットが受信するメッセージ内の `channelData` プロパティの例を示しています。

```json
"channelData": {
    "update_id": 43517575,
    "message": {
        "message_id": 618,
        "from": {
            "id": 803613355,
            "is_bot": false,
            "first_name": "Joe",
            "last_name": "Doe",
            "username": "jdoe",
            "language_code": "en"
        },
        "chat": {
            "id": 803613355,
            "first_name": "Joe",
            "last_name": "Doe",
            "username": "jdoe",
            "type": "private"
        },
        "date": 1582577834,
        "poll": {
        "id": "5089525250643722242",
        "question": "How to win?",
        "options": [
            {
                "text": "Be the best",
                "voter_count": 0
            },
            {
                "text": "Help those in need",
                "voter_count": 0
            },
            {
                "text": "All of the above",
                "voter_count": 0
            }
        ],
        "total_voter_count": 0,
        "is_closed": false,
        "is_anonymous": true,
        "type": "regular",
        "allows_multiple_answers": false
        }
    }
}
```

## <a name="create-a-native-kik-message"></a>ネイティブ Kik メッセージを作成する

ネイティブ Kik メッセージを作成するには、アクティビティ オブジェクトのチャネル データ プロパティを JSON オブジェクトに設定し、次のプロパティを指定します。

| プロパティ | 説明                                                                                                                                                                 |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| messages | Kik メッセージの配列。 Kik メッセージ形式の詳細については、<a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik メッセージ形式</a>に関するページをご覧ください。 |

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

## <a name="create-a-line-message"></a>LINE のメッセージを作成する

LINE に固有のメッセージのタイプ (スタンプ、テンプレート、または携帯電話のカメラを開くといった LINE 固有のアクションのタイプ) を実装するメッセージを作成するには、LINE のメッセージのタイプとアクションのタイプを指定する JSON オブジェクトに、アクティビティ オブジェクトのチャネル データ プロパティを設定します。 

| プロパティ | 説明                       |
| -------- | --------------------------------- |
| type     | LINE のアクションまたはメッセージのタイプ名 |

次の LINE メッセージのタイプがサポートされています。
* スタンプ
* イメージマップ 
* テンプレート (ボタン、確認、カルーセル) 
* Flex 

次の LINE アクションを、メッセージ タイプ JSON オブジェクトの action フィールドで指定できます。 
* ポストバック 
* Message 
* URI 
* 日時選択 
* Camera 
* カメラロール 
* 場所 

これらの LINE メソッドとそのパラメーターの詳細については、[LINE Bot API のドキュメント](https://developers.line.biz/en/docs/messaging-api/)を参照してください。 

次のスニペットでは、`channelData`チャネル メッセージのタイプを指定するプロパティ`ButtonTemplate`と、3 つのアクション タイプ (カメラ、日時選択、カメラロール) の例を示します。 

```json
"channelData": { 
    "type": "ButtonsTemplate", 
    "altText": "This is a buttons template", 
    "template": { 
        "type": "buttons", 
        "thumbnailImageUrl": "https://example.com/bot/images/image.jpg", 
        "imageAspectRatio": "rectangle", 
        "imageSize": "cover", 
        "imageBackgroundColor": "#FFFFFF", 
        "title": "Menu", 
        "text": "Please select", 
        "defaultAction": { 
            "type": "uri", 
            "label": "View detail", 
            "uri": "http://example.com/page/123" 
        }, 
        "actions": [{ 
                "type": "cameraRoll", 
                "label": "Camera roll" 
            }, 
            { 
                "type": "camera", 
                "label": "Camera" 
            }, 
            { 
                "type": "datetimepicker", 
                "label": "Select date", 
                "data": "storeId=12345", 
                "mode": "datetime", 
                "initial": "2017-12-25t00:00", 
                "max": "2018-01-24t23:59", 
                "min": "2017-12-25t00:00" 
            } 
        ] 
    } 
}
```

## <a name="adding-a-bot-to-teams"></a>ボットを Teams に追加する

チームに追加されたボットは、別のチーム メンバーとなり、会話の中で `@mentioned` によって言及される可能性があります。 実際、ボットは、`@mentioned` でのみメッセージを受信するため、チャネルでの他の会話はボットに送信されません。
詳細については、「[Microsoft Teams のボットを使用したチャネルおよびグループ チャットでの会話](https://aka.ms/bots-con-channel)」を参照してください。

グループまたはチャネル内のボットは、メッセージ内でメンションされた (`@botname`) 場合にのみ応答するため、ボットが受信するグループ チャネル内のすべてのメッセージにはその名前が含まれており、メッセージの解析においてそれが処理されるようにする必要があります。 また、ボットは、メンションされた他のユーザーを解析し、そのメッセージの中でユーザーをメンションできます。

### <a name="check-for-and-strip-bot-mention"></a>@bot メンションの確認と削除

```csharp

Mention[] m = sourceMessage.GetMentions();
var messageText = sourceMessage.Text;

for (int i = 0;i < m.Length;i++)
{
    if (m[i].Mentioned.Id == sourceMessage.Recipient.Id)
    {
        //Bot is in the @mention list.
        //The below example will strip the bot name out of the message, so you can parse it as if it wasn't included. Note that the Text object will contain the full bot name, if applicable.
        if (m[i].Text != null)
            messageText = messageText.Replace(m[i].Text, "");
    }
}
```

```javascript
var text = message.text;
if (message.entities) {
    message.entities
        .filter(entity => ((entity.type === "mention") && (entity.mentioned.id.toLowerCase() === botId)))
        .forEach(entity => {
            text = text.replace(entity.text, "");
        });
    text = text.trim();
}

```

> [!IMPORTANT] 
> テスト以外の目的のために、GUID によってボットを追加することはお勧めできません。 そのようにすると、ボットの機能が厳しく制限されます。 運用環境のボットは、アプリの一部として Teams に追加する必要があります。 「[ボットの作成](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create)」と[「Microsoft Teams のボットのテストとデバッグ](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test)」を参照してください。


## <a name="additional-resources"></a>その他のリソース

- [エンティティとアクティビティの種類](../bot-service-activities-entities.md)
- [Bot Framework のアクティビティ スキーマ](https://aka.ms/botSpecs-activitySchema)
