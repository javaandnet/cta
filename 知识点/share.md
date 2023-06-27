# 关于
1. 数据的安全访问模型
   1. オブジェクト、レコード、項目の各レベル
2. 对象利用者
   1. 面向在 Salesforce 实施中处理复杂记录访问要求和大型销售组织搬迁的专家架构师。
   2. 
# Salesforce 上的数据访问
1. 用户观点：不是必须但最好理解。
   1. Object[Filed(Account)] =>Data(Account)
   2. 这种情况下，取引先项目和取引先对象都能访问。
   3. 但是在共有Rule或是其它工具添加的访问控制的场合，一部分数据可能无法访问。
2. 架构观点
   1. 有两种角度
      1. オブジェクトレベルのアクセス権 （包含項目レベルのアクセス権）
         1. 对象级权限决定用户是否可以访问特定对象、可以查看哪些字段以及可以执行哪些操作。对象级访问权限在用户配置文件中设置。
         2. 种类
            1. アクセスの制限
               1. 「参照」、「作成」、「編集」、および「削除」のオブジェクト権限は决定拥有访问权限的用户对Object执行什么操作。
               2. 项目安全级别（FLS）决定对特定用户有些情报无法表示。
            2. アクセス権の開放
               1. 「すべて表示」および「すべて変更」オブジェクト権限を使用すると、ユーザが、レコードレベルのアクセス権に関係なく、オブジェクトの全レコードにアクセスできます。
      2. レコードレベルのアクセス権（按照下列优先级）
         1. 种类
            1. 組織の共有設定
            2. ロール階層
            3. テリトリー階層
            4. 共有ルール
            5. チーム
            6. 共有の直接設定
            7. プログラムによる共有
         2. 控制记录访问有很多种选项，固然可以实时更新，但是再计算可能花费时间
# レコードアクセス権の計算
1. 再打开用户界面，使用API，打开记录，报表，访问ListView的时候。首先要判断记录能否访问。
2. 特别是组织别较大的场合，打开页面要花费300MS以上。
3. 当使用共有ルール，すべての階層のトラバース（遍历），レコードアクセス権の継承の分析时，不是在使用时实时进行。
4. 在各种设定改变时计算レコードアクセスデータ，便于迅速被他人访问。
5. 以最小化确定运行时可以访问哪些记录所需的数据库表连接数量的方式进行维护。
# アクセス権の付与
1. 明确赋予
   1. ユーザまたはキューがレコードの所有者になる。
   2. 共有ルールが、非公開グループ、公開グループ、キュー、ロール、またはテリトリーとレコードを共有している。
   3. 割り当てルールが、ユーザまたはキューとレコードを共有している。
   4. テリトリー割り当てルールが、テリトリーとレコードを共有している。
   5. ユーザが、ユーザ、非公開グループ、公開グループ、キュー、ロール、またはテリトリーとレコードを手動で共有している。
   6. ユーザが、取引先、商談、またはケースのチームの一員になる。
   7. 使用程序、ユーザ、非公開グループ、公開グループ、キュー、ロール、またはテリトリーとレコードを共有している
2. 组关系の付与
   1. 某一个用户是已经分享租的一员
3. 继承赋予
4. 暗黙的付与
   1. 当 Salesforce 销售、服务和门户应用程序中嵌入的不可配置记录共享行为授予对特定父记录和子记录的访问权限时，会进行授予。
   2. 如果访问取引先の子纪录，也可以访问父纪录。
   3. 为了提高性能，建议使用リリース更新，将取引先の共有再適用の高速化有効。(在新的更新中建议关闭)
# 数据库架构分析
1. 保存数据的几个数据表
   1. Object Record テーブル
      1. 存储特定对象的记录并指示哪个用户、组或队列拥有每条记录的表。
   2. Object Sharing テーブル
      1. 保存明示的或者暗黙的访问権限的表。
      2. 組織几乎所有オブジェクト (表示するには、[設定] から [クイック検索] に「共有設定」と入力して、[共有設定] ，除了下列几种场合，都在Object Sharing表中可以查到。
         1. 主从关系的场合：主控制从的访问。
         2. 组织设定为Public
         3. Object Sharing テーブル不支持的种别存在的情况：(Activities や Files など)。这些有独自的控制方式。
   3. Group Maintenance テーブル
      1. 存储支持组成员身份和继承的访问权限的数据的表。
      2. 比如某一个用户可以访问，则访问这个表来确认相关的有继承关系的用户。
      3. 当创建或修改组（或角色或区域）成员资格信息时，会预先生成这些记录。
      4. 对象共享表存储对个人和组的访问授权，而组维护表存储属于每个组的用户或组的列表，指示组成员身份。
      5. 两种类型的表都用于确定用户在搜索、查询或提取报表或列表视图时对数据的访问权限。
2. 查询流程
   1. 当用户尝试检索一条或多条记录时，Salesforce 会生成一条 SQL 语句，在对象记录表中搜索与用户搜索字符串匹配的记录。
   2. 如果记录存在，Salesforce 会将 SQL 附加到Object Record テーブルと Object Sharing テーブル、Object Sharing テーブルと Group Maintenance テーブル的上面。看看是否存在访问权限。
   3. 一致标准及查询数据的顺序
      1. Object Record と Object Sharing：レコード ID
      2. Object Sharing と Group Maintenance：ユーザ ID またはグループ ID
3. 赋予权限说明
   1. 对象共享表只是将每个访问权限存储在称为共享行的单独行中，每个共享行授予用户或组对特定记录的访问权限。
   2. 组维护表更加复杂，因为每个用户和组可以通过单个组成员身份或继承的访问权限通过多种方式访问​​记录。

# 共有行
## 存储情报
   1. 该行授予访问权限的记录的 ID
   2. 该行授予访问权限的用户或组的 ID
   3. 该行允许的访问级别（只读、完全访问等）
   4. 共享原因：指示 Salesforce 授予用户或组记录访问权限的原因
## 例
   1. 担任销售主管角色的 Maria 为名为“Acme”的公司创建了一条帐户记录 (ID=A1)。在后台（实际过程中），Salesforce 在帐户共享表（帐户对象的对象共享表）中为 Maria 作为记录所有者创建共享行。
      1. AcctID:数据ID
      2. User/Group:Maria
      3. 访问级别：Full
      4. 原因：Owner
   2. Maria 直接设定Acme 帐户记录以与服务主管 Frank 共享。在内部，Salesforce 为 Frank 创建共享行。
      1. AcctID:数据ID
      2. User/Group:Frank
      3. 访问级别：Read/Write
      4. 原因：Manual
      5. **现在为止数据分析**  
            Acme 只有一条帐户记录，但帐户共享表包含 Acme 记录的两个条目。发生此更新是因为 Salesforce 两次授予对 Acme 帐户记录的访问权限，一次授予作为所有者的 Maria，一次授予 Frank。
    3. 系统管理员创建共享规则以与[Strategy]组共享销售主管记录，并授予[Strategy]组[Read]访问权限。在内部，Salesforce 创建一个共享行，使策略组能够访问 Maria 的 Acme 客户记录。
       1. AcctID:数据ID
       2. User/Group:Strategy
       3. 访问级别：Read
       4. 原因：Rule
 ## 注意点
   1. 对于对记录具有多个访问权限的用户，Salesforce 在确定记录访问权限时使用最高的权限。例如，如果 Frank 加入[Strategy]组，Frank 还将拥有示例 2 中 Maria 授予的读/写访问权限。
   2. 如果多个用户具有相同的记录访问要求，则将它们分组并向组而不是个人授予访问权限会更有效。此方法可以节省时间并减少共享行，从而减少组织中的记录访问数据量。
# Group Maintenance テーブル
## 基本概念
1. 访问权限被授予共享行中的用户和组，但描述哪些用户属于每个组的数据位于Group Maintenance テーブル中。
2. 这些表存储所有 Salesforce 组（包括系统定义的组）的成员资格数据。
3. 系统定义的组是由 Salesforce 在内部创建和管理的用户组，用于支持各种功能和行为，例如队列。
4. 通过这种类型的管理，支持队列的数据和支持私有或公共组的数据可以共存于同一个数据库表中，从而统一了 Salesforce 管理数据的方式。例如，Salesforce 可以向Queue授予记录访问权限，就像向公共组授予记录访问权限一样。
5. Salesforce 还使用系统定义的组来实施层次结构。
6. Salesforce 在重新计算期间为角色层次结构中的每个节点创建两种类型的系统定义组：Role 组和 RoleAndSubscribeds（角色和下属） 组。
7. 組織で外部組織の共有設定が有効になっている場合は、第三种种类の種類のシステム定義グループ RoleAndInternalSubordinates が作成されます。
## 类型
1. Role
   1. 構成
      1. 特定のロール
      2. いずれかのマネージャロール
   2. 目的：用于让经理访问其下属的记录
2. RoleAndSubordinates
   1. 構成
      1. 特定のロール
      2. いずれかのマネージャロール
      3. いずれかの下位ロール
   2. 目的：当您的组织定义与以下角色共享一组记录的规则时使用
      1. 特定のロール
      2. 下位ロール
3. RoleAndInternalSubordinates
   1. 構成
      1. 特定のロール
      2. いずれかのマネージャロール
      3. いずれかの下位ロール (ポータルロールを除く)
   2. 目的：当您的组织定义与以下角色共享一组记录的规则时使用
      1. 特定のロール
      2. 下位ロール (ポータルロールを除く)
4. 所有三种类型的组均包含以下成员：
   1. 间接成员从组的直接成员继承记录访问权限并被分配给经理角色
   2. 根据群组类型定义的直接成员
      1. 对于角色组，直接成员是分配给该组代表的角色的成员。
      2. 对于 RoleAndSubscribeds 组，直接成员是分配给该组代表的角色或其从属角色之一的成员。
      3. 对于 RoleAndInternalSubscribeds 组，直接成员是分配给该组代表的角色或除门户之外的任何从属角色的成员。
## 例子
### 从属关系
 1. CEO ->Marc
    1. SalesExecutive ->Maria
       1. East Sales Rep ->Bob
       2. West Sales Rep ->Wendy
### 保存数据形式 一种向上看，一种向下看
    1. CEO Role
       1. Marc
    2. SalesExecutive
       1. Marc *
       2. Maria
    3. West Sales Rep
       1. Marc *
       2. Maria *
       3. Wendy
    4. East Sales Rep
       1. Marc *
       2. Maria *
       3. Bob
2. RoleAndSubscribeds
    1. CEO:Marc,Maria,Bob,Wendy
    2. SalesExecutive:Marc*,Maria,Bob,Wendy
    3. West Sales Rep:Marc*,Maria,Wendy
    4. East Sales Rep:Marc*,Maria,Bob  
【*】 : 间接参照
3. 保存这样的形式便于直接扫描
### 其他说明
  1. 区域（テリトリ）管理也被系统支持。
  2. 对于每个区域，Salesforce 创建以下组。
     1. 区域组，其中分配到区域的用户是直接成员，分配到层次结构中较高区域的用户是间接成员
     2. TerritoryAndSubscribeds 组，其中分配到该区域或层次结构下方区域的用户是直接成员，分配到该分支上方区域的用户是间接成员
   3. 如果没有系统定义的组，每次记录访问尝试都必须在层次结构的每个分支上发送 Salesforce 遍历，在用户数据表和层次结构数据表之间来回移动。此外，Salesforce 必须使用完全不同的流程来处理几乎所有用户集合。影响时间
   4. 用户无法使用用户界面或 API（如私有或公共组）修改系统定义的组。但是，如果您更改队列或层次结构，Salesforce 会重新应用与其关联的系统定义的组。因此，组织队列和层次结构的大小和复杂性直接影响记录访问计算时间。
   
# [情景例子](https://developer.salesforce.com/docs/atlas.ja-jp.244.0.salesforce_record_access_under_the_hood.meta/salesforce_record_access_under_the_hood/uth_examples.htm)
## 前提
1. 说明如何计算
2. 不包含所有数据
3. 组织设定为Private
## 从属关系
 1. CEO ->Marc
    1. SalesExecutive ->Maria
       1. East Sales Rep ->Bob
       2. West Sales Rep ->Wendy
    2. ServicesExecutive ->Frank
       1. Services Rep ->Sam
 2. 由上述在 Group Maintance Table中会生成多条记录
## 操作
1. Maria 为 Acme 创建帐户记录 (A1)。
   1.   AccountShare   
        1.  AcctID:数据ID
         2. User/Group:Maria
         3. 原因：Owner
   2. Group Maintenance
      1. Marc -》SalesExecutive ：由于间接分享
2. Maria 直接设置 Acme 的帐户记录以与 Bob 共享
   1.   AccountShare   
        1.  AcctID:数据ID
         2. User/Group:Bob
         3. 原因：Manual
   2. Group Maintenance
      1. Marc* -》East Sales Rep ：由于间接分享
      2. Maria* -》East Sales Rep ：由于间接分享
3. 系统管理员创建共享规则，以便与Services Executive及以下角色的用户共享Sales Executive记录。
   1.   AccountShare   
        1.  AcctID:数据ID
         2. User/Group:Services Executive R&S
         3. 原因：Rule
   2. Group Maintenance
      1. Frank -》Services Executive 
      2. Marc* -》Services Executive  ：由于间接分享
      3. Sam -》Services Executive  
4. Maria は Acme のレコードの所有者を Wendy に変更します。
   1. 当记录的所有者发生更改时，发生以下变化
      1. 原先记录拥有者进行更新
         1. AccountShare
            1. AcctID:数据ID
            2.   User/Group:Maria 变更为Wendy
            3. 原因：Owner
      2. レコードの所有者が変わると、Salesforce は所有者に関連付けられ、共有理由が直接設定となっている共有行を削除するため、Bob はそのレコードへのアクセス権を失います。
         1. AccountShare 删除
            1. AcctID:数据ID
            2.   User/Group:Bob 
            3. 原因：Manual
       3.  此外，作为销售主管，Maria 不再拥有任何记录，因此场景 3 中的规则不再适用。**注意 共享规则只到销售主管这一层**
            1. AccountShare 删除
                 1.  AcctID:数据ID
                 2.   User/Group:Bob 
                 3. 原因： Services Executive R&S
   2. Group Maintenance 表中：Frank 和 Sam 失去了对 Acme 记录的访问权限，因为 Salesforce 删除了场景 3 中创建的服务主管角色的 RoleAndSubscribeds 组中的共享行。
# 综合分析
## 说明
1. Salesforce的用户访问权限是存放于Object Sharing 表与Group Maintenance 表。 
2. 再修改简单场景的时候，只需要修改几条记录。但是往往会出现复杂操作。
3. 我们以更改用户角色 Role 为例来说明这些操作是如何关联的。这个变化看起来像是一个简单的分组操作，但实际上对共享重计算有重大影响。
4. 这是在组织共有设定为不公开的前提下说明的。
5. Role阶层和用户，除了以下两点基本相同
   1. 为中小型业务合作伙伴销售组织创建了新的角色。
   2. 与授予服务分支访问所有销售数据的更广泛的共享规则不同，有一个集中的共享规则，仅授予西部销售代表角色的数据访问权限。 SMB 合作伙伴数据不与服务分支共享。
# 总结
大型组织通常拥有超过 10,000,000 条帐户记录、7,000 个用户、2,000 个角色和 1,000 个区域，这显着增加了复杂性。在为此类组织设计记录访问模型时，牢记这些基础知识将帮助您设计满足组织业务需求的最高效的 Salesforce 实施。