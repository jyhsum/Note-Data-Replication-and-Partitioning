# Data Replication and Partitioning

## Data Replication

當需求量變大，單一資料庫難以負荷，或是為了避免單一資料庫掛掉，而沒有資料庫可以用的情況，又或是你的使用者遍佈世界各地，可以考慮 Replication。
* 優點：
    * 增加讀取效能(N 台資料庫分散可以消化更多查詢請求)
    * 使用者去比較近的節點讀取資料，可降低 latency
    * 分散風險(一台爆了還有其他台可以用)
* 缺點：
    * 需要更多的硬碟儲存空間
    * Replication lag(資料從 master 複製到 slave 的時間差造成的不一致)
    * 花費更多人力管理
    * 需要較高網路頻寬

### Data Replication Types
* #### Data Replication Conponents
    * Articles: replication 最小單位(tables, stored procedures, views…)
    * Publisher: 負責讓資料可以被其他 db server 存取
        * Publications: articles 的邏輯集合
        * Publication database: 負責儲存 article
    * Distributor: 設定怎麼從 publications 收集 transactions 跟如何將 transactions 分配到 subscriber
        * Distribution Database: 儲存 replication status, publication metadata
    * Subscriber: 從 publication 接收資料，可以被動(push)或主動(pull)
        * Subscription database
        * Subscription: 定義要接收的 publication data 以及何時要 pull 跟 push

* #### Snapshot Replication
![](https://i.imgur.com/wv3zNkq.png)

直接將當下的資料 snapshot 下來後傳送給 Subscriber，Replica db 可能會比 Master db 還舊，適合資料量較小、在短期內頻繁變更的情況。

* #### Transactional Replication
![](https://i.imgur.com/uISxr4j.png)

在最一開始初始化 Subscriber 後(用 Snapshot agent 初始化)，便持續將資料以接近 realtime 的頻率傳輸給 Subscriber。會保存每次 transaction 的所有紀錄，所以可以保證資料的正確性與一致性，適合資料變動頻率高的情況。

* #### Merge Replication
![](https://i.imgur.com/qWFl0UR.png)

Publisher 跟 Subscriber 都可以修改資料，Subscriber 在 local 端更改後再與Publisher 跟其他 Subscriber 同步，因為有多個 Publisher 跟 Subscriber 可以變更資料，可能更容易遇到衝突。

* #### Replication Schemes
    * Full Replication：每個節點都是完整的資料庫，每個節點的資料都一樣，重複的資料是三種 Schemes 最多的，資料的完整性與整體的 performance 是最佳。缺點為儲存成本較高，以及更新節點要花費較多時間。
    ![](https://i.imgur.com/TiN33Vg.png)

    * Partial Replication：每個節點存的資料可能會與其他節點重複，依據資料在該節點的重要性決定，需要在 application 層決定要存在哪個節點。
    ![](https://i.imgur.com/55KpkAe.png)

    * No Replication：資料完全不會重複，每個節點只會有原始資料庫的一部份。優點是執行效率較高，但資料完整性以及承受風險的能力較低。
    ![](https://i.imgur.com/rRMTu94.png)


## Data Partitioning

就算已經有做 Replication 了但是單一資料庫的資源跟處理能力仍然有限，因此我們還可以使用 Partition，把邏輯資料庫（或其組成元素，例如資料表）拆分成各個獨立部分，各自處理一部分資料，共同分擔流量。

優點：
* 有效增進效能
* 增加擴充性
* 可以依Partition對資料作處理，增加管理的彈性。
* 快速的刪除資料

缺點：
* 如果 partition 分散在不同資料庫內，可能會使 code 複雜度增加，因為資料間的關聯 (Foreign Key) 必須在 application 寫邏輯去處理。
* 設計不良會降低查詢的效能
* 增加異動資料(Insert、Delete與Update)的時間

### Type of Data Partitioning

![](https://i.imgur.com/tO4qv6y.png)

* #### Horizontal partition：
水平分割，也叫 Sharding，對列進行分拆，會有不同小集合。可以拿其中的欄位當作partition key 例：range/id 奇數、偶數/日期區間/Hash based...。


* #### vertical partition：
垂直分割，根據欄來做分拆，資料庫正規化也算一種 vertical partition。


### Sharding 跟 Horizontal partition 有什麼不同？

水平擴展方式一般來說又可以分為 Horizontal Partitioning 與 Sharding，前者是在同一個資料庫中將 table 拆成數個小 table，後者則是將 table 放到數個資料庫中。Horizontal Partitioning 的 table 與 schema 可能會改變，Sharding 的 schema 則是相同，但分散在不同資料庫中。

#### Ref:
[What Is Data Replication? {Replication Types and Schemes Explained}](https://phoenixnap.com/kb/data-replication)

[SQL Server replication: Overview of components and topography](https://www.sqlshack.com/sql-server-replication-overview-of-components-and-topography/)

[Sharding](https://hazelcast.com/glossary/sharding/)

