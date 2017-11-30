---
title: "CQRS アーキテクチャのスタイル"
description: "CQRS アーキテクチャのメリット、課題、ベスト プラクティスを説明します"
author: MikeWasson
ms.openlocfilehash: dd3da5886587159f57646ff1bfffa2094725f798
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="cqrs-architecture-style"></a><span data-ttu-id="521f1-103">CQRS アーキテクチャのスタイル</span><span class="sxs-lookup"><span data-stu-id="521f1-103">CQRS architecture style</span></span>

<span data-ttu-id="521f1-104">コマンド クエリ責務分離 (CQRS) は、読み取り操作と書き込み操作を分離するアーキテクチャ スタイルです。</span><span class="sxs-lookup"><span data-stu-id="521f1-104">Command and Query Responsibility Segregation (CQRS) is an architecture style that separates read operations from write operations.</span></span> 

![](./images/cqrs-logical.svg)

<span data-ttu-id="521f1-105">従来のアーキテクチャでは、データベースの更新とクエリに同じデータ モデルが使用されます。</span><span class="sxs-lookup"><span data-stu-id="521f1-105">In traditional architectures, the same data model is used to query and update a database.</span></span> <span data-ttu-id="521f1-106">このシンプルな方法は、基本的な CRUD 操作に適しています。</span><span class="sxs-lookup"><span data-stu-id="521f1-106">That's simple and works well for basic CRUD operations.</span></span> <span data-ttu-id="521f1-107">ただし、複雑なアプリケーションの場合、このアプローチではうまくいかないことがあります。</span><span class="sxs-lookup"><span data-stu-id="521f1-107">In more complex applications, however, this approach can become unwieldy.</span></span> <span data-ttu-id="521f1-108">たとえば、読み取り側でさまざまなクエリが実行され、形式の異なる複数のデータ転送オブジェクト (DTO) が返される場合もあります。</span><span class="sxs-lookup"><span data-stu-id="521f1-108">For example, on the read side, the application may perform many different queries, returning data transfer objects (DTOs) with different shapes.</span></span> <span data-ttu-id="521f1-109">これにより、オブジェクトのマッピングが複雑になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-109">Object mapping can become complicated.</span></span> <span data-ttu-id="521f1-110">また書き込み側のモデルでは、複雑な検証とビジネス ロジックが実装される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-110">On the write side, the model may implement complex validation and business logic.</span></span> <span data-ttu-id="521f1-111">その結果、モデルが複雑になりすきる恐れがあります。</span><span class="sxs-lookup"><span data-stu-id="521f1-111">As a result, you can end up with an overly complex model that does too much.</span></span>

<span data-ttu-id="521f1-112">さらに、読み取りと書き込みのワークロードが不均衡になりやすいため、パフォーマンスやスケールの要件が大きく異なってくる可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="521f1-112">Another potential problem is that read and write workloads are often asymmetrical, with very different performance and scale requirements.</span></span> 

<span data-ttu-id="521f1-113">CQRS では、データを更新するための**コマンド**とデータを読み取るための**クエリ**を使用し、読み取りと書き込みを個別のモデルに分離することで、これらの問題に対処します。</span><span class="sxs-lookup"><span data-stu-id="521f1-113">CQRS addresses these problems by separating reads and writes into separate models, using **commands** to update data, and **queries** to read data.</span></span>

- <span data-ttu-id="521f1-114">コマンドは、データ中心ではなく、タスクベースにします</span><span class="sxs-lookup"><span data-stu-id="521f1-114">Commands should be task based, rather than data centric.</span></span> <span data-ttu-id="521f1-115">(「ReservationStatus を Reserved に設定する」などではなく、「ホテルの部屋を予約する」などの形式にします)。コマンドは、同期的に処理するのではなく、非同期処理のキューに配置できます。</span><span class="sxs-lookup"><span data-stu-id="521f1-115">("Book hotel room," not "set ReservationStatus to Reserved.") Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.</span></span>

- <span data-ttu-id="521f1-116">クエリでは、データベースは変更されません。</span><span class="sxs-lookup"><span data-stu-id="521f1-116">Queries never modify the database.</span></span> <span data-ttu-id="521f1-117">クエリでは、ドメイン ナレッジをカプセル化しない DTO が返されます。</span><span class="sxs-lookup"><span data-stu-id="521f1-117">A query returns a DTO that does not encapsulate any domain knowledge.</span></span>

<span data-ttu-id="521f1-118">分離性を高めるために、読み取りデータと書き込みデータを物理的に分離することもできます。</span><span class="sxs-lookup"><span data-stu-id="521f1-118">For greater isolation, you can physically separate the read data from the write data.</span></span> <span data-ttu-id="521f1-119">その場合は、読み取りデータベースでは、クエリ用に最適化された独自のデータ スキーマを使用できます。</span><span class="sxs-lookup"><span data-stu-id="521f1-119">In that case, the read database can use its own data schema that is optimized for queries.</span></span> <span data-ttu-id="521f1-120">たとえば、結合や O/RM マッピングが複雑になるのを回避するために、データの[具体化されたビュー][materialized-view]を格納することもできます。</span><span class="sxs-lookup"><span data-stu-id="521f1-120">For example, it can store a [materialized view][materialized-view] of the data, in order to avoid complex joins or complex O/RM mappings.</span></span> <span data-ttu-id="521f1-121">また、異なる種類のデータ ストアを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="521f1-121">It might even use a different type of data store.</span></span> <span data-ttu-id="521f1-122">たとえば、書き込みデータベースをリレーショナルにし、読み取りデータベースをドキュメント データベースにすることもできます。</span><span class="sxs-lookup"><span data-stu-id="521f1-122">For example, the write database might be relational, while the read database is a document database.</span></span>

<span data-ttu-id="521f1-123">読み取りデータベースと書き込みデータベースを個別に使用する場合は、両者の同期を維持する必要があります。これは通常、データベースの更新時に書き込みモデルでイベントを発行することによって達成されます。</span><span class="sxs-lookup"><span data-stu-id="521f1-123">If separate read and write databases are used, they must be kept in sync. Typically this is accomplished by  having the write model publish an event whenever it updates the database.</span></span> <span data-ttu-id="521f1-124">データベースの更新とイベントの発行は、単一のトランザクションで行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-124">Updating the database and publishing the event must occur in a single transaction.</span></span> 

<span data-ttu-id="521f1-125">CQRS の実装では、[イベント ソーシング パターン][event-sourcing]が使用される場合があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-125">Some implementations of CQRS use the [Event Sourcing pattern][event-sourcing].</span></span> <span data-ttu-id="521f1-126">このパターンを使用すると、アプリケーションの状態が一連のイベントとして格納されます。</span><span class="sxs-lookup"><span data-stu-id="521f1-126">With this pattern, application state is stored as a sequence of events.</span></span> <span data-ttu-id="521f1-127">各イベントは、データに対する一連の変更を表します。</span><span class="sxs-lookup"><span data-stu-id="521f1-127">Each event represents a set of changes to the data.</span></span> <span data-ttu-id="521f1-128">現在の状態は、これらのイベントを再生することによって構築されます。</span><span class="sxs-lookup"><span data-stu-id="521f1-128">The current state is constructed by replaying the events.</span></span> <span data-ttu-id="521f1-129">CQRS の場合、イベント ソーシングの利点の 1 つは、他のコンポーネントへの通知 (特に、読み取りモデルへの通知) に、同じイベントを使用できることです。</span><span class="sxs-lookup"><span data-stu-id="521f1-129">In a CQRS context, one benefit of Event Sourcing is that the same events can be used to notify other components &mdash; in particular, to notify the read model.</span></span> <span data-ttu-id="521f1-130">読み取りモデルでは、現在の状態のスナップショットを作成するのにイベントが使用されます (そのほうが、クエリにとってより効率的です)。</span><span class="sxs-lookup"><span data-stu-id="521f1-130">The read model uses the events to create a snapshot of the current state, which is more efficient for queries.</span></span> <span data-ttu-id="521f1-131">ただし、イベント ソーシングを使用すると、設計がより複雑になります。</span><span class="sxs-lookup"><span data-stu-id="521f1-131">However, Event Sourcing adds complexity to the design.</span></span>

![](./images/cqrs-events.svg)

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="521f1-132">どのようなときにこのアーキテクチャを使用するか</span><span class="sxs-lookup"><span data-stu-id="521f1-132">When to use this architecture</span></span>

<span data-ttu-id="521f1-133">多数のユーザーが同じデータにアクセスするコラボラティブなドメインについては、CQRS の使用を検討してください (特に、読み取りと書き込みのワークロードが不均衡な場合)。</span><span class="sxs-lookup"><span data-stu-id="521f1-133">Consider CQRS for collaborative domains where many users access the same data, especially when the read and write workloads are asymmetrical.</span></span>

<span data-ttu-id="521f1-134">CQRS は、システム全体に適用される最上位レベルのアーキテクチャではありません。</span><span class="sxs-lookup"><span data-stu-id="521f1-134">CQRS is not a top-level architecture that applies to an entire system.</span></span> <span data-ttu-id="521f1-135">CQRS は、読み取りと書き込みを分離することが効果的であるとはっきりわかっているサブシステムにのみ適用してください。</span><span class="sxs-lookup"><span data-stu-id="521f1-135">Apply CQRS only to those subsystems where there is clear value in separating reads and writes.</span></span> <span data-ttu-id="521f1-136">その他の場合は、複雑さが増すだけでメリットがありません。</span><span class="sxs-lookup"><span data-stu-id="521f1-136">Otherwise, you are creating additional complexity for no benefit.</span></span>

## <a name="benefits"></a><span data-ttu-id="521f1-137">メリット</span><span class="sxs-lookup"><span data-stu-id="521f1-137">Benefits</span></span>

- <span data-ttu-id="521f1-138">**独立してスケーリングできる**。</span><span class="sxs-lookup"><span data-stu-id="521f1-138">**Independently scaling**.</span></span> <span data-ttu-id="521f1-139">CQRS では、読み取りと書き込みの各ワークロードを個別にスケーリングできるので、ロック競合を減らせる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-139">CQRS allows the read and write workloads to scale independently, and may result in fewer lock contentions.</span></span>
- <span data-ttu-id="521f1-140">**最適化されたデータ スキーマ。**</span><span class="sxs-lookup"><span data-stu-id="521f1-140">**Optimized data schemas.**</span></span>  <span data-ttu-id="521f1-141">読み取り側ではクエリ用に最適化されたスキーマを使用し、書き込み側では更新用に最適化されたスキーマを使用できます。</span><span class="sxs-lookup"><span data-stu-id="521f1-141">The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.</span></span>  
- <span data-ttu-id="521f1-142">**セキュリティ**。</span><span class="sxs-lookup"><span data-stu-id="521f1-142">**Security**.</span></span> <span data-ttu-id="521f1-143">適切なドメイン エンティティだけがデータへの書き込みを実行している状態を維持しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="521f1-143">It's easier to ensure that only the right domain entities are performing writes on the data.</span></span>
- <span data-ttu-id="521f1-144">**懸念事項の分離**。</span><span class="sxs-lookup"><span data-stu-id="521f1-144">**Separation of concerns**.</span></span> <span data-ttu-id="521f1-145">読み取り側と書き込み側を分離することで、モデルの保守性と柔軟性を向上できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-145">Segregating the read and write sides can result in models that are more maintainable and flexible.</span></span> <span data-ttu-id="521f1-146">複雑なビジネス ロジックの多くは、書き込みモデルになります。</span><span class="sxs-lookup"><span data-stu-id="521f1-146">Most of the complex business logic goes into the write model.</span></span> <span data-ttu-id="521f1-147">読み取りモデルは、比較的シンプルにすることができます。</span><span class="sxs-lookup"><span data-stu-id="521f1-147">The read model can be relatively simple.</span></span>
- <span data-ttu-id="521f1-148">**クエリがよりシンプル**。</span><span class="sxs-lookup"><span data-stu-id="521f1-148">**Simpler queries**.</span></span> <span data-ttu-id="521f1-149">具体化されたビューを読み取りデータベースに格納することで、クエリ時の複雑な結合を回避できます。</span><span class="sxs-lookup"><span data-stu-id="521f1-149">By storing a materialized view in the read database, the application can avoid complex joins when querying.</span></span>

## <a name="challenges"></a><span data-ttu-id="521f1-150">課題</span><span class="sxs-lookup"><span data-stu-id="521f1-150">Challenges</span></span>

- <span data-ttu-id="521f1-151">**複雑さ**。</span><span class="sxs-lookup"><span data-stu-id="521f1-151">**Complexity**.</span></span> <span data-ttu-id="521f1-152">CQRS の基本的な考え方はシンプルです。</span><span class="sxs-lookup"><span data-stu-id="521f1-152">The basic idea of CQRS is simple.</span></span> <span data-ttu-id="521f1-153">ただし、アプリケーションの設計は複雑になる可能性があります。このことは、イベント ソーシング パターンが含まれる場合には特に顕著です。</span><span class="sxs-lookup"><span data-stu-id="521f1-153">But it can lead to a more complex application design, especially if they include the Event Sourcing pattern.</span></span>

- <span data-ttu-id="521f1-154">**メッセージング**。</span><span class="sxs-lookup"><span data-stu-id="521f1-154">**Messaging**.</span></span> <span data-ttu-id="521f1-155">CQRS ではメッセージングは必須ではありませんが、コマンドの発行やイベントの更新を処理するためにメッセージングが使用されることもよくあります。</span><span class="sxs-lookup"><span data-stu-id="521f1-155">Although CQRS does not require messaging, it's common to use messaging to process commands and publish update events.</span></span> <span data-ttu-id="521f1-156">その場合には、メッセージのエラーや重複を処理する必要が生じます。</span><span class="sxs-lookup"><span data-stu-id="521f1-156">In that case, the application must handle message failures or duplicate messages.</span></span> 

- <span data-ttu-id="521f1-157">**最終的な一貫性**。</span><span class="sxs-lookup"><span data-stu-id="521f1-157">**Eventual consistency**.</span></span> <span data-ttu-id="521f1-158">読み取りデータベースと書き込みデータベースを分割すると、読み取りデータが古くなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="521f1-158">If you separate the read and write databases, the read data may be stale.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="521f1-159">ベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="521f1-159">Best practices</span></span>

- <span data-ttu-id="521f1-160">CQRS の実装の詳細については、「[CQRS パターン][cqrs-pattern]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="521f1-160">For more information about implementing CQRS, see [CQRS Pattern][cqrs-pattern].</span></span>

- <span data-ttu-id="521f1-161">[イベント ソーシング][event-sourcing] パターンを使用して、更新の競合を回避することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="521f1-161">Consider using the [Event Sourcing][event-sourcing] pattern to avoid update conflicts.</span></span>

- <span data-ttu-id="521f1-162">読み取りモデルに[具体化されたビュー パターン][materialized-view]を使用して、クエリのスキーマを最適化することを検討してしてください。</span><span class="sxs-lookup"><span data-stu-id="521f1-162">Consider using the [Materialized View pattern][materialized-view] for the read model, to optimize the schema for queries.</span></span>

## <a name="cqrs-in-microservices"></a><span data-ttu-id="521f1-163">マイクロサービスでのCQRS</span><span class="sxs-lookup"><span data-stu-id="521f1-163">CQRS in microservices</span></span>

<span data-ttu-id="521f1-164">CQRS は、[マイクロサービス アーキテクチャ][microservices]で特に役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="521f1-164">CQRS can be especially useful in a [microservices architecture][microservices].</span></span> <span data-ttu-id="521f1-165">マイクロサービスの原則の 1 つは、サービスから別のサービスのデータ ストアに直接アクセスできないということです。</span><span class="sxs-lookup"><span data-stu-id="521f1-165">One of the principles of microservices is that a service cannot directly access another service's data store.</span></span>

![](./images/cqrs-microservices-wrong.png)

<span data-ttu-id="521f1-166">次の図では、サービス A がデータ ストアに書き込みを行い、サービス B が、具体化されたビューのデータを保持しています。</span><span class="sxs-lookup"><span data-stu-id="521f1-166">In the following diagram, Service A writes to a data store, and Service B keeps a materialized view of the data.</span></span> <span data-ttu-id="521f1-167">サービス A は、データ ストアに書き込みを行う際、必ずイベントを発行します。</span><span class="sxs-lookup"><span data-stu-id="521f1-167">Service A publishes an event whenever it writes to the data store.</span></span> <span data-ttu-id="521f1-168">サービス B は、そのイベントをサブスクライブします。</span><span class="sxs-lookup"><span data-stu-id="521f1-168">Service B subscribes to the event.</span></span>

![](./images/cqrs-microservices-right.png)


<!-- links -->

[cqrs-pattern]: ../../patterns/cqrs.md
[event-sourcing]: ../../patterns/event-sourcing.md
[materialized-view]: ../../patterns/materialized-view.md
[microservices]: ./microservices.md