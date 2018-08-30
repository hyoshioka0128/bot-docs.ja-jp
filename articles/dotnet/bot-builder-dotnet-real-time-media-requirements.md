---
title: リアルタイム メディア ボットの要件と考慮事項 | Microsoft Docs
description: Bot Builder SDK for .NET を使用した Skype 用のリアルタイム メディア ボットの作成に関連する要件と考慮事項を理解します。
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 091c90da9b14c0abe70d08f45f528a3cce818cef
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904036"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>リアルタイム メディア ボットの要件と考慮事項

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

メッセージング ボットと IVR 通話ボット開発に適用されるすべてのガイダンスが、リアルタイム メディア ボットの構築に同じように適用されるわけではありません。 この記事では、リアルタイム メディア ボットを開発するためのいくつかの重要な要件と考慮事項について説明します。 

> [!NOTE]
> ボット用の Real-Time Media Platform はプレビュー段階にあるテクノロジーであるため、この記事に記載されているガイダンスは変更されることがあります。

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>リアルタイム メディア ボットの開発では C#/.NET と Windows Server が必要

- リアルタイム メディア ボットは、音声メディアと動画メディアにアクセスするために `Microsoft.Skype.Bots.Media` .NET ライブラリ (<a href="https://www.nuget.org/" target="_blank">NuGet</a> から入手可能) が必要であり、Windows Server マシン (または Azure 内の Windows Server ゲスト OS) 上にデプロイされる必要があります。 したがって、ボットは、C# と Standard .NET Framework を使用して開発し、Azure 内にデプロイする必要があります。 C++ と Node.js API を使用してリアルタイム メディアにアクセスすることはできません。

- リアルタイム メディア ボットは、`Microsoft.Skype.Bots.Media` .NET ライブラリの最新バージョン上で実行する必要があります。 ボットは、<a href="https://www.nuget.org/" target="_blank">NuGet</a> パッケージの使用可能な最新のバージョン、または 3 か月以内にリリースされたバージョンのいずれかを使用する必要があります。 古いバージョンのライブラリは非推奨になり、数か月後に機能しなくなります。

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>リアルタイム メディア通話は作成されたマシン上にとどまる

- リアルタイム メディア ボットは、非常にステートフルです。 リアルタイム メディア通話は、着信通話を受け取った仮想マシン (VM) インスタンスに固定されます。Skype の発信元からの音声メディアと動画メディアは、その VM インスタンスにフローします。ボットが発信元に戻すメディアも、その VM を起源にする必要があります。

- VM が停止されたときに進行中のリアルタイム メディア通話がある場合、それらの通話は突然終了します。 ボットは、保留中の VM のシャット ダウンを知っていれば、通話の "正常な" 終了を試すことができます。

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>リアルタイム メディア ボットはインターネットに直接アクセスできる必要がある

- リアルタイム メディア ボットをホストする各 VM インスタンスは、インターネットから直接アクセス可能である必要があります。 Azure でホストされるリアルタイム メディア ボットでは、各 VM インスタンスが、インスタンスレベルのパブリック IP アドレス (ILPIP) を持っている必要があります。 ILPIP の取得と構成の情報については、「<a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">インスタンス レベル パブリック IP (クラシック) の概要</a>」を参照してください。 既定では、Azure サブスクリプションは、5 つの ILPIP アドレスを取得できます。サブスクリプションのクォータを増やす場合は、Azure サポートにお問い合わせください。

- パブリック IP アドレスに関する要件のため、リアルタイム メディア ボットは、"IaaS" Azure Virtual Machines または "クラシック" Azure Cloud Services のいずれかでホストする必要があります。 Bot Service や VM Scale Sets を使用する Service Fabric などのランタイム環境は ILPIP をサポートしていないため、これらの環境はサポートされません。

- リアルタイム メディア ボットは、[Bot Framework Emulator](../bot-service-debug-emulator.md) によるサポートもありません。

## <a name="scalability-and-performance-considerations"></a>スケーラビリティとパフォーマンスに関する考慮事項

- リアルタイム メディア ボットは、メッセージング ボット よりも多くの計算容量とネットワーク (帯域幅) 容量が必要であり、非常に高い運用コストが発生する可能性があります。 リアルタイム メディア ボットの開発者は、ボットのスケーラビリティを慎重に測定し、管理できる数を超える呼び出しをボットが同時に受理しないことを保証する必要があります。 動画対応ボットは、CPU コアあたり 1 つまたは 2 つの同時実行されるリアルタイム メディア呼び出しを維持できる場合があります。

- Real-Time Media Platform for Bots の現在のプレビュー リリースには、ボット開発者が注意する必要がある一定のスケーラビリティの制限があります (これらは、将来のリリースでは改善される可能性があります)。 
  1. 1 つの VM インスタンスは、特定の時点で、10 個を超えるビデオ ソケットを作成することはできません。
  2. Real-Time Media Platform は、現時点では、VM 上で使用できるグラフィックス処理ユニット (GPU) を利用したオフロードの H.264 ビデオ エンコード/デコードを行っていません。 代わりに、ビデオのエンコードとデコードは、CPU 上のソフトウェア内で実行されます。 GPU を利用できる場合、ボットは、独自のグラフィックス レンダリング (ボットが 3D グラフィックス エンジンを使用する場合など) でそれを活用できます。

- リアルタイム メディア ボットをホストする VM インスタンスには、少なくとも 2 つの CPU コアが必要です。 Azure では、Dv2 シリーズの仮想マシンをお勧めします。 Azure VM の種類に関する詳細情報については、<a href="/azure/virtual-machines/windows/sizes-general" target="_blank">Azure ドキュメント</a>を参照してください。 
