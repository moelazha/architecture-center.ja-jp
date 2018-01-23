---
title: CQRS
description: "個別のインターフェイスを使用して、データを更新する操作とデータを読み取る操作を分離します。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: f36f759b16566a6c46bf78b8c8b8df4fcd2c493d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="28028-104">コマンド クエリ責務分離 (CQRS) パターン</span><span class="sxs-lookup"><span data-stu-id="28028-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="28028-105">個別のインターフェイスを使用して、データを更新する操作とデータを読み取る操作を分離します。</span><span class="sxs-lookup"><span data-stu-id="28028-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="28028-106">これにより、パフォーマンス、スケーラビリティ、セキュリティを最大化できます。</span><span class="sxs-lookup"><span data-stu-id="28028-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="28028-107">高い柔軟性でシステムの進化に経時的に対応し、更新コマンドによってドメイン レベルでマージ競合が発生するのを防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="28028-107">Supports the evolution of the system over time through higher flexibility, and prevent update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="28028-108">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="28028-108">Context and problem</span></span>

<span data-ttu-id="28028-109">従来のデータ管理システムでは、コマンド (データの更新) とクエリ (データの要求) は、1 つのデータ リポジトリ内のエンティティの同じセットに対して実行されます。</span><span class="sxs-lookup"><span data-stu-id="28028-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="28028-110">これらのエンティティは、リレーショナル データベース (SQL Server など) における 1 つ以上のテーブル内の行のサブセットである可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="28028-111">これらのシステムでは通常、作成、読み取り、更新、削除 (CRUD) の操作はすべて、同じエンティティ表現に適用されます。</span><span class="sxs-lookup"><span data-stu-id="28028-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="28028-112">たとえば、顧客を表すデータ転送オブジェクト (DTO) は、データ アクセス層 (DAL) によってデータ ストアから取得され、画面に表示されます。</span><span class="sxs-lookup"><span data-stu-id="28028-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="28028-113">ユーザーが (データ バインドによって) DTO の一部のフィールドを更新すると、DTO は DAL によってデータ ストアに保存されます。</span><span class="sxs-lookup"><span data-stu-id="28028-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="28028-114">読み取り操作と書き込み操作に同じ DTO が使用されます。</span><span class="sxs-lookup"><span data-stu-id="28028-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="28028-115">次の図は、従来の CRUD アーキテクチャを示しています。</span><span class="sxs-lookup"><span data-stu-id="28028-115">The figure illustrates a traditional CRUD architecture.</span></span>

![従来の CRUD アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="28028-117">従来の CRUD 設計は、限られたビジネス ロジックだけをデータ操作に適用する場合に適しています。</span><span class="sxs-lookup"><span data-stu-id="28028-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="28028-118">開発ツールで提供されるスキャフォールディング メカニズムにより、データ アクセス コードを非常に迅速に作成することができ、必要に応じてカスタマイズできます。</span><span class="sxs-lookup"><span data-stu-id="28028-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="28028-119">ただし、従来の CRUD アプローチには、いくつかの短所があります。</span><span class="sxs-lookup"><span data-stu-id="28028-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="28028-120">読み取りと書き込みのデータ表現が一致しないことがよくあります。具体的には、操作の一部としては必要ないものの、正しく更新しなければならない追加の列やプロパティなどです。</span><span class="sxs-lookup"><span data-stu-id="28028-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="28028-121">複数のアクターが同じデータ セットに対して操作を並列で実行するコラボレーション ドメインのデータ ストアでレコードがロックされている場合、データ競合のリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="28028-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="28028-122">また、オプティミスティック ロックを使用している場合、同時更新による更新の競合のリスクもあります。</span><span class="sxs-lookup"><span data-stu-id="28028-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="28028-123">システムの複雑さとスループットの増加に伴って、これらのリスクも増加します。</span><span class="sxs-lookup"><span data-stu-id="28028-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="28028-124">さらに、従来のアプローチは、データ ストアとデータ アクセス層への負荷、および情報を取得するために必要なクエリの複雑さによって、パフォーマンスに悪影響を及ぼす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="28028-125">各エンティティは読み取りと書き込みの両方の操作の対象となり、誤ったコンテキストでデータが公開されることがあるため、セキュリティとアクセス許可の管理が複雑化する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="28028-126">CRUD アプローチの制限の詳細については、[余裕がある場合にのみ CRUD を使用する方法](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="28028-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="28028-127">解決策</span><span class="sxs-lookup"><span data-stu-id="28028-127">Solution</span></span>

<span data-ttu-id="28028-128">コマンド クエリ責務分離 (CQRS) は、別のインターフェイスを使用することで、データを更新する操作 (コマンド) からデータを読み取る操作 (クエリ) を分離するパターンです。</span><span class="sxs-lookup"><span data-stu-id="28028-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="28028-129">つまり、クエリと更新にそれぞれ異なるデータ モデルを使用します。</span><span class="sxs-lookup"><span data-stu-id="28028-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="28028-130">絶対条件ではありませんが、次の図に示すようにモデルを分離できます。</span><span class="sxs-lookup"><span data-stu-id="28028-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![基本的な CQRS アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="28028-132">CQRS ベースのシステムでデータのクエリと更新の分離モデルを使用すると、CRUD ベースのシステムで使用される単一データ モデルと比べて、設計と実装が簡単になります。</span><span class="sxs-lookup"><span data-stu-id="28028-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="28028-133">ただし、CRUD 設計とは異なり、CQRS コードはスキャフォールディング メカニズムを使用して自動的に生成できないという短所があります。</span><span class="sxs-lookup"><span data-stu-id="28028-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="28028-134">データを読み取るためのクエリ モデルとデータを書き込むための更新モデルは、おそらく SQL ビューを使用するか、即時にプロジェクションを生成することにより、同じ物理ストアにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="28028-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="28028-135">ただし、パフォーマンス、スケーラビリティ、セキュリティを最大化するため、次の図に示すように、データを異なる物理ストアに分けるのが一般的です。</span><span class="sxs-lookup"><span data-stu-id="28028-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![読み取りストアと書き込みストアを分けた CQRS アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="28028-137">書き込みストアの読み取り専用レプリカを読み取りストアにすることも、読み取りストアと書き込みストアをまったく別の構造にすることもできます。</span><span class="sxs-lookup"><span data-stu-id="28028-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="28028-138">読み取りストアの読み取り専用レプリカを複数使用すると、読み取り専用レプリカがアプリケーション インスタンスの近くに配置されている分散シナリオでは特に、クエリのパフォーマンスとアプリケーション UI の応答性が大幅に向上します。</span><span class="sxs-lookup"><span data-stu-id="28028-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="28028-139">一部のデータベース システム (SQL Server) には、可用性を最大化するためのフェールオーバー レプリカなどの追加機能があります。</span><span class="sxs-lookup"><span data-stu-id="28028-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="28028-140">読み取りストアと書き込みストアを分離することにより、それぞれの負荷に合わせて適切にスケーリングすることもできます。</span><span class="sxs-lookup"><span data-stu-id="28028-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="28028-141">たとえば、読み取りストアには通常、書き込みストアよりはるかに高い負荷が発生します。</span><span class="sxs-lookup"><span data-stu-id="28028-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="28028-142">クエリ/読み取りモデルに非正規化データが含まれている場合 (「[具体化されたビュー パターン](materialized-view.md)」を参照)、パフォーマンスが最大化されるのは、アプリケーションの各ビューのデータを読み取るときと、システム内のデータを照会する実行するときです。</span><span class="sxs-lookup"><span data-stu-id="28028-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="28028-143">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="28028-143">Issues and considerations</span></span>

<span data-ttu-id="28028-144">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="28028-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="28028-145">データ ストアを読み取り操作用と書き込み操作用という別々の物理ストアに分けると、システムのパフォーマンスとセキュリティは向上する一方で、回復性と最終的な整合性の観点からは複雑さが増す可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="28028-146">読み取りモデル ストアは、書き込みモデル ストアへの変更を反映させるために更新する必要がありますが、ユーザーが古い読み取りデータに基づいた要求をいつ発行したのかを検出するのは容易ではなく、この場合、操作を完了できません。</span><span class="sxs-lookup"><span data-stu-id="28028-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="28028-147">最終的な整合性の詳細については、「[Data Consistency Primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28028-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="28028-148">システムの最も重要な、限られたセクションに CQRS を適用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="28028-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="28028-149">最終的な整合性をデプロイする一般的な方法は、イベント ソーシングと CQRS を併用することです。これにより、書き込みモデルは、コマンドの実行によって発生するイベントの追加専用ストリームになります。</span><span class="sxs-lookup"><span data-stu-id="28028-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="28028-150">これらのイベントは、読み取りモデルとして機能する具体化されたビューの更新に使用されます。</span><span class="sxs-lookup"><span data-stu-id="28028-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="28028-151">詳細については、「[Event Sourcing and CQRS (イベント ソーシングと CQRS)](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28028-151">For more information see [Event Sourcing and CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="28028-152">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="28028-152">When to use this pattern</span></span>

<span data-ttu-id="28028-153">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="28028-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="28028-154">同じデータに対して複数の操作が並列で実行されるコラボレーション ドメイン。</span><span class="sxs-lookup"><span data-stu-id="28028-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="28028-155">CQRS を使用すると、同じ種類のデータに見えるものを更新する場合でも、ドメイン レベルでマージの競合を最小化するのに十分な細分性でコマンドを定義できます (発生したすべての競合をコマンドでマージできます)。</span><span class="sxs-lookup"><span data-stu-id="28028-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="28028-156">一連の手順として、または複雑なドメイン モデルを使用して、複雑なプロセスがユーザーにされるタスクベースのユーザー インターフェイス。</span><span class="sxs-lookup"><span data-stu-id="28028-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="28028-157">既にドメインベースの設計 (DDD) 手法を使い慣れているチームにも有用です。</span><span class="sxs-lookup"><span data-stu-id="28028-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="28028-158">書き込みモデルには、ビジネス ロジック、入力の検証、ビジネスの検証を使用する完全なコマンド処理スタックがあり、その書き込みモデル内で各集計 (データ変更の単位として処理される関連オブジェクトの各クラスター) についてすべてが一貫していることを保証します。</span><span class="sxs-lookup"><span data-stu-id="28028-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="28028-159">読み取りモデルにはビジネス ロジックや検証スタックはなく、ビュー モデルで使用する DTO を返すだけです。</span><span class="sxs-lookup"><span data-stu-id="28028-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="28028-160">読み取りモデルは、最終的には書き込みモデルと一致します。</span><span class="sxs-lookup"><span data-stu-id="28028-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="28028-161">データ読み取りのパフォーマンスを、データ書き込みのパフォーマンスとは別に細かく調整する必要があるシナリオ (特に、読み取り/書き込みの比率が非常に高く、水平スケーリングが必要な場合)。</span><span class="sxs-lookup"><span data-stu-id="28028-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="28028-162">たとえば、多くのシステムで、読み取り操作の回数は書き込み操作の回数の数倍になります。</span><span class="sxs-lookup"><span data-stu-id="28028-162">For example, in many systems the number of read operations is many times greater that the number of write operations.</span></span> <span data-ttu-id="28028-163">これに対応できるように、読み取りモデルをスケール アウトし、書き込みモデルは 1 つまたは少数のインスタンスでのみ実行することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="28028-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="28028-164">書き込みモデルのインスタンス数を少なくすることは、マージ競合の発生を最小化するうえでも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="28028-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="28028-165">1 つの開発者チームが書き込みモデルの一部である複雑なドメイン モデルに注力し、もう 1 つのチームが読み取りモデルとユーザー インターフェイスに注力できるシナリオ。</span><span class="sxs-lookup"><span data-stu-id="28028-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="28028-166">システムが時間の経過に伴って進化し、モデルの複数のバージョンを含むようになることが予測されるシナリオや、ビジネス ルールが定期的に変更されるシナリオ。</span><span class="sxs-lookup"><span data-stu-id="28028-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="28028-167">他のシステムとの統合 (特にイベント ソーシングとの組み合わせ)。この場合、1 つのサブシステムの一時的なエラーは、他のサブシステムの可用性に影響しません。</span><span class="sxs-lookup"><span data-stu-id="28028-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="28028-168">次の状況では、このパターンはお勧めしません。</span><span class="sxs-lookup"><span data-stu-id="28028-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="28028-169">ドメインやビジネス ルールが単純な場合。</span><span class="sxs-lookup"><span data-stu-id="28028-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="28028-170">単純な CRUD スタイルのユーザー インターフェイスと、関連するデータ アクセス操作で十分な場合。</span><span class="sxs-lookup"><span data-stu-id="28028-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="28028-171">システム全体にまたがる実装の場合。</span><span class="sxs-lookup"><span data-stu-id="28028-171">For implementation across the whole system.</span></span> <span data-ttu-id="28028-172">CQRS が役立つ全体的なデータ管理シナリオに特有のコンポーネントもあるものの、必要ない場合には煩雑さが増してしまう可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="28028-173">イベント ソーシングと CQRS</span><span class="sxs-lookup"><span data-stu-id="28028-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="28028-174">CQRS パターンは、イベント ソーシング パターンと共によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="28028-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="28028-175">CQRS ベースのシステムは、個別の読み取りデータ モデルと書き込みデータ モデルを使用します。これらはそれぞれ関連するタスクに合わせて調整されており、多くの場合、物理的に分離されたストアに存在します。</span><span class="sxs-lookup"><span data-stu-id="28028-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="28028-176">[イベント ソーシング](event-sourcing.md) パターンと共に使用される場合、イベントのストアは書き込みモデルであり、公式の情報ソースです。</span><span class="sxs-lookup"><span data-stu-id="28028-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="28028-177">CQRS ベースのシステムの読み取りモデルは、データの具体化されたビュー (通常は高度に非正規化されたビュー) を提供します。</span><span class="sxs-lookup"><span data-stu-id="28028-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="28028-178">これらのビューは、アプリケーションのインターフェイスとディスプレイの要件に合わせて調整されており、ディスプレイとクエリの両方のパフォーマンスを最大化するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="28028-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="28028-179">ある時点での実際のデータではなく、イベントのストリームを書き込みストアとして使用することにより、単一の集計での更新の競合を回避し、パフォーマンスとスケーラビリティを最大化します。</span><span class="sxs-lookup"><span data-stu-id="28028-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="28028-180">イベントを使用して、読み取りストアへのデータ入力に使用されるデータの具体化されたビューを非同期的に生成できます。</span><span class="sxs-lookup"><span data-stu-id="28028-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="28028-181">イベント ストアは公式の情報ソースであるため、システムが進化したり読み取りモデルの変更が必要になったりした場合に、具体化されたビューを削除し、過去のすべてのイベントを再生して最新の状態の新しい表現を作成することができます。</span><span class="sxs-lookup"><span data-stu-id="28028-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="28028-182">具体化されたビューは、実質的にはデータの持続的な読み取り専用キャッシュです。</span><span class="sxs-lookup"><span data-stu-id="28028-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="28028-183">CQRS とイベント ソーシング パターンを組み合わせて使用する場合、次の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="28028-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="28028-184">書き込みストアと読み取りストアが分離しているすべてのシステムと同様に、このパターンに基づくシステムは、最後の段階にならないと一貫性が確保されません。</span><span class="sxs-lookup"><span data-stu-id="28028-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="28028-185">イベントの生成とデータ ストアの更新の間には、いくらかの遅延があります。</span><span class="sxs-lookup"><span data-stu-id="28028-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="28028-186">イベントを開始および処理し、クエリや読み取りモデルで必要な適切なビューやオブジェクトをアセンブルまたは更新するようにコードを作成する必要があるため、このパターンでは複雑さが増します。</span><span class="sxs-lookup"><span data-stu-id="28028-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="28028-187">CQRS パターンをイベント ソーシング パターンと併用すると複雑さが増すため、実装が難しくなる可能性があり、システム設計に別のアプローチが必要になります。</span><span class="sxs-lookup"><span data-stu-id="28028-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="28028-188">ただし、イベント ソーシングを使用するとドメインのモデル化が容易になります。また、データの変更の目的が保持されるため、ビューの再構築や新規作成も容易になります。</span><span class="sxs-lookup"><span data-stu-id="28028-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="28028-189">特定のエンティティまたはエンティティのコレクションのイベントを再生または処理することにより、データの読み取りモデルまたはプロジェクションで使用する具体化されたビューを生成すると、大量の処理時間とリソース使用量が必要になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="28028-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="28028-190">これは特に、長期にわたる値の合計や解析が必要な場合に当てはまります。関連するすべてのイベントの検証が必要な場合があるためです。</span><span class="sxs-lookup"><span data-stu-id="28028-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="28028-191">この問題を解決するには、スケジュールされた間隔 (発生した特定のアクションの合計数、エンティティの現在の状態など) でデータのスナップショットを実装します。</span><span class="sxs-lookup"><span data-stu-id="28028-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="28028-192">例</span><span class="sxs-lookup"><span data-stu-id="28028-192">Example</span></span>

<span data-ttu-id="28028-193">次のコードは、読み取りモデルと書き込みモデルに異なる定義を使用する CQRS 実装の例から抽出したものです。</span><span class="sxs-lookup"><span data-stu-id="28028-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="28028-194">モデル インターフェイスは、基になるデータ ストアの機能に影響しません。また、進化することができ、インターフェイスどうしが分離しているため個別に微調整もできます。</span><span class="sxs-lookup"><span data-stu-id="28028-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="28028-195">次のコードは、読み取りモデルの定義を示しています。</span><span class="sxs-lookup"><span data-stu-id="28028-195">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="28028-196">ユーザーは製品を評価することができます。</span><span class="sxs-lookup"><span data-stu-id="28028-196">The system allows users to rate products.</span></span> <span data-ttu-id="28028-197">そのためには、次のコードに示すように、アプリケーション コードで `RateProduct` コマンドを使用します。</span><span class="sxs-lookup"><span data-stu-id="28028-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="28028-198">システムは `ProductsCommandHandler` クラスを使用して、アプリケーションから送信されたコマンドを処理します。</span><span class="sxs-lookup"><span data-stu-id="28028-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="28028-199">クライアントは通常、キューなどのメッセージング システムを使用して、ドメインにコマンドを送信します。</span><span class="sxs-lookup"><span data-stu-id="28028-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="28028-200">コマンド ハンドラーはこれらのコマンドを受け入れ、ドメイン インターフェイスのメソッドを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="28028-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="28028-201">各コマンドの細分性は、要求の競合が発生する可能性が少なくなるように設計されています。</span><span class="sxs-lookup"><span data-stu-id="28028-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="28028-202">次のコードは、`ProductsCommandHandler` クラスのアウトラインを示しています。</span><span class="sxs-lookup"><span data-stu-id="28028-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="28028-203">次のコードは、書き込みモデルからの `IProductsDomain` インターフェイスを示しています。</span><span class="sxs-lookup"><span data-stu-id="28028-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="28028-204">ドメイン内で意味を持つメソッドが `IProductsDomain` インターフェイスにどのように含まれているかにも注意してください。</span><span class="sxs-lookup"><span data-stu-id="28028-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="28028-205">通常、これらのメソッドは CRUD 環境では `Save` や `Update` などの汎用的な名前を持ち、DTO が唯一の引数です。</span><span class="sxs-lookup"><span data-stu-id="28028-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="28028-206">CQRS アプローチは、この組織のビジネスと在庫管理システムのニーズを満たすように設計できます。</span><span class="sxs-lookup"><span data-stu-id="28028-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="28028-207">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="28028-207">Related patterns and guidance</span></span>

<span data-ttu-id="28028-208">このパターンを実装する場合、次のパターンとガイダンスが役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="28028-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="28028-209">CQRS と他のアーキテクチャ スタイルとの比較については、「[アーキテクチャ スタイル](/azure/architecture/guide/architecture-styles/)」と「[CQRS アーキテクチャのスタイル](/azure/architecture/guide/architecture-styles/cqrs)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28028-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="28028-210">[Data consistency primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)。</span><span class="sxs-lookup"><span data-stu-id="28028-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="28028-211">CQRS パターン使用時に読み取りデータ ストアと書き込みデータ ストアの間の最終的な整合性が原因で通常発生する問題と、これらの問題の解決方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="28028-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="28028-212">[データのパーティション分割のガイダンス](https://msdn.microsoft.com/library/dn589795.aspx)。</span><span class="sxs-lookup"><span data-stu-id="28028-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="28028-213">CQRS パターンで使用される読み取りデータ ストアと書き込みデータ ストアを、個別に管理およびアクセス可能なパーティションに分割して、スケーラビリティの向上、競合の削減、パフォーマンスの最適化を図る方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="28028-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="28028-214">[イベント ソーシング パターン](event-sourcing.md)。</span><span class="sxs-lookup"><span data-stu-id="28028-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="28028-215">イベント ソーシングを CQRS パターンと共に使用して、複雑なドメインでのタスクを簡略化しながら、パフォーマンス、スケーラビリティ、応答性を向上させる方法について、さらに詳しく説明します。</span><span class="sxs-lookup"><span data-stu-id="28028-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="28028-216">補正アクションを有効にできる完全な監査証跡と履歴を保持しながら、トランザクション データの整合性を提供する方法についても説明します。</span><span class="sxs-lookup"><span data-stu-id="28028-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="28028-217">[具体化されたビュー パターン](materialized-view.md)。</span><span class="sxs-lookup"><span data-stu-id="28028-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="28028-218">CQRS 実装の読み取りモデルには、書き込みモデル データの具体化されたビューを含めることができます。また、読み取りモデルは具体化されたビューの生成に使用できます。</span><span class="sxs-lookup"><span data-stu-id="28028-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="28028-219">パターンとプラクティスのガイド「[CQRS Journey (CQRS の旅)](http://aka.ms/cqrs)」。</span><span class="sxs-lookup"><span data-stu-id="28028-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="28028-220">具体的には、「[Introducing the Command Query Responsibility Segregation Pattern (コマンド クエリ責務分離パターンの概要)](https://msdn.microsoft.com/library/jj591573.aspx)」でパターンとそのパターンが役立つ状況について説明します。「[Epilogue: Lessons Learned (エピローグ: 得られた教訓)](https://msdn.microsoft.com/library/jj591568.aspx)」は、このパターンを使用したときに発生する問題の一部を理解するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="28028-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="28028-221">Martin Fowler の投稿「[CQRS](http://martinfowler.com/bliki/CQRS.html)」では、パターンの基本と他の有用なリソースへのリンクを紹介しています。</span><span class="sxs-lookup"><span data-stu-id="28028-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="28028-222">[Greg Young の投稿](http://codebetter.com/gregyoung/)では、CQRS パターンのさまざまな側面について説明しています。</span><span class="sxs-lookup"><span data-stu-id="28028-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>