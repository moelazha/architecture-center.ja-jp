---
title: "バッチ処理テクノロジの選択"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bfb850ee8e9d8fd41927b4ca3b612e15b5ae6b11
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a><span data-ttu-id="9ea32-102">Azure で使用するバッチ処理テクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="9ea32-102">Choosing a batch processing technology in Azure</span></span>

<span data-ttu-id="9ea32-103">ビッグ データ ソリューションでは、長時間実行されるバッチ ジョブを使用して、データのフィルター処理、集計、または分析の準備を行うことがよくあります。</span><span class="sxs-lookup"><span data-stu-id="9ea32-103">Big data solutions often use long-running batch jobs to filter, aggregate, and otherwise prepare the data for analysis.</span></span> <span data-ttu-id="9ea32-104">通常、このようなジョブには、スケーラブル ストレージ (HDFS、Azure Data Lake Store、Azure Storage など) からソース ファイルを読み取り、処理し、出力をスケーラブル ストレージの新しいファイルに書き込む作業が含まれます。</span><span class="sxs-lookup"><span data-stu-id="9ea32-104">Usually these jobs involve reading source files from scalable storage (like HDFS, Azure Data Lake Store, and Azure Storage), processing them, and writing the output to new files in scalable storage.</span></span> 

<span data-ttu-id="9ea32-105">このようなバッチ処理エンジンの重要な要件は、大量のデータを処理するために、計算をスケールアウトできることです。</span><span class="sxs-lookup"><span data-stu-id="9ea32-105">The key requirement of such batch processing engines is the ability to scale out computations, in order to handle a large volume of data.</span></span> <span data-ttu-id="9ea32-106">ただし、リアルタイム処理とは異なり、バッチ処理の場合、分単位から時間単位の待機時間 (データ インジェストから結果の計算までの時間) が生じることが予想されます。</span><span class="sxs-lookup"><span data-stu-id="9ea32-106">Unlike real-time processing, however, batch processing is expected to have latencies (the time between data ingestion and computing a result) that measure in minutes to hours.</span></span>

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a><span data-ttu-id="9ea32-107">バッチ処理テクノロジを選択する場合のオプション</span><span class="sxs-lookup"><span data-stu-id="9ea32-107">What are your options when choosing a batch processing technology?</span></span>

<span data-ttu-id="9ea32-108">Azure では、以下のすべてのデータ ストアがバッチ処理のコア要件を満たしています。</span><span class="sxs-lookup"><span data-stu-id="9ea32-108">In Azure, all of the following data stores will meet the core requirements for batch processing:</span></span>

- [<span data-ttu-id="9ea32-109">Azure Data Lake Analytics</span><span class="sxs-lookup"><span data-stu-id="9ea32-109">Azure Data Lake Analytics</span></span>](/azure/data-lake-analytics/)
- [<span data-ttu-id="9ea32-110">Azure SQL Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="9ea32-110">Azure SQL Data Warehouse</span></span>](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [<span data-ttu-id="9ea32-111">Spark を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-111">HDInsight with Spark</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="9ea32-112">Hive を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-112">HDInsight with Hive</span></span>](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [<span data-ttu-id="9ea32-113">Hive LLAP を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-113">HDInsight with Hive LLAP</span></span>](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a><span data-ttu-id="9ea32-114">主要な選択条件</span><span class="sxs-lookup"><span data-stu-id="9ea32-114">Key selection criteria</span></span>

<span data-ttu-id="9ea32-115">選択肢を絞り込むために、まず次の質問に答えてください。</span><span class="sxs-lookup"><span data-stu-id="9ea32-115">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="9ea32-116">独自のサーバーを管理するのではなく、マネージド サービスを使用しますか。</span><span class="sxs-lookup"><span data-stu-id="9ea32-116">Do you want a managed service rather than managing your own servers?</span></span>

- <span data-ttu-id="9ea32-117">バッチ処理ロジックの作成は宣言型と命令型のどちらですか。</span><span class="sxs-lookup"><span data-stu-id="9ea32-117">Do you want to author batch processing logic declaratively or imperatively?</span></span>

- <span data-ttu-id="9ea32-118">大量にバッチ処理を実行しますか。</span><span class="sxs-lookup"><span data-stu-id="9ea32-118">Will you perform batch processing in bursts?</span></span> <span data-ttu-id="9ea32-119">"はい" の場合は、クラスターを一時停止するか、バッチ ジョブごとの価格モデルを選択することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9ea32-119">If yes, consider options that let you pause the cluster or whose pricing model is per batch job.</span></span>

- <span data-ttu-id="9ea32-120">参照データを検索するなど、バッチ処理と共にリレーショナル データ ストアに対してクエリを実行する必要はありますか。</span><span class="sxs-lookup"><span data-stu-id="9ea32-120">Do you need to query relational data stores along with your batch processing, for example to look up reference data?</span></span> <span data-ttu-id="9ea32-121">"はい" の場合は、外部リレーショナル ストアのクエリ処理が可能なオプションを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9ea32-121">If yes, consider the options that enable querying of external relational stores.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="9ea32-122">機能のマトリックス</span><span class="sxs-lookup"><span data-stu-id="9ea32-122">Capability matrix</span></span>

<span data-ttu-id="9ea32-123">次の表は、機能の主な相違点をまとめたものです。</span><span class="sxs-lookup"><span data-stu-id="9ea32-123">The following tables summarize the key differences in capabilities.</span></span> 

### <a name="general-capabilities"></a><span data-ttu-id="9ea32-124">一般的な機能</span><span class="sxs-lookup"><span data-stu-id="9ea32-124">General capabilities</span></span>

| | <span data-ttu-id="9ea32-125">Azure Data Lake Analytics</span><span class="sxs-lookup"><span data-stu-id="9ea32-125">Azure Data Lake Analytics</span></span> | <span data-ttu-id="9ea32-126">Azure SQL Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="9ea32-126">Azure SQL Data Warehouse</span></span> | <span data-ttu-id="9ea32-127">Spark を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-127">HDInsight with Spark</span></span> | <span data-ttu-id="9ea32-128">Hive を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-128">HDInsight with Hive</span></span> | <span data-ttu-id="9ea32-129">Hive LLAP を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-129">HDInsight with Hive LLAP</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9ea32-130">マネージド サービスか</span><span class="sxs-lookup"><span data-stu-id="9ea32-130">Is managed service</span></span> | <span data-ttu-id="9ea32-131">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-131">Yes</span></span> | <span data-ttu-id="9ea32-132">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-132">Yes</span></span> | <span data-ttu-id="9ea32-133">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-133">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-134">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-134">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-135">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-135">Yes <sup>1</sup></span></span> |
| <span data-ttu-id="9ea32-136">計算の一時停止をサポート</span><span class="sxs-lookup"><span data-stu-id="9ea32-136">Supports pausing compute</span></span> | <span data-ttu-id="9ea32-137">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-137">No</span></span> | <span data-ttu-id="9ea32-138">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-138">Yes</span></span> | <span data-ttu-id="9ea32-139">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-139">No</span></span> | <span data-ttu-id="9ea32-140">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-140">No</span></span> | <span data-ttu-id="9ea32-141">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-141">No</span></span> |
| <span data-ttu-id="9ea32-142">リレーショナル データ ストア</span><span class="sxs-lookup"><span data-stu-id="9ea32-142">Relational data store</span></span> | <span data-ttu-id="9ea32-143">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-143">Yes</span></span> | <span data-ttu-id="9ea32-144">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-144">Yes</span></span> | <span data-ttu-id="9ea32-145">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-145">No</span></span> | <span data-ttu-id="9ea32-146">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-146">No</span></span> | <span data-ttu-id="9ea32-147">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-147">No</span></span> |
| <span data-ttu-id="9ea32-148">プログラミング</span><span class="sxs-lookup"><span data-stu-id="9ea32-148">Programmability</span></span> | <span data-ttu-id="9ea32-149">U-SQL</span><span class="sxs-lookup"><span data-stu-id="9ea32-149">U-SQL</span></span> | <span data-ttu-id="9ea32-150">T-SQL</span><span class="sxs-lookup"><span data-stu-id="9ea32-150">T-SQL</span></span> | <span data-ttu-id="9ea32-151">Python、Scala、Java、R</span><span class="sxs-lookup"><span data-stu-id="9ea32-151">Python, Scala, Java, R</span></span> | <span data-ttu-id="9ea32-152">HiveQL</span><span class="sxs-lookup"><span data-stu-id="9ea32-152">HiveQL</span></span> | <span data-ttu-id="9ea32-153">HiveQL</span><span class="sxs-lookup"><span data-stu-id="9ea32-153">HiveQL</span></span> |
| <span data-ttu-id="9ea32-154">プログラミング パラダイム</span><span class="sxs-lookup"><span data-stu-id="9ea32-154">Programming paradigm</span></span> | <span data-ttu-id="9ea32-155">宣言型と命令型の混合</span><span class="sxs-lookup"><span data-stu-id="9ea32-155">Mixture of declarative and imperative</span></span>  | <span data-ttu-id="9ea32-156">宣言型</span><span class="sxs-lookup"><span data-stu-id="9ea32-156">Declarative</span></span> | <span data-ttu-id="9ea32-157">宣言型と命令型の混合</span><span class="sxs-lookup"><span data-stu-id="9ea32-157">Mixture of declarative and imperative</span></span> | <span data-ttu-id="9ea32-158">宣言型</span><span class="sxs-lookup"><span data-stu-id="9ea32-158">Declarative</span></span> | <span data-ttu-id="9ea32-159">宣言型</span><span class="sxs-lookup"><span data-stu-id="9ea32-159">Declarative</span></span> | 
| <span data-ttu-id="9ea32-160">価格モデル</span><span class="sxs-lookup"><span data-stu-id="9ea32-160">Pricing model</span></span> | <span data-ttu-id="9ea32-161">バッチ ジョブごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-161">Per batch job</span></span> | <span data-ttu-id="9ea32-162">クラスター時間単位</span><span class="sxs-lookup"><span data-stu-id="9ea32-162">By cluster hour</span></span> | <span data-ttu-id="9ea32-163">クラスター時間単位</span><span class="sxs-lookup"><span data-stu-id="9ea32-163">By cluster hour</span></span> | <span data-ttu-id="9ea32-164">クラスター時間単位</span><span class="sxs-lookup"><span data-stu-id="9ea32-164">By cluster hour</span></span> | <span data-ttu-id="9ea32-165">クラスター時間単位</span><span class="sxs-lookup"><span data-stu-id="9ea32-165">By cluster hour</span></span> |  

<span data-ttu-id="9ea32-166">[1] 手動構成とスケーリングを使用。</span><span class="sxs-lookup"><span data-stu-id="9ea32-166">[1] With manual configuration and scaling.</span></span>
 
### <a name="integration-capabilities"></a><span data-ttu-id="9ea32-167">統合機能</span><span class="sxs-lookup"><span data-stu-id="9ea32-167">Integration capabilities</span></span>
| | <span data-ttu-id="9ea32-168">Azure Data Lake Analytics</span><span class="sxs-lookup"><span data-stu-id="9ea32-168">Azure Data Lake Analytics</span></span> | <span data-ttu-id="9ea32-169">SQL Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="9ea32-169">SQL Data Warehouse</span></span> | <span data-ttu-id="9ea32-170">Spark を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-170">HDInsight with Spark</span></span> | <span data-ttu-id="9ea32-171">Hive を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-171">HDInsight with Hive</span></span> | <span data-ttu-id="9ea32-172">Hive LLAP を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-172">HDInsight with Hive LLAP</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9ea32-173">Azure Data Lake Store からのアクセス</span><span class="sxs-lookup"><span data-stu-id="9ea32-173">Access from Azure Data Lake Store</span></span> | <span data-ttu-id="9ea32-174">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-174">Yes</span></span> | <span data-ttu-id="9ea32-175">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-175">Yes</span></span> | <span data-ttu-id="9ea32-176">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-176">Yes</span></span> | <span data-ttu-id="9ea32-177">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-177">Yes</span></span> | <span data-ttu-id="9ea32-178">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-178">Yes</span></span> |
| <span data-ttu-id="9ea32-179">Azure Storage からのクエリ</span><span class="sxs-lookup"><span data-stu-id="9ea32-179">Query from Azure Storage</span></span> | <span data-ttu-id="9ea32-180">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-180">Yes</span></span> | <span data-ttu-id="9ea32-181">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-181">Yes</span></span> | <span data-ttu-id="9ea32-182">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-182">Yes</span></span> | <span data-ttu-id="9ea32-183">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-183">Yes</span></span> | <span data-ttu-id="9ea32-184">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-184">Yes</span></span> |
| <span data-ttu-id="9ea32-185">外部リレーショナル ストアからのクエリ</span><span class="sxs-lookup"><span data-stu-id="9ea32-185">Query from external relational stores</span></span> | <span data-ttu-id="9ea32-186">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-186">Yes</span></span> | <span data-ttu-id="9ea32-187">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-187">No</span></span> | <span data-ttu-id="9ea32-188">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-188">Yes</span></span> | <span data-ttu-id="9ea32-189">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-189">No</span></span> | <span data-ttu-id="9ea32-190">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-190">No</span></span> |

### <a name="scalability-capabilities"></a><span data-ttu-id="9ea32-191">スケーラビリティ機能</span><span class="sxs-lookup"><span data-stu-id="9ea32-191">Scalability capabilities</span></span>
| | <span data-ttu-id="9ea32-192">Azure Data Lake Analytics</span><span class="sxs-lookup"><span data-stu-id="9ea32-192">Azure Data Lake Analytics</span></span> | <span data-ttu-id="9ea32-193">SQL Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="9ea32-193">SQL Data Warehouse</span></span> | <span data-ttu-id="9ea32-194">Spark を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-194">HDInsight with Spark</span></span> | <span data-ttu-id="9ea32-195">Hive を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-195">HDInsight with Hive</span></span> | <span data-ttu-id="9ea32-196">Hive LLAP を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-196">HDInsight with Hive LLAP</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9ea32-197">スケールアウトの細分性</span><span class="sxs-lookup"><span data-stu-id="9ea32-197">Scale-out granularity</span></span>  | <span data-ttu-id="9ea32-198">ジョブごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-198">Per job</span></span> | <span data-ttu-id="9ea32-199">クラスターごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-199">Per cluster</span></span> | <span data-ttu-id="9ea32-200">クラスターごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-200">Per cluster</span></span> | <span data-ttu-id="9ea32-201">クラスターごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-201">Per cluster</span></span> | <span data-ttu-id="9ea32-202">クラスターごと</span><span class="sxs-lookup"><span data-stu-id="9ea32-202">Per cluster</span></span> |
| <span data-ttu-id="9ea32-203">高速スケールアウト (1 分未満)</span><span class="sxs-lookup"><span data-stu-id="9ea32-203">Fast scale out (less than 1 minute)</span></span> | <span data-ttu-id="9ea32-204">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-204">Yes</span></span> | <span data-ttu-id="9ea32-205">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-205">Yes</span></span> | <span data-ttu-id="9ea32-206">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-206">No</span></span> | <span data-ttu-id="9ea32-207">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-207">No</span></span> | <span data-ttu-id="9ea32-208">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-208">No</span></span> |
| <span data-ttu-id="9ea32-209">データのメモリ内キャッシュ</span><span class="sxs-lookup"><span data-stu-id="9ea32-209">In-memory caching of data</span></span> | <span data-ttu-id="9ea32-210">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-210">No</span></span> | <span data-ttu-id="9ea32-211">可能 </span><span class="sxs-lookup"><span data-stu-id="9ea32-211">Yes</span></span> | <span data-ttu-id="9ea32-212">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-212">Yes</span></span> | <span data-ttu-id="9ea32-213">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-213">No</span></span> | <span data-ttu-id="9ea32-214">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-214">Yes</span></span> | 

### <a name="security-capabilities"></a><span data-ttu-id="9ea32-215">セキュリティ機能</span><span class="sxs-lookup"><span data-stu-id="9ea32-215">Security capabilities</span></span>
| | <span data-ttu-id="9ea32-216">Azure Data Lake Analytics</span><span class="sxs-lookup"><span data-stu-id="9ea32-216">Azure Data Lake Analytics</span></span> | <span data-ttu-id="9ea32-217">SQL Data Warehouse</span><span class="sxs-lookup"><span data-stu-id="9ea32-217">SQL Data Warehouse</span></span> | <span data-ttu-id="9ea32-218">Spark を使用する HDInsight</span><span class="sxs-lookup"><span data-stu-id="9ea32-218">HDInsight with Spark</span></span> | <span data-ttu-id="9ea32-219">HDInsight 上の Apache Hive</span><span class="sxs-lookup"><span data-stu-id="9ea32-219">Apache Hive on HDInsight</span></span> | <span data-ttu-id="9ea32-220">HDInsight 上の Hive LLAP</span><span class="sxs-lookup"><span data-stu-id="9ea32-220">Hive LLAP on HDInsight</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="9ea32-221">認証</span><span class="sxs-lookup"><span data-stu-id="9ea32-221">Authentication</span></span>  | <span data-ttu-id="9ea32-222">Azure Active Directory (Azure AD)</span><span class="sxs-lookup"><span data-stu-id="9ea32-222">Azure Active Directory (Azure AD)</span></span> | <span data-ttu-id="9ea32-223">SQL / Azure AD</span><span class="sxs-lookup"><span data-stu-id="9ea32-223">SQL / Azure AD</span></span> | <span data-ttu-id="9ea32-224">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-224">No</span></span> | <span data-ttu-id="9ea32-225">ローカル / Azure AD <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-225">local / Azure AD <sup>1</sup></span></span> | <span data-ttu-id="9ea32-226">ローカル / Azure AD <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-226">local / Azure AD <sup>1</sup></span></span> |
| <span data-ttu-id="9ea32-227">承認</span><span class="sxs-lookup"><span data-stu-id="9ea32-227">Authorization</span></span>  | <span data-ttu-id="9ea32-228">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-228">Yes</span></span> | <span data-ttu-id="9ea32-229">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-229">Yes</span></span>| <span data-ttu-id="9ea32-230">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-230">No</span></span> | <span data-ttu-id="9ea32-231">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-231">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-232">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-232">Yes <sup>1</sup></span></span> |
| <span data-ttu-id="9ea32-233">監査</span><span class="sxs-lookup"><span data-stu-id="9ea32-233">Auditing</span></span>  | <span data-ttu-id="9ea32-234">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-234">Yes</span></span> | <span data-ttu-id="9ea32-235">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-235">Yes</span></span> | <span data-ttu-id="9ea32-236">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-236">No</span></span> | <span data-ttu-id="9ea32-237">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-237">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-238">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-238">Yes <sup>1</sup></span></span> |
| <span data-ttu-id="9ea32-239">保存データの暗号化</span><span class="sxs-lookup"><span data-stu-id="9ea32-239">Data encryption at rest</span></span> | <span data-ttu-id="9ea32-240">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-240">Yes</span></span>| <span data-ttu-id="9ea32-241">はい <sup>2</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-241">Yes <sup>2</sup></span></span> | <span data-ttu-id="9ea32-242">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-242">Yes</span></span> | <span data-ttu-id="9ea32-243">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-243">Yes</span></span> | <span data-ttu-id="9ea32-244">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-244">Yes</span></span> |
| <span data-ttu-id="9ea32-245">行レベルのセキュリティ</span><span class="sxs-lookup"><span data-stu-id="9ea32-245">Row-level security</span></span> | <span data-ttu-id="9ea32-246">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-246">No</span></span> | <span data-ttu-id="9ea32-247">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-247">Yes</span></span> | <span data-ttu-id="9ea32-248">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-248">No</span></span> | <span data-ttu-id="9ea32-249">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-249">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-250">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-250">Yes <sup>1</sup></span></span> |
| <span data-ttu-id="9ea32-251">ファイアウォールをサポート</span><span class="sxs-lookup"><span data-stu-id="9ea32-251">Supports firewalls</span></span> | <span data-ttu-id="9ea32-252">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-252">Yes</span></span> | <span data-ttu-id="9ea32-253">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-253">Yes</span></span> | <span data-ttu-id="9ea32-254">[はい]</span><span class="sxs-lookup"><span data-stu-id="9ea32-254">Yes</span></span> | <span data-ttu-id="9ea32-255">はい <sup>3</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-255">Yes <sup>3</sup></span></span> | <span data-ttu-id="9ea32-256">はい <sup>3</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-256">Yes <sup>3</sup></span></span> |
| <span data-ttu-id="9ea32-257">動的データ マスク</span><span class="sxs-lookup"><span data-stu-id="9ea32-257">Dynamic data masking</span></span> | <span data-ttu-id="9ea32-258">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-258">No</span></span> | <span data-ttu-id="9ea32-259">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-259">No</span></span> | <span data-ttu-id="9ea32-260">いいえ </span><span class="sxs-lookup"><span data-stu-id="9ea32-260">No</span></span> | <span data-ttu-id="9ea32-261">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-261">Yes <sup>1</sup></span></span> | <span data-ttu-id="9ea32-262">はい <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="9ea32-262">Yes <sup>1</sup></span></span> |

<span data-ttu-id="9ea32-263">[1] [ドメイン参加済み HDInsight クラスター](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9ea32-263">[1] Requires using a [domain-joined HDInsight cluster](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).</span></span>

<span data-ttu-id="9ea32-264">[2] 保存データの暗号化と暗号化の解除には、Transparent Data Encryption (TDE) を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9ea32-264">[2] Requires using Transparent Data Encryption (TDE) to encrypt and decrypt your data at rest.</span></span>

<span data-ttu-id="9ea32-265">[3] [Azure Virtual Network 内で使用する](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)場合にサポートされます。</span><span class="sxs-lookup"><span data-stu-id="9ea32-265">[3] Supported when [used within an Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).</span></span>