---
title: 设计 ELT 而不是 ETL
description: 在 Azure Synapse Analytics 中实现 Synapse SQL 池的灵活数据加载策略
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 02/19/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: e99fd898956e11a4827d023691111a47e5a790c0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "80744955"
---
# <a name="data-loading-strategies-for-synapse-sql-pool"></a>Synapse SQL 池的数据加载策略

传统 SMP SQL 池使用用于加载数据的提取、转换和加载（ETL）过程。 Azure Synapse 分析中的 Synapse SQL 池具有大规模的并行处理（MPP）体系结构，该体系结构利用了计算和存储资源的可伸缩性和灵活性。

使用提取、加载和转换（ELT）过程可以利用 MPP，并消除在加载前数据转换所需的资源。

尽管 SQL 池支持多种加载方法（包括[bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)和[SqlBulkCopy API](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)等常用 SQL Server 选项），但通过 PolyBase 外部表和[COPY 语句](/sql/t-sql/statements/copy-into-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)（预览版），可以最快、最灵活的方式来加载数据。

使用 PolyBase 和 COPY 语句，可通过 T-sql 语言访问存储在 Azure Blob 存储或 Azure Data Lake Store 中的外部数据。 若要在加载时获得最大的灵活性，我们建议使用 COPY 语句。

> [!NOTE]  
> COPY 语句目前为公共预览版功能。 若要提供反馈，请将电子邮件发送到以下sqldwcopypreview@service.microsoft.com通讯组列表：。

> [!VIDEO https://www.youtube.com/embed/l9-wP7OdhDk]

## <a name="what-is-elt"></a>什么是 ELT？

提取、加载和转换（ELT）是一个过程，通过该过程从源系统中提取数据，并将其加载到 SQL 池中，然后转换。

实现 ELT 的基本步骤如下：

1. 将源数据提取到文本文件中。
2. 将数据移入 Azure Blob 存储或 Azure Data Lake Store。
3. 准备要加载的数据。
4. 用 PolyBase 或 COPY 命令将数据加载到临时表中。
5. 转换数据。
6. 将数据插入生产表。

有关 PolyBase 加载教程，请参阅[使用 polybase 从 Azure blob 存储加载数据](load-data-from-azure-blob-storage-using-polybase.md)。

有关详细信息，请参阅[加载模式博客](https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-loading-patterns-and-strategies/)。

## <a name="1-extract-the-source-data-into-text-files"></a>1. 将源数据提取到文本文件中

从源系统中取出数据的过程取决于存储位置。  目标是将数据移入 PolyBase 和 COPY 支持的带分隔符文本文件或 CSV 文件。

### <a name="polybase-and-copy-external-file-formats"></a>PolyBase 和 COPY 外部文件格式

使用 PolyBase 和 COPY 语句，可以从 UTF-8 和 UTF-16 编码的带分隔符文本文件或 CSV 文件加载数据。 除了带分隔符文本文件或 CSV 文件以外，它还可以从 ORC 和 Parquet 等 Hadoop 文件格式加载数据。 PolyBase 和 COPY 语句还可以从 Gzip 和 Snappy 压缩文件加载数据。

不支持扩展的 ASCII、固定宽度格式和不支持的嵌套格式（如 WinZip 或 XML）。 如果要从 SQL Server 导出，可以使用[bcp 命令行工具](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)将数据导出到分隔的文本文件中。

## <a name="2-land-the-data-into-azure-blob-storage-or-azure-data-lake-store"></a>2. 将数据置于 Azure Blob 存储或 Azure Data Lake Store

若要在 Azure 存储中存储数据，可以将数据移动到[Azure Blob 存储](../../storage/blobs/storage-blobs-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)或[Azure Data Lake Store Gen2](../../data-lake-store/data-lake-store-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 应将数据存储在任一位置的文本文件中。 PolyBase 和 COPY 语句可从任一位置加载数据。

可使用来将数据移到 Azure 存储的工具和服务：

- [Azure ExpressRoute](../../expressroute/expressroute-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 服务可以增强网络吞吐量、性能和可预测性。 ExpressRoute 是通过专用连接将数据路由到 Azure 的服务。 ExpressRoute 连接不通过公共 Internet 路由数据。 与基于公共 Internet 的典型连接相比，这些连接提供更高的可靠性、更快的速度、更低的延迟和更高的安全性。
- [AZCopy 实用工具](../../storage/common/storage-choose-data-transfer-solution.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)可以通过公共 Internet 将数据移到 Azure 存储。 如果数据小于 10 TB，则很适合使用此工具。 若要使用 AZCopy 定期执行加载操作，请测试网络速度是否在可接受的范围内。
- [Azure 数据工厂 (ADF)](../../data-factory/introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 提供一个可以安装在本地服务器上的网关。 然后，你可以创建管道，以便将数据从本地服务器移到 Azure 存储。 若要在 SQL Analytics 中使用数据工厂，请参阅[加载用于 Sql analytics 的数据](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

## <a name="3-prepare-the-data-for-loading"></a>3. 准备要加载的数据

在加载之前，你可能需要准备并清除存储帐户中的数据。 可以在数据仍保留在源中、将数据导出到文本文件时或者在数据进入 Azure 存储之后执行数据准备。  最好是在加载过程的早期阶段处理数据。  

### <a name="define-external-tables"></a>定义外部表

如果使用的是 PolyBase，则需要在加载之前在 SQL 池中定义外部表。 COPY 语句不需要外部表。 PolyBase 使用外部表来定义和访问 Azure 存储中的数据。

外部表和数据库视图相似。 外部表包含表架构，并指向存储在 SQL 池外部的数据。

定义外部表涉及到指定数据源、文本文件的格式和表定义。 T-sql 语法参考文章，你将需要：

- [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

加载 Parquet 时，SQL 数据类型映射为：

| **Parquet 数据类型** | **SQL 数据类型** |
| :-------------------: | :---------------: |
|        tinyint        |      tinyint      |
|       smallint        |     smallint      |
|          int          |        int        |
|        bigint         |      bigint       |
|        boolean        |        bit        |
|        Double         |       float       |
|         FLOAT         |       real        |
|        double         |       money       |
|        double         |    smallmoney     |
|        字符串         |       nchar       |
|        字符串         |     nvarchar      |
|        字符串         |       char        |
|        字符串         |      varchar      |
|        binary         |      binary       |
|        binary         |     varbinary     |
|       timestamp       |       date        |
|       timestamp       |   smalldatetime   |
|       timestamp       |     datetime2     |
|       timestamp       |     datetime      |
|       timestamp       |       time        |
|         date          |       date        |
|        decimal        |      Decimal      |

有关创建外部对象的示例，请参阅加载教程中的[创建外部表](load-data-from-azure-blob-storage-using-polybase.md#create-external-tables-for-the-sample-data)步骤。

### <a name="format-text-files"></a>设置文本文件的格式

使用 PolyBase 时，定义的外部对象需要使文本文件中的行与外部表和文件格式定义相符。 文本文件的每一行中的数据必须与表定义相符。
设置文本文件的格式：

- 如果数据来自非关系源，则需要将其转换为行与列。 不管数据来自关系源还是非关系源，都必须转换数据，使之与数据预期要载入到的表的列定义相符。
- 格式化文本文件中的数据，使其与目标表中的列和数据类型保持一致。 外部文本文件中的数据类型和 SQL pool 表中的数据类型不一致会导致在负载期间拒绝行。
- 使用终止符分隔文本文件中的字段。  请确保使用源数据中找不到的字符或字符序列。 使用通过 [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) 指定的终止符。

## <a name="4-load-the-data-using-polybase-or-the-copy-statement"></a>4. 使用 PolyBase 或 COPY 语句加载数据

这是将数据加载到临时表中的最佳做法。 利用临时表，可以处理错误，且不会干扰生产表。 使用临时表，还可以在将数据插入生产表之前，使用 SQL 池 MPP 进行数据转换。

使用 COPY 载入临时表时，需要预先创建表。

### <a name="options-for-loading-with-polybase-and-copy-statement"></a>使用 PolyBase 和 COPY 语句加载数据的选项

若要使用 PolyBase 加载数据，可以使用下列任一加载选项：

- 如果数据位于 Azure Blob 存储或 Azure Data Lake Store 中，则 [PolyBase 与 T-SQL](load-data-from-azure-blob-storage-using-polybase.md) 可以发挥作用。 使用此方法可以获得加载过程的最大控制度，不过同时需要定义外部数据对象。 其他方法在你将源表映射到目标表时，在幕后定义这些对象。  若要协调 T-SQL 负载，可以使用 Azure 数据工厂、SSIS 或 Azure Functions。
- 如果源数据位于本地 SQL Server 或云中的 SQL Server，则 [PolyBase 与 SSIS](/sql/integration-services/load-data-to-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) 可以发挥作用。 SSIS 定义源到目标表的映射，同时可协调负载。 如果已有 SSIS 包，可将这些包修改为使用新的数据仓库目标。
- [Azure 数据工厂 (ADF) 的 PolyBase 和 COPY 语句](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)是另一个业务流程工具。  它定义管道并计划作业。
- [Polybase 与 Azure Databricks](../../azure-databricks/databricks-extract-load-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)将数据从表传输到 Databricks 数据帧和/或使用 PolyBase 将数据从 Databricks 数据帧写入表。

### <a name="other-loading-options"></a>其他加载选项

除 PolyBase 和 COPY 语句外，还可以使用[bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)或[SqlBulkCopy API](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 bcp 直接加载到数据库，而无需通过 Azure Blob 存储，而仅适用于小型负载。

> [!NOTE]
> 这些选项的负载性能比 PolyBase 和 COPY 语句要慢。

## <a name="5-transform-the-data"></a>5. 转换数据

在数据已进入临时表时，请执行工作负荷所需的转换。 然后将数据移到生产表。

## <a name="6-insert-the-data-into-production-tables"></a>6. 将数据插入生产表

INSERT INTO .。。SELECT 语句将数据从临时表移到永久表中。

设计 ETL 过程时，请尝试针对一个较小的测试示例运行该过程。 尝试将表中的 1000 行提取到某个文件，将该文件移到 Azure，然后将其载入临时表。

## <a name="partner-loading-solutions"></a>合作伙伴加载解决方案

我们的很多合作伙伴都提供加载解决方案。 若要获取更多解决方案，请参阅我们的[解决方案合作伙伴](sql-data-warehouse-partner-business-intelligence.md)的列表。

## <a name="next-steps"></a>后续步骤

有关加载指南，请参阅[加载数据的指南](guidance-for-loading-data.md)。
