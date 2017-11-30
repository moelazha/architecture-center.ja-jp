---
title: "スケールアウトのための設計"
description: "クラウド アプリケーションは、水平方向のスケーリングで設計する必要があります。"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 8f9b3e99a53f5941f708b0de124f37e6ff7e5ab2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="design-to-scale-out"></a>スケールアウトのための設計

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>水平方向に拡張できるようにアプリケーションを設計する

クラウドの主な利点は、柔軟なスケーリングです。必要なだけの容量を使用でき、負荷が増加すればスケールアウトし、余分な容量が必要ない場合にはスケールインします。 需要に応じて新規インスタンスを追加または削除し、水平方向に拡張できるようにアプリケーションを設計します。

## <a name="recommendations"></a>Recommendations

**インスタンスの粘性の阻止**。 粘性または*セッション親和性*とは、同じクライアントからの要求は常に同じサーバーにルーティングされることを指します。 粘性により、アプリケーションのスケールアウトの機能が制限されます。たとえば、大量のユーザーからのトラフィックがインスタンス間で分散されません。 粘性の原因となるものには、メモリでのセッション状態の保存、および暗号化用のマシン固有のキーの使用などがあります。 あらゆるインスタンスであらゆる要求を処理できることを確認してください。 

**ボトルネックの特定** スケールアウトは、すべてのパフォーマンスの問題が修正されるマジックではありません。 たとえば、バックエンドのデータベースがボトルネックである場合、Web サーバーを追加しても改善されません。 問題に対してより多くのインスタンスを投入する前に、まずシステムでボトルネックを特定して解決します。 システムのステートフルな部分は、最もボトルネックの原因となりやすいものです。 

**スケーラビリティの要件によるワークロードの分解**。  アプリケーションは多くの場合、異なるスケーリング要件がある、複数のワークロードで構成されます。 たとえば、公開サイトと、それとは別に管理サイトがあるアプリケーションがあります。 公開サイトでは突然のトラフィック増加が発生する可能性がある一方、管理サイトの負荷はより小さな、より予測がしやすいものです。 

**多くのリソースを消費するタスクのオフロード**。 多くの CPU または I/O リソースを必要とするタスクは、可能であれば[バックグラウンド ジョブ][background-jobs]に移動し、ユーザーの要求を処理しているフロント エンド上の負荷を最小限にする必要があります。

**組み込みの自動スケール機能の使用**。 多くの Azure コンピューティング サービスでは、自動スケーリングの組み込みサポートがあります。 アプリケーションに予測可能な通常のワークロードがある場合は、スケジュールに基づいてスケールアウトします。 たとえば、営業時間内にスケールアウトします。 ワークロードが予測可能でない場合は、CPU や要求キューの長さなどのパフォーマンス メトリックを使用して、自動スケーリングをトリガーします。 自動スケールのベスト プラクティスについては、「[自動スケール][autoscaling]」をご覧ください。

**クリティカルなワークロードの積極的な自動スケーリングの検討**。 クリティカルなワークロードの場合、ニーズには常に事前に対処する必要があります。 負荷の高い場所に新しいインスタンスを追加して過度のトラフィックを処理し、段階的にスケールバックすることをお勧めします。

**スケールインの設計**。  柔軟なスケールでは、インスタンスが削除されると、アプリケーションにスケールインの期間が生じることに注意してください。 アプリケーションでは、削除中のインスタンスを適切に処理する必要があります。 スケールインを処理する方法はいくつかあります。

- シャットダウン イベント (使用可能な場合) をリッスンし、正常にシャットダウンします。 
- サービスのクライアントまたはコンシューマーは一時的な障害の処理に対応し、再試行する必要があります。 
- 実行時間の長いタスクの場合は、チェックポイントまたは[パイプとフィルター処理][pipes-filters-pattern]パターンを使用して、作業を分割することを検討してください。 
- 作業項目をキューに配置して、インスタンスが処理の最中に削除された場合は別のインスタンスが作業を取得できるようにします。 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md