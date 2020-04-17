---
title: チャネル リファレンス
description: Bot Framework チャネル リファレンス
keywords: チャネル リファレンス, bot builder チャネル, bot framework チャネル
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/03/2019
ms.openlocfilehash: d8465a1cfa567790d62fc58ac29b6a452418f582
ms.sourcegitcommit: 9d77f3aff9521d819e88efd0fbd19d469b9919e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "80117561"
---
# <a name="categorized-activities-by-channel"></a>チャネル別に分類されたアクティビティ

以下の表は、どのようなイベント (ネットワーク上のアクティビティ) がどのチャネルから発生する可能性があるかを示しています。

各表の凡例を次に示します。

Symbol              | 意味
:------------------:|:------------------------------------------------
:white_check_mark:  |ボットが受信することが予想されるアクティビティ
:x:                 |ボットが受信することの**ない**アクティビティ
:white_large_square:|ボットが受信できるかどうか、現時点では未定

アクティビティは、意味のある方法で別々のカテゴリに分けることができます。 カテゴリごとに、発生する可能性のあるアクティビティの表を以下に示します。

<a name="conversational"></a>会話
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Message                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- すべてのチャネルがメッセージ アクティビティを送信します。
- ダイアログを使用する場合、一般に、Message アクティビティは常にダイアログに渡す必要があります。
- MessageReaction は会話の非常に大きな部分を占めていますが、これは MessageReaction には当てはまらないと考えられます。
- 論理的に 2 種類の MessageReaction があります (Added と Removed)。


> [!TIP]
> "メッセージの反応" は、前のコメントに対する "サムズアップ (賛成、OK、グッドなど)" のようなものです。 これらは順不同で発生する可能性があるため、ボタンと同じように考えることができます。 現在、このアクティビティは Teams チャネルによって送信されます。  


<a name="welcome"></a>ようこそ
-------

 \                         | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- チャネルが ConversationUpdate アクティビティを送信するのが一般的です。
- 論理的に 2 種類の MessageReaction があります (Added と Removed)。
- ConversationUpdate.Added につなぐことで、ボットの "ウェルカム" 動作を簡単に実装できると想定しがちであり、これが機能する場合もあります。
- しかし、これは単純化されたものであり、信頼性のある "ウェルカム" 動作を生成するために、ボット実装で状態を使用することが必要な場合もあります。


<a name="application-extensibility"></a>アプリケーションの機能拡張
-------------------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Event アクティビティは、Direct Line ("_Web チャットとも呼ばれます_") の機能拡張メカニズムです。
- クライアントとサーバーの両方を所有するアプリケーションでは、この Event アクティビティを使用して、サービスを介して独自のイベントをトンネリングできます。


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- Microsoft Teams では、他の型指定された多数のアクティビティに加え、Teams 固有の Invoke アクティビティがいくつか定義されています。
- Invoke アクティビティはアプリケーションに固有のものであり、クライアントが定義するものではありません。
- アクティビティの特定のサブタイプを呼び出すという一般的概念はありません。
- 現在、ボット上で要求/応答動作をトリガーするアクティビティは Invoke だけです。

これは非常に重要です。ダイアログを使用して OAuth プロンプトを動作させる場合、Invoke.TeamsVerification アクティビティをダイアログに転送する必要があります。


<a name="message-update"></a>メッセージの更新
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- メッセージの更新は、Teams で現在サポートされています。


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

これは非常に重要です。ダイアログを使用して OAuth プロンプトを動作させる場合、Event.TokenResponse アクティビティをダイアログに転送する必要があります。


<a name="uncategorized"></a>未分類 
-------------

 \                      | Cortana  | Direct Line        | Direct Line (Web チャット) | Email | Facebook | GroupMe | Kik     | Teams | Slack | Skype | Skype Business | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing (入力)                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>使用されていないアクティビティ (支払い固有の呼び出しを含む)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- ping

---

## <a name="summary-of-activities-supported-per-channel"></a>各チャネルでサポートされているアクティビティの概要

<a name="cortana"></a>Cortana
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (ウィンドウが閉じたとき?)_

<a name="direct-line"></a>Direct Line
--------
- Message
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Message

<a name="facebook"></a>Facebook
--------
- Message
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- Message
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Message
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype Business
--------------
- Message
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Message

## <a name="summary-table-all-activities-to-all-channels"></a>すべてのチャネルに対するすべてのアクティビティの概要表

 \                         | Cortana              | Direct Line          | Direct Line (Web チャット) | Email                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Message                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing (入力)                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>Web チャット 
Web チャットは以下を送信します。
- "message": "text" や "attachments" が含まれます。
- "event": "name" と "value" が (JSON/文字列として) 含まれます。
- "typing": ユーザーがオプション ("sendTypingIndicator") を設定した場合、Web チャットは "contactRelationUpdate" を送信しなくなります。 また、Web チャットでは "messageReaction" はサポートされていません。Microsoft はお客様からこの機能のサポートを明示的に要求されたことはありません。

既定では、Web チャットは以下をレンダリングします。
- "message": アクティビティのオプションに応じて、カルーセルまたはスタックとしてレンダリングします。
- "typing": 5 秒間レンダリングして非表示にするか、次のアクティビティが発生するまでレンダリングします。
- "conversationUpdate": 非表示になります。
- "event": 非表示になります。
- その他: 警告ボックスを表示します (運用環境で表示されたことはありません)。このレンダリング パイプラインを変更して、カスタム レンダリングを追加、削除、または置換できます。

Web チャットを使用して任意の種類のアクティビティとペイロードを送信できますが、この機能は文書化されておらず、推奨もされていません。 代わりに "event" アクティビティを使用してください。

## <a name="action-support-by-channel"></a>チャネル別のアクションのサポート

次の表は、それぞれのチャネルでサポートされるカード アクションと推奨されるアクションの最大数を示しています。  :x: は、そのアクションが、当該のチャネルでは一切サポートされないことを意味します。

| \                 | Cortana | Direct Line | Direct Line (Web チャット) | Email | Facebook | GroupMe |   Kik   | Line  | Teams | Slack | Skype | Skype Business | Telegram | Twilio | 
| :---------------- | :-----: | :---------: | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :---: | :------------: | :------: | :----: |
| Suggested Actions (推奨されるアクション) |   :x:   |     100     |          100           |  :x:  |    10    |   :x:   |   20    |  13   |  :x:  |  100  |  10   |      :x:       |    100   |   :x:  |  
| カード アクション      |   100   |     100     |          100           |  :x:  |     3    |   :x:   |   20    |  99   |   3   |  100  |   3   |      :x:       |    :x:   |   :x:  |  

上の表に示されている数値の詳細については、[こちら](https://aka.ms/channelactions)に記載されているチャネル サポート コードを参照してください。 

"_推奨されるアクション_" の詳細については、「[入力にボタンを使用する](https://aka.ms/howto-add-buttons)」の記事を参照してください。

"_カード アクション_" の詳細については、「_メッセージにメディアを追加する_」の記事の「[ヒーロー カードを送信する](https://aka.ms/howto-add-media#send-a-hero-card)」セクションを参照してください。

## <a name="card-support-by-channel"></a>チャネル別のカードのサポート

| チャネル | アダプティブ カード | アニメーション カード | オーディオ カード | ヒーロー カード | Receipt Card (領収書カード) | Signin Card (サインイン カード) | Thumbnail Card (サムネイル カード) | ビデオ カード |
|:-------:|:-------------:|:--------------:|:----------:|:---------:|:------------:|:-----------:|:--------------:|:----------:|
|Cortana|✔|❌|❌|❌|✔|✔|✔|❌|
|Email|🔶|🌐|🌐|✔|✔|✔|✔|🌐|
|Facebook|⚠🔶|✔|❌|✔|✔|✔|✔|❌|
|GroupMe|🔶|🌐|🌐|🌐|🌐|🌐|🌐|🌐|
|Kik|🔶|✔|✔|❌|🌐|❌|✔|🌐|
|Line|⚠🔶|✔|🌐|✔|✔|✔|✔|🌐|
|Microsoft Teams|✔|❌|❌|✔|✔|✔|✔|❌|
|Skype|❌|✔|✔|✔|✔|✔|✔|✔|
|Slack|🔶|✔|🌐|🌐|✔|✔|🌐|🌐|
|Telegram|⚠🔶|✔|🌐|✔|✔|✔|✔|✔|
|Twilio|🔶|🌐|❌|🌐|🌐|🌐|🌐|❌|
|Web チャット|✔|✔|✔|✔|✔|✔|✔|✔|

*注意事項: Direct Line チャネルは技術的にすべてのカードをサポートしていますが、それらを実装するかどうかの判断はクライアントに任されます*

* ✔:サポート対象 - カードは完全にサポートされています。ただし、一部のチャネルでは、CardActions のサブセットのみがサポートされている場合や、各カードで許可されるアクションの数が制限されている場合があります。  チャネルによって異なります。
* ⚠:部分的なサポート - 入力やボタンが含まれているカードは全く表示されないことがあります。 チャネルによって異なります。
* ❌:サポートなし
* 🔶:カードは画像に変換されます
* 🌐:カードは書式なしテキストに変換されます - リンクのクリック、画像の表示、およびメディアの再生が実行できない場合があります。 チャネルによって異なります。

カード、機能、およびチャネルの可能な組み合わせが多数あるため、これらのカテゴリでは、意図的に広範囲を扱い、各チャネルであらゆるカード機能がどのようにサポートされているかを完全には説明していません。 この表は基本的な参照として使用し、目的のチャネルで各カードをテストしてください。
