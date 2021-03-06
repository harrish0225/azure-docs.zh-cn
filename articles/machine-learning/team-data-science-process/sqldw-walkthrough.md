---
title: 使用 Azure Synapse Analytics 构建和部署模型-团队数据科学流程
description: 使用 Azure Synapse Analytics 和公开可用的数据集，生成和部署机器学习模型。
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 9c4c1cfdb927cfd2ee607bfe2a951e06c80f9bfb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "81418535"
---
# <a name="the-team-data-science-process-in-action-using-azure-synapse-analytics"></a>工作中的团队数据科学过程：使用 Azure Synapse Analytics
在本教程中，我们将逐步介绍如何使用 Azure Synapse Analytics 对公开提供的数据集（ [NYC 出租车行程](https://www.andresmh.com/nyctaxitrips/)数据集）生成和部署机器学习模型。 构造的二元分类模型预测是否为行程支付了提示。  模型包括多类分类（无论是否有提示）和回归（支付的刀尖数量的分布）。

该过程遵循[团队数据科学过程 (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/) 工作流。 本文介绍如何设置数据科学环境，如何将数据加载到 Azure Synapse Analytics 中，以及如何使用 Azure Synapse Analytics 或 IPython 笔记本浏览要建模的数据和工程功能。 然后，我们会介绍如何使用 Azure 机器学习来构建和部署模型。

## <a name="the-nyc-taxi-trips-dataset"></a><a name="dataset"></a>NYC 出租车行程数据集
NYC 出租车车程数据包含大约 20 GB（未压缩约为 48 GB）的压缩 CSV 文件，记录了超过 1.73 亿个单独车程及每个车程支付的费用。 每个行程记录都包括分拣和下车位置和时间、匿名黑客（驾照）许可证编号和 medallion （出租车的唯一 ID）编号。 数据涵盖  2013 年的所有行程，并在每个月的以下两个数据集中提供：

1. **trip_data.csv** 文件包含行程的详细信息，例如乘客编号、上车和下车时间、行程持续时间和行程距离。 下面是一些示例记录：

        medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868
2. **trip_fare.csv** 文件包含每个行程费用的详细信息，例如付款类型、费用金额、附加税和税金、小费和通行税以及支付的总金额等。 下面是一些示例记录：

        medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

用于联接 trip\_data 和 trip\_fare 的**唯一键**由以下三个字段组成︰

* medallion、
* hack\_license 和 
* pickup\_datetime。

## <a name="address-three-types-of-prediction-tasks"></a><a name="mltasks"></a>解决三种类型的预测任务
我们根据 *tip\_amount* 编写了三个预测问题的公式，来阐明三种类型的建模任务︰

1. **二元分类**：预测是否为行程支付了提示（即大于 $0 的*tip\_量*）为正示例，而*\_tip 的大小*$0 为负的示例。
2. **多元分类**：预测为行程支付的小费的范围。 我们将 *tip\_amount* 划分五个分类收纳组或类别：

        Class 0 : tip_amount = $0
        Class 1 : tip_amount > $0 and tip_amount <= $5
        Class 2 : tip_amount > $5 and tip_amount <= $10
        Class 3 : tip_amount > $10 and tip_amount <= $20
        Class 4 : tip_amount > $20
3. **回归任务**：预测为行程支付的 tip 量。

## <a name="set-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>设置 Azure 数据科学环境进行高级分析
要设置 Azure 数据科学环境，请遵循以下步骤。

**创建自己的 Azure Blob 存储帐户**

* 设置自己的 Azure Blob 存储时，请为 Azure Blob 存储选择一个位于**美国中南部**或尽可能靠近美国中南部的地理位置，NYC 出租车数据存储在美国中南部。 将使用 AzCopy 将数据从公共 blob 存储容器复制到自己的存储中的某个容器。 Azure Blob 存储越接近美国中南部，完成此任务（步骤 4）的速度将越快。
* 若要创建自己的 Azure 存储帐户，请按照[关于 Azure 存储帐户](../../storage/common/storage-create-storage-account.md)中所述的步骤进行操作。 请务必记下以下存储帐户凭据的值，因为在此演练中稍后将需要它们。

  * **存储帐户名称**
  * **存储帐户密钥**
  * **容器名称**（希望在 Azure Blob 存储中存储数据的容器的名称）

**预配 Azure Synapse Analytics 实例。**
请参阅在[Azure 门户中创建和查询 AZURE SQL 数据仓库中](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md)的文档，预配 Azure Synapse Analytics 实例。 请确保在以下步骤中将用到的以下 Azure Synapse Analytics 凭据上进行了注释。

* **服务器名称**： \<服务器名称>. database.windows.net
* **SQLDW（数据库）名称**
* **用户名**
* **密码**

**安装 Visual Studio 和 SQL Server Data Tools。** 有关说明，请参阅[Visual Studio 2019 FOR SQL 数据仓库](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-install-visual-studio.md)入门。

**通过 Visual Studio 连接到 Azure Synapse Analytics。** 有关说明，请参阅[连接到 Azure Synapse Analytics 中的 SQL Analytics](../../synapse-analytics/sql/connect-overview.md)中的步骤 1 & 2。

> [!NOTE]
> 在 Azure Synapse Analytics 中创建的数据库上运行以下 SQL 查询（而不是在连接主题的步骤3中提供的查询）来**创建主密钥**。
>
>

    BEGIN TRY
           --Try to create the master key
        CREATE MASTER KEY
    END TRY
    BEGIN CATCH
           --If the master key exists, do nothing
    END CATCH;

**在 Azure 订阅下创建一个 Azure 机器学习工作区。** 有关说明，请参阅[创建 Azure 机器学习工作区](../studio/create-workspace.md)。

## <a name="load-the-data-into-azure-synapse-analytics"></a><a name="getdata"></a>将数据加载到 Azure Synapse Analytics
打开 Windows PowerShell 命令控制台。 运行以下 PowerShell 命令将我们在 GitHub 上与你共享的示例 SQL 脚本文件下载到使用参数 *-DestDir* 指定的本地目录中。 可以将参数 *-DestDir* 的值更改为任何本地目录。 如果 *-DestDir* 不存在，PowerShell 脚本将创建它。

> [!NOTE]
> 如果 *DestDir* 目录需要管理员权限才能创建或向其中写入数据，那么在执行下面的 PowerShell 脚本时，可能需要**以管理员身份运行**。
>
>

    $source = "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/Download_Scripts_SQLDW_Walkthrough.ps1"
    $ps1_dest = "$pwd\Download_Scripts_SQLDW_Walkthrough.ps1"
    $wc = New-Object System.Net.WebClient
    $wc.DownloadFile($source, $ps1_dest)
    .\Download_Scripts_SQLDW_Walkthrough.ps1 –DestDir 'C:\tempSQLDW'

在成功执行之后，当前工作目录会更改为 *-DestDir*。 应该能够看到类似下面的屏幕︰

![当前工作目录更改][19]

在 *-DestDir* 中，在管理员模式下执行下面的 PowerShell 脚本︰

    ./SQLDW_Data_Import.ps1

第一次运行 PowerShell 脚本时，系统会要求你输入 Azure Synapse Analytics 和 Azure blob 存储帐户中的信息。 此 PowerShell 脚本完成首次运行之后，输入的凭据那时已经写入到现有工作目录中的一个名为 SQLDW.conf 的配置文件中。 以后运行此 PowerShell 脚本文件时，可以选择从此配置文件中读取所有需要的参数。 如果需要更改某些参数，可以选择通过删除此配置文件并按提示输入参数值，根据提示在屏幕上输入参数，或者通过编辑 *-DestDir* 目录中的 SQLDW.conf 文件更改参数值。

> [!NOTE]
> 若要避免架构名称与 Azure Azure Synapse Analytics 中已存在的架构名称冲突，请在直接从 SQLDW 文件读取参数时，将 SQLDW 文件中的架构名称添加为每个运行的默认架构名称。 PowerShell 脚本可能会提示输入架构名称︰该名称可以由用户自行指定。
>
>

此 **PowerShell 脚本**文件完成以下任务︰

* **下载并安装 AzCopy**（如果 AzCopy 尚未安装）

        $AzCopy_path = SearchAzCopy
        if ($AzCopy_path -eq $null){
               Write-Host "AzCopy.exe is not found in C:\Program Files*. Now, start installing AzCopy..." -ForegroundColor "Yellow"
            InstallAzCopy
            $AzCopy_path = SearchAzCopy
        }
            $env_path = $env:Path
            for ($i=0; $i -lt $AzCopy_path.count; $i++){
                if ($AzCopy_path.count -eq 1){
                    $AzCopy_path_i = $AzCopy_path
                } else {
                    $AzCopy_path_i = $AzCopy_path[$i]
                }
                if ($env_path -notlike '*' +$AzCopy_path_i+'*'){
                    Write-Host $AzCopy_path_i 'not in system path, add it...'
                    [Environment]::SetEnvironmentVariable("Path", "$AzCopy_path_i;$env_path", "Machine")
                    $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
                    $env_path = $env:Path
                }
* 使用 AzCopy **将数据从公共 blob 复制到专用 blob 存储帐户**

        Write-Host "AzCopy is copying data from public blob to yo storage account. It may take a while..." -ForegroundColor "Yellow"
        $start_time = Get-Date
        AzCopy.exe /Source:$Source /Dest:$DestURL /DestKey:$StorageAccountKey /S
        $end_time = Get-Date
        $time_span = $end_time - $start_time
        $total_seconds = [math]::Round($time_span.TotalSeconds,2)
        Write-Host "AzCopy finished copying data. Please check your storage account to verify." -ForegroundColor "Yellow"
        Write-Host "This step (copying data from public blob to your storage account) takes $total_seconds seconds." -ForegroundColor "Green"
* 使用以下命令将数据从专用 blob 存储帐户中的**Polybase （通过执行 LoadDataToSQLDW）加载到 Azure Synapse Analytics**中。

  * 创建架构

          EXEC (''CREATE SCHEMA {schemaname};'');
  * 创建数据库范围的凭据

          CREATE DATABASE SCOPED CREDENTIAL {KeyAlias}
          WITH IDENTITY = ''asbkey'' ,
          Secret = ''{StorageAccountKey}''
  * 为 Azure 存储 blob 创建外部数据源

          CREATE EXTERNAL DATA SOURCE {nyctaxi_trip_storage}
          WITH
          (
              TYPE = HADOOP,
              LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
              CREDENTIAL = {KeyAlias}
          )
          ;

          CREATE EXTERNAL DATA SOURCE {nyctaxi_fare_storage}
          WITH
          (
              TYPE = HADOOP,
              LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
              CREDENTIAL = {KeyAlias}
          )
          ;
  * 为 csv 文件创建外部文件格式。 数据是未压缩的，字段是使用管道字符进行分隔的。

          CREATE EXTERNAL FILE FORMAT {csv_file_format}
          WITH
          (
              FORMAT_TYPE = DELIMITEDTEXT,
              FORMAT_OPTIONS
              (
                  FIELD_TERMINATOR ='','',
                  USE_TYPE_DEFAULT = TRUE
              )
          )
          ;
  * 在 Azure Blob 存储中为 NYC 出租车数据集创建外部费用和行程表。

          CREATE EXTERNAL TABLE {external_nyctaxi_fare}
          (
              medallion varchar(50) not null,
              hack_license varchar(50) not null,
              vendor_id char(3),
              pickup_datetime datetime not null,
              payment_type char(3),
              fare_amount float,
              surcharge float,
              mta_tax float,
              tip_amount float,
              tolls_amount float,
              total_amount float
          )
          with (
              LOCATION    = ''/nyctaxifare/'',
              DATA_SOURCE = {nyctaxi_fare_storage},
              FILE_FORMAT = {csv_file_format},
              REJECT_TYPE = VALUE,
              REJECT_VALUE = 12
          )

            CREATE EXTERNAL TABLE {external_nyctaxi_trip}
            (
                   medallion varchar(50) not null,
                   hack_license varchar(50)  not null,
                   vendor_id char(3),
                   rate_code char(3),
                   store_and_fwd_flag char(3),
                   pickup_datetime datetime  not null,
                   dropoff_datetime datetime,
                   passenger_count int,
                   trip_time_in_secs bigint,
                   trip_distance float,
                   pickup_longitude varchar(30),
                   pickup_latitude varchar(30),
                   dropoff_longitude varchar(30),
                   dropoff_latitude varchar(30)
            )
            with (
                LOCATION    = ''/nyctaxitrip/'',
                DATA_SOURCE = {nyctaxi_trip_storage},
                FILE_FORMAT = {csv_file_format},
                REJECT_TYPE = VALUE,
                REJECT_VALUE = 12
            )

    - 将数据从 Azure blob 存储中的外部表加载到 Azure Synapse Analytics

            CREATE TABLE {schemaname}.{nyctaxi_fare}
            WITH
            (
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            SELECT *
            FROM   {external_nyctaxi_fare}
            ;

            CREATE TABLE {schemaname}.{nyctaxi_trip}
            WITH
            (
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            SELECT *
            FROM   {external_nyctaxi_trip}
            ;

    - 创建示例数据表 (NYCTaxi_Sample)，并通过在行程和费用表上选择 SQL 查询向其插入数据。 （本演练的一些步骤需要使用此示例表。）

            CREATE TABLE {schemaname}.{nyctaxi_sample}
            WITH
            (
                CLUSTERED COLUMNSTORE INDEX,
                DISTRIBUTION = HASH(medallion)
            )
            AS
            (
                SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount,
                tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
                tip_class = CASE
                        WHEN (tip_amount = 0) THEN 0
                        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                        ELSE 4
                    END
                FROM {schemaname}.{nyctaxi_trip} t, {schemaname}.{nyctaxi_fare} f
                WHERE datepart("mi",t.pickup_datetime) = 1
                AND t.medallion = f.medallion
                AND   t.hack_license = f.hack_license
                AND   t.pickup_datetime = f.pickup_datetime
                AND   pickup_longitude <> ''0''
                AND   dropoff_longitude <> ''0''
            )
            ;

存储帐户的地理位置会影响加载时间。

> [!NOTE]
> 根据专用 blob 存储帐户的地理位置，将数据从公共 blob 复制到专用存储帐户的过程可能需要大约15分钟甚至更长时间，将数据从存储帐户加载到 Azure Azure Synapse Analytics 的过程可能需要20分钟或更长时间。
>
>

将必须决定有重复的源和目标文件时该怎么办。

> [!NOTE]
> 如果专用 blob 存储帐户中已经有要从公共 blob 存储复制到专用 blob 存储帐户的 .csv 文件，那么 AzCopy 将询问你是否要将其覆盖。 如果不希望覆盖它们，请在提示时输入 **n**。 如果希望覆盖它们**全部**，请在提示时输入 **a**。 也可以输入 **y** 单独覆盖.csv 文件。
>
>

![AzCopy 中的输出][21]

可以使用自己的数据。 如果数据位于本地计算机上的实际应用程序中，仍然可以使用 AzCopy 将本地数据上传到专用 Azure blob 存储。 仅需要将 PowerShell 脚本文件的 AzCopy 命令中的**源**位置 `$Source = "http://getgoing.blob.core.windows.net/public/nyctaxidataset"` 更改为包含数据的本地目录。

> [!TIP]
> 如果你的数据已在实际应用程序中的专用 Azure blob 存储中，则可以跳过 PowerShell 脚本中的 AzCopy 步骤，并直接将数据上传到 Azure Azure Synapse Analytics。 这会需要对脚本进行额外的编辑，使其适合数据的格式。
>
>

此 PowerShell 脚本还将 Azure Synapse Analytics 信息插入到数据浏览示例文件 SQLDW_Explorations .sql、SQLDW_Explorations ipynb SQLDW_Explorations_Scripts 和 py，以便在 PowerShell 脚本完成后可以立即立即试用这三个文件。

在成功执行之后，看到的屏幕与下面类似︰

![成功执行脚本的输出][20]

## <a name="data-exploration-and-feature-engineering-in-azure-synapse-analytics"></a><a name="dbexplore"></a>Azure Synapse 分析中的数据浏览和功能设计
在本部分中，我们将通过使用**Visual Studio Data Tools**直接对 Azure Synapse ANALYTICS 运行 SQL 查询来执行数据浏览和功能生成。 本部分中使用的所有 SQL 查询都可以在名为 *SQLDW_Explorations.sql* 的示例脚本中找到。 此文件已经由 PowerShell 脚本下载到本地目录。 也可以从 [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/SQLDW_Explorations.sql) 检索它。 但 GitHub 中的文件并未插入 Azure Synapse Analytics 信息。

使用 Visual Studio 和 Azure Synapse Analytics 登录名和密码连接到 Azure Synapse Analytics，并打开**SQL 对象资源管理器**以确认已导入数据库和表。 检索 *SQLDW_Explorations.sql* 文件。

> [!NOTE]
> 要打开并行数据仓库 (PDW) 查询编辑器，请使用 **New Query** 命令，同时保持 PDW 在 **SQL 对象资源管理器**中处于选中状态。 PDW 不支持标准 SQL 查询编辑器。
>
>

以下是本部分中执行的数据浏览和功能生成任务的类型：

* 在不同的时间范围中探索几个字段的数据分布。
* 调查经度和纬度字段的数据质量。
* 根据 **tip\_amount** 生成二元和多元分类标签。
* 生成特征，并计算/比较行程距离。
* 联接两个表，并提取用于构建模型的随机示例。

### <a name="data-import-verification"></a>数据导入验证
通过这些查询，可以快速验证早先使用 Polybase 的平行批量导入填充的表中的行和列的数量，

    -- Report number of rows in table <nyctaxi_trip> without table scan
    SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')

    -- Report number of columns in table <nyctaxi_trip>
    SELECT COUNT(*) FROM information_schema.columns WHERE table_name = '<nyctaxi_trip>' AND table_schema = '<schemaname>'

**输出︰** 行数应该是 173,179,759，列数应该是 14。

### <a name="exploration-trip-distribution-by-medallion"></a>浏览：依据徽章的行程分布
此示例查询标识在指定的时间段内完成超过 100 个行程的徽章（出租车编号）。 查询将受益于分区表访问，因为它受 **pickup\_datetime** 分区方案的限制。 查询完整数据集还将使用分区表和/或索引扫描。

    SELECT medallion, COUNT(*)
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
    GROUP BY medallion
    HAVING COUNT(*) > 100

**输出：** 查询应返回一个表，其中的行指定13369徽章（出租车）和在2013中完成的行程数。 最后一列包含已完成的行程数量的计算。

### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>浏览：依据徽章和 hack_license 的行程分布
此示例标识在指定的时间段内完成超过 100 个行程的徽章（出租车编号）和 hack_license 编号（驾驶员）。

    SELECT medallion, hack_license, COUNT(*)
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
    GROUP BY medallion, hack_license
    HAVING COUNT(*) > 100

**输出︰** 查询应返回一个包含 13,369 行的表，这些行指定在 2013年已完成超过 100 个行程的 13,369 个汽车/驾驶员 ID。 最后一列包含已完成的行程数量的计算。

### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>数据质量评估：验证含有不正确的经度和/或纬度的记录
此示例将调查是否有任何一个经度和/或纬度字段包含无效的值（弧度应介于 -90 到 90 之间），或者具有（0，0）坐标。

    SELECT COUNT(*) FROM <schemaname>.<nyctaxi_trip>
    WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
    AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
    OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
    OR    (pickup_longitude = '0' AND pickup_latitude = '0')
    OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))

**输出︰** 查询返回经度和/或纬度字段无效的 837,467 个行程。

### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>浏览︰已付小费与未付小费的行程分布
此示例查找指定时间段（或如果像此处设置的那在，时间段为全年，则表示完整的数据集）内已付小费与未付小费的行程的数量。 此分布反映二元标签分布，以便以后用于二元分类建模。

    SELECT tipped, COUNT(*) AS tip_freq FROM (
      SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
      FROM <schemaname>.<nyctaxi_fare>
      WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
    GROUP BY tipped

**输出︰** 查询应返回 2013 年度的以下小费频率︰90,447,622 个已付小费的和 82,264,709 个未付小费的。

### <a name="exploration-tip-classrange-distribution"></a>浏览：小费分类/范围分布
此示例将计算给定的时间段（或如果时间段为全年，则表示完整的数据集）内的小费范围分布。 此标签类的分布稍后将用于多类分类建模。

    SELECT tip_class, COUNT(*) AS tip_freq FROM (
        SELECT CASE
            WHEN (tip_amount = 0) THEN 0
            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
            ELSE 4
        END AS tip_class
    FROM <schemaname>.<nyctaxi_fare>
    WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
    GROUP BY tip_class

**输出**

| tip_class | tip_freq |
| --- | --- |
| 1 |82230915 |
| 2 |6198803 |
| 3 |1932223 |
| 0 |82264625 |
| 4 |85765 |

### <a name="exploration-compute-and-compare-trip-distance"></a>浏览：计算并比较行程距离
此示例将上车和下车经度和纬度转换为 SQL 地理位置点，使用 SQL 地理位置点差异计算行程距离，并返回一个随机抽样的结果进行比较。 该示例仅在使用前面介绍的数据质量评估查询，将结果限制为有效坐标。

    /****** Object:  UserDefinedFunction [dbo].[fnCalculateDistance] ******/
    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO

    IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
      DROP FUNCTION fnCalculateDistance
    GO

    -- User-defined function to calculate the direct distance  in mile between two geographical coordinates.
    CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

    RETURNS float
    AS
    BEGIN
          DECLARE @distance decimal(28, 10)
          -- Convert to radians
          SET @Lat1 = @Lat1 / 57.2958
          SET @Long1 = @Long1 / 57.2958
          SET @Lat2 = @Lat2 / 57.2958
          SET @Long2 = @Long2 / 57.2958
          -- Calculate distance
          SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
          --Convert to miles
          IF @distance <> 0
          BEGIN
            SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
          END
          RETURN @distance
    END
    GO

    SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
    dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
    FROM <schemaname>.<nyctaxi_trip>
    WHERE datepart("mi",pickup_datetime)=1
    AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
    AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
    AND pickup_longitude != '0' AND dropoff_longitude != '0'

### <a name="feature-engineering-using-sql-functions"></a>使用 SQL 函数的特性工程
有时 SQL 函数可以成为特性工程的一个有效选项。 在本演练中，我们定义了一个 SQL 函数，用于计算提取位置和减少位置之间的直接距离。 可以在 **Visual Studio Data Tools** 中运行以下 SQL 脚本。

下面是定义距离函数的 SQL 脚本。

    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO

    IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
      DROP FUNCTION fnCalculateDistance
    GO

    -- User-defined function calculate the direct distance between two geographical coordinates.
    CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

    RETURNS float
    AS
    BEGIN
          DECLARE @distance decimal(28, 10)
          -- Convert to radians
          SET @Lat1 = @Lat1 / 57.2958
          SET @Long1 = @Long1 / 57.2958
          SET @Lat2 = @Lat2 / 57.2958
          SET @Long2 = @Long2 / 57.2958
          -- Calculate distance
          SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
          --Convert to miles
          IF @distance <> 0
          BEGIN
            SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
          END
          RETURN @distance
    END
    GO

下面是一个示例，显示如何调用此函数以在 SQL 查询中生成功能︰

    -- Sample query to call the function to create features
    SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
    dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
    FROM <schemaname>.<nyctaxi_trip>
    WHERE datepart("mi",pickup_datetime)=1
    AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
    AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
    AND pickup_longitude != '0' AND dropoff_longitude != '0'

**输出︰** 此查询生成一个表（包含 2,803,538 行），其中有提取纬度和减少纬度、提取经度和减少经度以及相应的直接距离（以英里计）。 下面是前三行的结果：

|  | pickup_latitude | pickup_longitude | dropoff_latitude | dropoff_longitude | DirectDistance |
| --- | --- | --- | --- | --- | --- |
| 1 |40.731804 |-74.001083 |40.736622 |-73.988953 |.7169601222 |
| 2 |40.715794 |-74,010635 |40.725338 |-74.00399 |.7448343721 |
| 3 |40.761456 |-73.999886 |40.766544 |-73.988228 |0.7037227967 |

### <a name="prepare-data-for-model-building"></a>准备建模的数据
下面的查询联接 **nyctaxi\_trip** 和 **nyctaxi\_fare** 表，生成一个二元分类标签 **tipped**，多类分类标签 **tip\_class**，并从完整联接的数据集中提取样本。 采样是通过检索基于提取时间的行程的子集来完成的。  可以复制此查询，然后将其直接粘贴到[Azure 机器学习 Studio （经典）](https://studio.azureml.net) [导入数据][模块中]，以便从 Azure 中的 SQL 数据库实例进行直接数据引入。 此查询将排除具有不正确（0，0）坐标的记录。

    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
        CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
        CASE WHEN (tip_amount = 0) THEN 0
            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
            ELSE 4
        END AS tip_class
    FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
    WHERE datepart("mi",t.pickup_datetime) = 1
    AND   t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
    AND   pickup_longitude != '0' AND dropoff_longitude != '0'

准备好进行 Azure 机器学习后，也可以：

1. 保存最终的 SQL 查询，以提取和采样数据，并直接将查询复制并粘贴到 Azure 机器学习中的[导入数据模块，]或
2. 保留计划用于在新的 Azure Synapse 分析表中进行建模的抽样和工程数据，并使用 Azure 机器学习中的 "[导入数据][导入数据]" 模块中的新表。 之前步骤中的 PowerShell 脚本已为你执行此任务。 可以直接从“导入数据”模块中的此表读取。

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>IPython Notebook 中的数据浏览和功能设计
在本部分中，我们将使用 Python 和 SQL 查询对先前创建的 Azure Synapse Analytics 执行数据浏览和功能生成。 名为 **SQLDW_Explorations.ipynb** 的 IPython Notebook 示例和名为 **SQLDW_Explorations_Scripts.py** 的 Python 脚本文件已下载到本地目录。 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/SQLDW) 上也有提供。 这两个文件在 Python 脚本中相同。 提供 Python 脚本文件用于应对没有 IPython Notebook 服务器的情况。 这两个示例 Python 文件在 **Python 2.7** 下开发。

PowerShell 脚本已将示例 IPython 笔记本中所需的 Azure Synapse 分析信息以及下载到本地计算机的 Python 脚本文件预先插入到你的本地计算机。 它们是可执行文件，无需任何修改。

如果已设置好 Azure 机器学习工作区，可以直接将示例 IPython Notebook 上传到 AzureML IPython Notebook 服务，并开始运行。 下面是上传到 AzureML IPython Notebook 服务的步骤：

1. 登录 Azure 机器学习工作区、单击顶部的“工作室”，并单击网页左侧的“NOTEBOOKS”。********

    ![依次单击“Studio”和“笔记本”][22]
2. 单击网页左下角的 "**新建**"，然后选择 " **Python 2**"。 然后，为笔记本提供名称，并单击复选标记以创建新的空白 IPython Notebook。

    ![单击“新建”，然后选择“Python 2”][23]
3. 单击新 IPython 笔记本的左上角的**Jupyter**符号。

    ![单击 Jupyter 符号][24]
4. 将示例 IPython Notebook 拖放到 AzureML IPython Notebook 服务的“树”**** 页面，然后单击“上传”****。 然后，示例 IPython Notebook 将上传到 AzureML IPython Notebook 服务。

    ![单击“上传”][25]

若要运行示例 IPython Notebook 或 Python 脚本文件，需要使用以下 Python 包。 如果使用的是 AzureML IPython Notebook 服务，这些包已经预安装。

- pandas
- numpy
- matplotlib
- pyodbc
- PyTables

使用大型数据 Azure 机器学习生成高级分析解决方案时，建议使用以下顺序：

* 将小型数据示例读入到内存中的数据帧。
* 使用抽样数据执行一些可视化效果和浏览。
* 使用抽样数据进行功能设计实验。
* 对于大型数据浏览、数据操作和功能设计，请使用 Python 直接针对 Azure Synapse Analytics 发出 SQL 查询。
* 决定适用于 Azure 机器学习建模的样本大小。

以下是几个数据浏览、数据可视化和功能设计示例。 示例 IPython Notebook 和示例 Python 脚本文件中包含更多数据浏览。

### <a name="initialize-database-credentials"></a>初始化数据库凭据
使用以下变量初始化数据库连接设置：

    SERVER_NAME=<server name>
    DATABASE_NAME=<database name>
    USERID=<user name>
    PASSWORD=<password>
    DB_DRIVER = <database driver>

### <a name="create-database-connection"></a>创建数据库连接
下面是创建数据库连接的连接字符串。

    CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
    conn = pyodbc.connect(CONNECTION_STRING)

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>报告表 <nyctaxi_trip> 中的行数和列数
    nrows = pd.read_sql('''
        SELECT SUM(rows) FROM sys.partitions
        WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
    ''', conn)

    print 'Total number of rows = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''
        SELECT COUNT(*) FROM information_schema.columns
        WHERE table_name = ('<nyctaxi_trip>') AND table_schema = ('<schemaname>')
    ''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]

* 总行数 = 173179759
* 总列数 = 14

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_fare"></a>报告表 <nyctaxi_fare> 中的行数和列数
    nrows = pd.read_sql('''
        SELECT SUM(rows) FROM sys.partitions
        WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_fare>')
    ''', conn)

    print 'Total number of rows = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''
        SELECT COUNT(*) FROM information_schema.columns
        WHERE table_name = ('<nyctaxi_fare>') AND table_schema = ('<schemaname>')
    ''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]

* 总行数 = 173179759
* 总列数 = 11

### <a name="read-in-a-small-data-sample-from-the-azure-synapse-analytics-database"></a>从 Azure Synapse 分析数据库读入小型数据样本
    t0 = time.time()

    query = '''
        SELECT TOP 10000 t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
            f.tolls_amount, f.total_amount, f.tip_amount
        FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
        WHERE datepart("mi",t.pickup_datetime) = 1
        AND   t.medallion = f.medallion
        AND   t.hack_license = f.hack_license
        AND   t.pickup_datetime = f.pickup_datetime
    '''

    df1 = pd.read_sql(query, conn)

    t1 = time.time()
    print 'Time to read the sample table is %f seconds' % (t1-t0)

    print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])

读取示例表的时间为 14.096495 秒。
检索到的行数和列数 = (1000, 21)。

### <a name="descriptive-statistics"></a>描述性统计信息
现在可浏览抽样数据。 首先查看 **trip\_distance**（或选择指定的任何其他字段）的一些描述性统计信息。

    df1['trip_distance'].describe()

### <a name="visualization-box-plot-example"></a>可视化效果：盒须图示例
接下来，我们查看行程距离的盒须图，以可视化分位数。

    df1.boxplot(column='trip_distance',return_type='dict')

![盒须图输出][1]

### <a name="visualization-distribution-plot-example"></a>可视化效果：分布图示例
可视化分布图和抽样行程距离的直方图。

    fig = plt.figure()
    ax1 = fig.add_subplot(1,2,1)
    ax2 = fig.add_subplot(1,2,2)
    df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
    df1['trip_distance'].hist(ax=ax2, bins=100, color='k')

![分布图输出][2]

### <a name="visualization-bar-and-line-plots"></a>可视化效果：条形图和折线图
在此示例中，我们可以将行程距离量化为五个分类收纳组，并将分类收纳结果可视化。

    trip_dist_bins = [0, 1, 2, 4, 10, 1000]
    df1['trip_distance']
    trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
    trip_dist_bin_id

我们可以使用以下方法在条形图或折线图中绘制上述量化分布：

    pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')

![条形图输出][3]

and

    pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')

![折线图输出][4]

### <a name="visualization-scatterplot-examples"></a>可视化效果：散点图示例
我们会显示在 **trip\_time\_in\_secs** 和 **trip\_distance** 之间的散点图，以查看是否存在任何关联

    plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])

![时间和距离之间的关系的散点图输出][6]

同样地，我们可以检查 **rate\_code** 和 **trip\_distance** 之间的关系。

    plt.scatter(df1['passenger_count'], df1['trip_distance'])

![代码和距离之间的关系的散点图输出][8]

### <a name="data-exploration-on-sampled-data-using-sql-queries-in-ipython-notebook"></a>在 IPython Notebook 中使用 SQL 查询对抽样数据进行数据浏览
在本部分中，我们将使用在前面创建的新表中保存的抽样数据来浏览数据分布。 可以使用原始表执行类似的探索。

#### <a name="exploration-report-number-of-rows-and-columns-in-the-sampled-table"></a>浏览：报告抽样表中的行数和列数
    nrows = pd.read_sql('''SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_sample>')''', conn)
    print 'Number of rows in sample = %d' % nrows.iloc[0,0]

    ncols = pd.read_sql('''SELECT count(*) FROM information_schema.columns WHERE table_name = ('<nyctaxi_sample>') AND table_schema = '<schemaname>'''', conn)
    print 'Number of columns in sample = %d' % ncols.iloc[0,0]

#### <a name="exploration-tippednot-tripped-distribution"></a>浏览：已付小费/未付小费分布
    query = '''
        SELECT tipped, count(*) AS tip_freq
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY tipped
        '''

    pd.read_sql(query, conn)

#### <a name="exploration-tip-class-distribution"></a>浏览：小费类型分布
    query = '''
        SELECT tip_class, count(*) AS tip_freq
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY tip_class
    '''

    tip_class_dist = pd.read_sql(query, conn)

#### <a name="exploration-plot-the-tip-distribution-by-class"></a>浏览：按类型绘制小费分布
    tip_class_dist['tip_freq'].plot(kind='bar')

![图 26][26]

#### <a name="exploration-daily-distribution-of-trips"></a>浏览：每日行程分布
    query = '''
        SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY CONVERT(date, dropoff_datetime)
    '''

    pd.read_sql(query,conn)

#### <a name="exploration-trip-distribution-per-medallion"></a>浏览：按徽章的行程分布
    query = '''
        SELECT medallion,count(*) AS c
        FROM <schemaname>.<nyctaxi_sample>
        GROUP BY medallion
    '''

    pd.read_sql(query,conn)

#### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>浏览：依据徽章和出租汽车执照的行程分布
    query = '''select medallion, hack_license,count(*) from <schemaname>.<nyctaxi_sample> group by medallion, hack_license'''
    pd.read_sql(query,conn)


#### <a name="exploration-trip-time-distribution"></a>浏览：行程时间分布
    query = '''select trip_time_in_secs, count(*) from <schemaname>.<nyctaxi_sample> group by trip_time_in_secs order by count(*) desc'''
    pd.read_sql(query,conn)

#### <a name="exploration-trip-distance-distribution"></a>浏览：行程距离分布
    query = '''select floor(trip_distance/5)*5 as tripbin, count(*) from <schemaname>.<nyctaxi_sample> group by floor(trip_distance/5)*5 order by count(*) desc'''
    pd.read_sql(query,conn)

#### <a name="exploration-payment-type-distribution"></a>浏览：付款类型分布
    query = '''select payment_type,count(*) from <schemaname>.<nyctaxi_sample> group by payment_type'''
    pd.read_sql(query,conn)

#### <a name="verify-the-final-form-of-the-featurized-table"></a>验证功能化表的最终形式
    query = '''SELECT TOP 100 * FROM <schemaname>.<nyctaxi_sample>'''
    pd.read_sql(query,conn)

## <a name="build-models-in-azure-machine-learning"></a><a name="mlmodel"></a>在 Azure 机器学习中构建模型
我们现已准备好在 [Azure 机器学习](https://studio.azureml.net) 中进行建模和模型部署。 数据已可用于之前识别的任意预测问题，即：

1. **二元分类**：预测是否为行程支付了提示。
2. **多类分类**：根据以前定义的类，预测小费支付范围。
3. **回归任务**：预测为行程支付的 tip 量。

若要开始建模练习，请登录到 **Azure 机器学习（经典版）** 工作区。 如果尚未创建机器学习工作区，请参阅[创建 Azure 机器学习工作室（经典版）工作区](../studio/create-workspace.md)。

1. 要开始使用 Azure 机器学习，请参阅[什么是 Azure 机器学习工作室（经典版）？](../studio/what-is-ml-studio.md)
2. 登录 [Azure 机器学习工作室（经典版）](https://studio.azureml.net)。
3. 机器学习工作室（经典版）主页上提供丰富的信息、视频、教程、指向模块参考链接及其他资源。 有关 Azure 机器学习的详细信息，请参阅[Azure 机器学习文档中心](https://azure.microsoft.com/documentation/services/machine-learning/)。

典型的训练实验由以下步骤组成：

1. 创建 **+ 新**试验。
2. 将数据导入 Azure 机器学习工作室（经典版）。
3. 根据需要预先处理、转换和处理数据。
4. 根据需要生成功能。
5. 将数据拆分为训练/验证/测试数据集（或每个类具有单独的数据集）。
6. 根据要解决的学习问题，选择一个或多个机器学习算法。 例如，二元分类、多类分类和回归。
7. 使用定型数据集训练一个或多个模型。
8. 使用定型模型对验证数据集评分。
9. 评估模型来计算针对学习问题的相关指标。
10. 优化模型并选择要部署的最佳模型。

在此练习中，我们已在 Azure Synapse Analytics 中探索并设计了数据，并决定了要在 Azure 机器学习 Studio （经典）中引入的示例大小。 下面是构建一个或多个预测模型的过程：

1. 使用 "**数据输入和输出**" 部分提供的 "[导入数据][导入数据]" 模块，将数据导入 Azure 机器学习 Studio （经典）。 有关详细信息，请参阅[导入数据][import-data]模块参考页。

    ![Azure ML 导入数据][17]
2. 在“**属性**”面板中，选择“**Azure SQL 数据库**”作为**数据源**。
3. 在“**数据库服务器名称**”字段中输入数据库 DNS 名称。 格式：`tcp:<your_virtual_machine_DNS_name>,1433`
4. 在相应字段中输入**数据库名称**。
5. 在“服务器用户帐户名”**** 中输入 *SQL 用户名*，在“服务器用户帐户密码”**** 中输入*密码*。
7. 在 "**数据库查询**" 编辑文本区域中，粘贴用于提取必需的数据库字段（包括所有计算字段，如标签）的查询，然后向下采样数据，使数据达到所需的样本大小。

下图列出了直接从 Azure Synapse Analytics 数据库读取数据的二进制分类试验的示例（请记住，将表名称替换 nyctaxi_trip，并使用在你的演练中使用的表名称 nyctaxi_fare。 可以针对多类分类和回归问题构建类似实验。

![Azure ML 训练][10]

> [!IMPORTANT]
> 在上一部分中提供的建模数据提取和采样查询示例中，**这三个建模练习的所有标签都包括在此查询中**。 每个建模练习的一个重要（必需）步骤是**排除**其他两个问题不需要的标签，以及任何其他的**目标泄漏**。 例如，使用二元分类时，使用标签 **tipped** 并排除字段 **tip\_class**、**tip\_amount** 和 **total\_amount**。 后者是目标泄漏，因为它们指示支付的小费。
>
> 为了排除任何不需要的列或目标泄漏，可以使用[选择数据集中的列][select-columns]模块或[编辑元数据][edit-metadata]。 有关详细信息，请参阅[选择数据集中的列][select-columns]和[编辑元数据][edit-metadata]参考页。
>
>

## <a name="deploy-models-in-azure-machine-learning"></a><a name="mldeploy"></a>在 Azure 机器学习中部署模型
模型已就绪时，即可轻松地从实验直接将其部署为 Web 服务。 有关部署 Azure ML Web 服务的详细信息，请参阅[部署 Azure 机器学习 Web 服务](../studio/deploy-a-machine-learning-web-service.md)。

要部署新 Web 服务，需要：

1. 创建评分实验。
2. 部署 Web 服务。

若要从**已完成**的训练实验中创建评分实验，请单击下方操作栏中的“**创建评分实验**”。

![Azure 评分][18]

Azure 机器学习将尝试根据训练实验的组件创建评分实验。 特别是，它将：

1. 保存训练的模型，并删除模型训练模块。
2. 标识逻辑**输入端口**，以表示预期输入数据架构。
3. 标识逻辑**输出端口**，以表示预期 Web 服务输出架构。

创建评分实验后，请检查结果，并根据需要进行调整。 典型的调整是将输入数据集或查询替换为排除标签字段的数据集，因为在调用服务时，这些标签字段将不会映射到架构。 更好的做法是，将输入数据集和/或查询的大小减小到一些记录，足以指示输入架构。 对于输出端口，通常会使用[选择数据集中的列][select-columns]模块在输出中排除所有输入字段，仅包括“评分标签”**** 和“评分概率”****。

下图提供评分实验示例。 准备部署时，请单击下方操作栏中的“**发布 WEB 服务**”按钮。

![Azure ML 发布][11]

## <a name="summary"></a>“摘要”
简单概括一下我们在此演练教程中完成的任务，已创建 Azure 数据科学环境，并使用大型公共数据集，从而将它的用法贯穿整个数据团队科学过程，从数据采集到 Azure 机器学习 Web 服务的模型训练，再到部署等多项工作。

### <a name="license-information"></a>许可证信息
此示例演练和及其附带脚本和 IPython notebook 是在 MIT 许可证下由 Microsoft 共享。 如需更多详细信息，请查看 GitHub 上示例代码目录中的 LICENSE.txt 文件。

## <a name="references"></a>参考
- [Andrés Monroy NYC 出租车行程下载页](https://www.andresmh.com/nyctaxitrips/)
- [丽丽 Whong) FOILing NYC 的出租车行程数据](https://chriswhong.com/open-data/foil_nyc_taxi/)
- [NYC 出租车和礼车委员会研究和统计信息](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sqldw-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sqldw-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sqldw-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sqldw-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sqldw-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sqldw-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sqldw-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sqldw-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sqldw-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sqldw-walkthrough/azuremltrain.png
[11]: ./media/sqldw-walkthrough/azuremlpublish.png
[12]: ./media/sqldw-walkthrough/ssmsconnect.png
[13]: ./media/sqldw-walkthrough/executescript.png
[14]: ./media/sqldw-walkthrough/sqlserverproperties.png
[15]: ./media/sqldw-walkthrough/sqldefaultdirs.png
[16]: ./media/sqldw-walkthrough/bulkimport.png
[17]: ./media/sqldw-walkthrough/amlreader.png
[18]: ./media/sqldw-walkthrough/amlscoring.png
[19]: ./media/sqldw-walkthrough/ps_download_scripts.png
[20]: ./media/sqldw-walkthrough/ps_load_data.png
[21]: ./media/sqldw-walkthrough/azcopy-overwrite.png
[22]: ./media/sqldw-walkthrough/ipnb-service-aml-1.png
[23]: ./media/sqldw-walkthrough/ipnb-service-aml-2.png
[24]: ./media/sqldw-walkthrough/ipnb-service-aml-3.png
[25]: ./media/sqldw-walkthrough/ipnb-service-aml-4.png
[26]: ./media/sqldw-walkthrough/tip_class_hist_1.png


<!-- Module References -->
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[导入数据]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
