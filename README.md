Python ecommerce-order-data-analytics

基于 Hadoop 与 Hive 的电商订单数据分析项目，通过构建 MySQL → Sqoop → HDFS → Hive 的离线数据处理链路，对客户、商品、订单、库存及需求等业务数据进行统一存储和多维分析，为区域仓储库存配置提供数据支持。

 项目简介

本项目面向电商订单与仓储资源管理场景，构建包含客户、商品、订单、库存、供应商及购物车需求等多类业务数据的数据处理环境。

项目将业务数据存储于 MySQL，并通过 Sqoop 将数据批量导入 Hadoop HDFS；在 Hive 中完成数据表创建、分区、分桶及多表关联，并使用 Hive SQL 对不同区域的商品库存与需求情况进行统计分析。

项目共覆盖 26 个区域（State/Domain），通过区域维度对相关数据进行划分，以支持不同仓储区域的数据查询与库存分析。

 技术栈

 数据存储： MySQL 8、Hadoop HDFS
 数据迁移： Apache Sqoop 1.4.7
 数据处理： Apache Hive 3.1.2、Hive SQL
 数据生成： Python
 任务执行： Shell
 运行环境： Linux、Java 8+

 数据规模

项目包含以下核心业务数据：

| 数据类型            | 数据量 |
| ------------------ | ----: |
| Customers          | 2,000 |
| Items              |   619 |
| Orders             | 3,000 |
| Stocks             | 2,000 |
| Cart / Requirement | 5,000 |
| Vendors            |   300 |
| Register Dates     | 2,000 |
| 区域               |    26 |

数据覆盖客户、商品、订单、库存、供应商及商品需求等多个业务维度。

 数据处理流程

```text
Python
   │
   ▼
生成业务数据
   │
   ▼
MySQL
   │
   │ Sqoop
   ▼
HDFS
   │
   ▼
Hive
   │
   ├── 普通表
   ├── 分桶表
   ├── 关联表
   └── 分区表
   │
   ▼
Hive SQL
   │
   ▼
区域库存 / 商品需求分析
   │
   ▼
CSV 分析结果
```

 Hive 数据建模

项目首先将客户、供应商、商品、购物车、库存及订单等业务数据加载至 Hive。

在此基础上设计不同的数据组织方式：

 创建 6 张基础 Hive 表
 创建 5 张分桶表
 创建 2 张关联结果表
 创建 2 张分区表
 分桶表采用 5 Buckets
 分区数据按照区域维度进行组织

通过不同表结构对数据进行组织，为后续多表关联和区域维度分析提供基础。

 数据分析

项目主要围绕不同区域的商品库存与需求情况展开分析。

核心分析过程包括：

1. 根据指定区域筛选相关业务数据；
2. 关联商品、库存及需求等数据；
3. 对商品库存和需求进行聚合统计；
4. 对比不同商品的库存量与需求量；
5. 根据分析结果判断区域内商品库存是否需要调整。

同时，在 Hive Join 场景中使用 MAPJOIN 对适合的关联任务进行优化，降低大表关联过程中的数据交换压力。

 自动化处理

项目提供 Shell 脚本用于组织 Hive 查询任务。

通过脚本完成：

 Hive 查询执行
 分析结果导出
 CSV 文件生成
 临时表清理

最终形成从数据导入、Hive 建表、SQL 分析到结果输出的完整离线处理流程。

 项目结构

```text
ecommerce-order-data-analytics/
│
├── CSV/
│   └── 区域数据文件
│
├── Creating Data/
│   ├── create_customer.py
│   ├── create_items.py
│   ├── create_orders.py
│   ├── create_stocks.py
│   ├── create_cart.py
│   ├── create_vendors.py
│   └── ...
│
├── Hive/
│   ├── createNormalTables.sql
│   ├── createBucketedTables.sql
│   ├── createJoinTables.sql
│   ├── createPatitionedTables.sql
│   ├── hiveQuery1.sql
│   ├── hiveQuery2.sql
│   └── DoQuery.sh
│
├── SQLFiles/
│   ├── Create_Tables.sql
│   ├── Insert_Customers.sql
│   ├── Insert_Items.sql
│   ├── Insert_Orders.sql
│   └── ...
│
└── README.md
```

 运行环境

项目运行前需要准备以下环境：

 Java 8+
 Hadoop 3.2.1
 MySQL 8
 Apache Sqoop 1.4.7
 Apache Hive 3.1.2

 1. 启动 Hadoop

进入 Hadoop 的 `sbin` 目录并启动相关服务：

```bash
./start-all.sh
```

 2. 准备业务数据

运行 `Creating Data` 目录中的 Python 脚本生成业务数据，并使用 `SQLFiles` 中的 SQL 文件将数据写入 MySQL。

 3. 导入 HDFS

通过 Sqoop 将 MySQL 中的业务表导入 HDFS，例如：

```bash
sqoop import \
--connect jdbc:mysql://localhost/ABShopping \
--table Customers \
--username <your_username> \
-P \
--m 1
```

其他业务表按照相同方式进行导入。

 4. 创建 Hive 表

按照以下顺序执行 Hive SQL：

```text
createNormalTables.sql
        ↓
createBucketedTables.sql
        ↓
createJoinTables.sql
        ↓
createPatitionedTables.sql
```

 5. 执行数据分析

在 `hiveQuery1.sql` 中指定需要分析的区域，然后执行：

```bash
./DoQuery.sh OutputCsv
```

分析结果将输出为 CSV 文件。

 项目结果

通过 Hadoop HDFS 和 Hive 建立电商业务数据的离线处理环境，实现 MySQL 业务数据向分布式存储平台的迁移，并完成区域、商品、库存及需求等维度的数据关联和聚合分析。

项目最终支持 26 个区域的库存与需求数据分析，为区域仓储库存调整提供数据参考。

 Reference

本项目的 Hadoop/Hive 电商数据处理思路参考开源项目：

`abhinaba-fbr/E-Commerce-data-analysis-using-Hadoop`

在此基础上进行学习、整理及项目结构调整。
