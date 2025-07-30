# **业务背景：**

​	在数字化平台发展过程中，随着用户规模持续扩大与行为数据的不断积累，流量与数据逐渐成为平台最具价值的资产之一。为了进一步提升用户生命周期价值、加强平台的运营能力，同时打造自主可控的商业化基础设施，公司决定自研一套高性能、可扩展的广告投放系统。该系统面向多个业务应用场景，支撑启动页广告、信息流、弹窗等多种形式的广告内容投放，提供用户级定向、智能推荐、频次控制、投放监控等一整套完整能力，全面打通“用户—数据—内容—价值”的闭环，成为平台商业变现能力的重要支撑模块。

------

# 二、构建广告系统的动因分析

### 1. 私域流量运营与变现诉求

- 拥有海量实名可识别用户，具备高频使用、长期绑定等特征；
- 原有平台以服务为主，流量变现能力薄弱；
- 自建广告系统可实现首页Banner、弹窗、信息流等多入口广告投放，提升整体营收能力。

------

### 2. 构建智能定向与标签体系

- 平台沉淀了丰富的用户行为数据，包括设备类型、操作系统、地域、兴趣偏好等；

------

## **联系方式 & 更多分享**

如果您对本文档中的技术细节、架构设计有任何疑问或希望进行技术交流，欢迎通过以下方式联系我：

- **微信：** [请在此处填写您的微信号]
- **邮箱：** [请在此处填写您的邮箱地址]
- **个人博客/CSDN：** [请在此处填写您的博客或CSDN链接]
- **GitHub：** [请在此处填写您的GitHub个人主页链接]

期待与您共同探讨技术！

- 具备构建用户画像与人群标签的技术基础；
- 可支持精准定向、多维交叉组合、实时人群匹配，增强投放效果。

------

### 3. 实现合规可控的广告体系

- 面对日益严格的广告监管与数据保护法规，自研系统可保证数据在链路内可控；
- 支持素材审核、频控限流、日志留存等合规能力；
- 避免使用第三方平台造成的用户数据外泄或隐私风险。

------

### 4. 补齐平台商业化能力短板

- 公司已有较为完整的中台与基础设施能力（如用户中台、数据中台、支付平台等）；
- 广告系统作为商业化组件，可与中台系统无缝集成；
- 具备使用 Netty、Kafka、ClickHouse、Redis 等高性能技术栈支撑工程落地的能力。

------

### 5. 行业趋势推动与市场对标压力

- 越来越多互联网平台、垂直行业产品、公共服务应用开始建设自有广告系统；
- 自建系统成为增强平台营收能力、提升技术护城河的重要方式；
- 广告系统作为“低侵扰、强变现”的能力模块，可长期持续优化与扩展。

# **广告系统要求：**

​	第一：支持超高业务流量。当前平台系统极限可支持超过10W/S(QPS)的请求,实际使用过程中至少承受超过5W/S(QPS)的请求,同时也具有快速扩容能力；

​	第二：支持超低延迟。广告服务对于请求时间非常敏感，平均内部响应耗时控制在20ms以下,120ms超时率≤0.05%,远低于流量平台2%的阈值；

​	第三：监控告警。可监控和告警实时流量、接口耗时、超时率、数据存储等各项数据。



# **整体架构**

### 业务架构图

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1751313072408-bd7b2b03-3716-46a3-8904-f73a21d0ae7b.png)

### **k8s架构图：**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476062377-c2d5ad46-a3ed-4da2-a8f6-ba4241cb473b.png)

### **集群架构**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476062108-f61a7b5d-9238-4c8a-92bf-e30ae9a9d7e6.png)

### **整体业务流程图**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1753693485280-cf27965d-b950-42f5-be27-de55743f090e.png)

# **功能设计**

## 广告高并发服务器

​	广告系统对服务端性能和稳定性的要求极高，尤其是在面对海量请求、高并发压力以及复杂业务逻辑时。Netty作为一个高性能的异步事件驱动网络应用框架，是很多大型互联网广告系统的首选技术方案。下面从多个维度详细分析为什么选择Netty作为广告系统的Server。

#### 1. 高并发性能，满足海量请求的挑战

广告系统往往每天处理亿级甚至更高数量级的请求，单机QPS（每秒查询率）需求极大。Netty基于Java NIO实现，采用Reactor多路复用机制，避免了传统阻塞IO模型中每连接一个线程的资源浪费。

- **事件驱动架构**：Netty通过事件循环（EventLoop）来调度和处理网络事件，线程数固定且复用，极大减少线程切换和上下文切换开销。
- **非阻塞IO**：请求处理不依赖阻塞等待，响应速度快，延迟低。
- **高吞吐量**：通过零拷贝和直接内存访问优化，减少数据在内核和用户空间的复制次数，提高传输效率。

这让广告系统能在有限硬件资源下支持数十万甚至百万级并发连接。

------

#### 2. 低延迟设计，提升广告响应速度

广告投放系统对响应时延非常敏感，延迟越低用户体验越好，广告匹配速度越快带来的收益越大。Netty在设计上非常注重减少延迟：

- **零拷贝技术**：减少数据拷贝次数，降低CPU和内存负担。
- **高效内存管理**：通过ByteBuf池化管理，避免频繁分配和回收，降低GC压力。
- **支持异步非阻塞处理**：避免请求线程阻塞，提高资源利用率，保证请求能快速响应。

------

#### 3. 灵活的协议支持和扩展性

广告系统业务复杂，需要处理多种协议和自定义通信格式。Netty天生支持多协议：

- TCP、UDP、HTTP/HTTPS、HTTP2、WebSocket等多种协议；
- 支持自定义协议编解码器，方便与业务层结合，实现广告请求的精准解析与构建；
- 灵活的Pipeline机制，业务逻辑可以模块化拆分，便于维护和扩展。

这保证了系统未来迭代过程中，可以平滑升级通信协议，兼容更多业务场景。

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476063798-bb516a01-229c-4618-b5e8-23302b3d9b9e.png)

​	

## 广告定向检索

### 背景说明

在广告系统中，为了实现高性能、可扩展的多维度定向过滤，我们使用 **RoaringBitmap + Redis** 来构建广告计划的倒排索引，加速广告筛选过程。

RoaringBitmap 特别适用于：

- 高基数、大范围 ID 集合管理（如上万个广告计划）；
- RoaringBitmap采用分块压缩技术，能够在保证高速访问的同时，极大节省存储空间
- 广告定向过滤的本质是多个条件集合的交集、并集等集合运算。RoaringBitmap针对这些操作进行了深度优化，能够以亚毫秒的速度完成千万级数据的集合计算，极大提升了广告匹配的实时性
- 占RoaringBitmap序列化后的数据体积小，便于存储和网络传输，适合分布式系统中缓存和索引的同步更新。新增或修改广告计划，只需更新相关的位图索引，极大简化了维护成本。



## 各定向维度设计

广告过滤链路主要围绕多个定向维度进行快速判断，典型流程如下：

| 维度         | 描述                                     | 实现方式                   |
| ------------ | ---------------------------------------- | -------------------------- |
| 地域定向     | 城市/省份/国家等                         | 基于IP解析与城市码匹配     |
| 时段定向     | 一周内具体小时段                         | BitSet 位图（168位）       |
| 人群标签定向 | 兴趣、年龄、性别、职业、设备、行为等标签 | RoaringBitmap 标签交集判断 |
| 预算与频控   | 用户点击上限、投放节奏等                 | Redis bitmap + 分时 pacing |
| 黑白名单     | 用户/设备/渠道等黑白名单                 | 布隆过滤器 + 缓存命中判断  |





### 定向代码demo：

##### 添加到指定维度的RoaringBitmap中

```java
function syncSingleDimension(dimensionType, dimensionValue, adPlanId):
    redisKey = "rb:" + dimensionType + ":" + dimensionValue
    
    // 读取 Redis 中的 Base64 编码的 Bitmap
    base64 = Redis.get(redisKey)
    if base64 != null:
        bytes = Base64.decode(base64)
        bitmap = RoaringBitmap.deserialize(bytes)
    else:
        bitmap = new RoaringBitmap()
    
    // 将广告计划ID加入 Bitmap
    bitmap.add(adPlanId)
    
    // 序列化并 Base64 编码写回 Redis
    encoded = Base64.encode(bitmap.serialize())
    Redis.set(redisKey, encoded)
```



##### 从维度中移除广告计划

```java
function removeAdPlanFromDimension(dimensionType, dimensionValue, adPlanId):
    redisKey = "rb:" + dimensionType + ":" + dimensionValue
    encoded = Redis.get(redisKey)
    
    if encoded is null:
        return
    
    bitmap = RoaringBitmap.deserialize(Base64.decode(encoded))
    bitmap.remove(adPlanId)
    Redis.set(redisKey, Base64.encode(bitmap.serialize()))
```



##### 筛选出符合定向要求的计划

```java
function filterAdPlansByDimensions(dimensions):

    if dimensions is empty:
        return empty set

    redisKeys = []
    for (dimensionType, dimensionValue) in dimensions:
        redisKeys.append("rb:" + dimensionType + ":" + dimensionValue)

    base64Bitmaps = Redis.multiGet(redisKeys)

    if any value in base64Bitmaps is null:
        // 有维度无匹配，直接无广告计划
        return empty set

    resultBitmap = null

    for base64 in base64Bitmaps:
        bitmap = RoaringBitmap.deserialize(Base64.decode(base64))
        if resultBitmap is null:
            resultBitmap = bitmap
        else:
            resultBitmap.and(bitmap)  // 求交集

    if resultBitmap is null:
        return empt
```



### 黑名单过滤（Blacklist）

#### 需求背景

- 黑名单：不允许某类用户或设备看到某些广告；
- 白名单：仅允许某些 ID 命中的广告计划被投放。

#### Key设计

- `rb:blacklist:{userId}` → bitmap of adPlanIds（该用户禁看的广告）
- `rb:whitelist:{userId}` → bitmap of adPlanIds（该用户可看的广告）

#### 

### 操作系统定向（OS）

#### 需求背景

- 广告主可选择投放给 Android 或 iOS 用户；
- 系统需根据设备类型快速过滤。

#### Key设计

- `rb:os:android`
- `rb:os:ios`

#### B端写入示例

```java
syncSingleDimension("os", "android", adPlanId);
```

#### C端过滤示例

```java
函数 filterAdPlansByConditions(维度Key列表):
    如果维度Key列表为空:
        返回空集合

    定义变量 resultBitmap = 空

    对于每一个维度Key in 维度Key列表:
        从Redis中读取Base64编码的位图数据

        如果数据为空:
            跳过当前维度

        将Base64数据解码为字节数组
        反序列化为RoaringBitmap对象，记为 currentBitmap

        如果 resultBitmap 为 null:
            将 currentBitmap 赋值给 resultBitmap
        否则:
            resultBitmap 与 currentBitmap 做交集运算（AND）

    如果 resultBitmap 仍为 null:
        返回空集合

    将 resultBitmap 中的所有值收集为广告计划ID集合
    返回该集合
```

------

### 广告位定向（Ad Slot）

#### 需求背景

- 广告主选择将计划绑定到具体的广告位，例如首页Banner；
- 系统需根据请求中广告位 ID 做过滤。

#### Redis Key

- `rb:slot:homepage_banner`
- `rb:slot:launch_popup`

#### 示例

```java
syncSingleDimension("slot", "homepage_banner", adPlanId);
filterAdPlansByConditions(List.of("slot:homepage_banner"));
```



### 地域定向

####  需求背景

- **IP库构建与更新：** 上线初期采用静态IP库，后续通过定期从权威第三方数据源同步并更新，确保IP地址归属地信息的实时性和准确性。
- **IP解析与缓存：** 广告请求中的用户IP通过IP库解析为`cityId`。为提高效率，解析结果会进行多级缓存（如本地缓存、分布式缓存），减少重复计算。
- **高性能索引：** 采用RoaringBitmap对`cityId`进行索引，实现亚毫秒级的地域集合运算，满足高并发查询需求。



```java
地域定向Bitmap：
cityId=101 → RoaringBitmap(planIds: 12, 56, 89)
cityId=201 → RoaringBitmap(planIds: 13, 22, 77)
```

后续使用RoaringBitmap来做交并集计算。

地域定向RoaringBitmap格式，后面的数字是具体的广告	

#### Key设计

- `geo:region:{cityId} -> {101,102,105}`



### 时段定向

####    一、业务背景：

​	该模块用于支持广告系统中的时段定向投放，实现“在一周的某些具体时段内，允许指定广告计划投放”的功能。 核心思路是将 星期 + 小时 映射为唯一的时段索引（0~167），并使用高效压缩结构 RoaringBitmap 来存储每个时段下允许投放的广告计划ID集合，支持快速匹配与高效扩展。

------

#### 二、设计思路

- 使用 **BitSet** 表示每周 7 天 × 每天 24 小时 = 168 个小时的时间投放控制位。
- 每个广告计划维护一个 168 位的 BitSet，表示哪些时间可投。
- 使用 Redis 存储广告计划的时间投放位图，Key 为计划ID，Value 为 Base64 编码的 BitSet。
- 投放时只需取出当前小时在一周中对应的 BitSet index，批量与操作过滤可投计划。

------

#### 三、BitSet 编码规则

```plain
BitSet 长度: 168 位
索引计算方式:
  index = (dayOfWeek - 1) * 24 + hourOfDay
  dayOfWeek: 周一=1，周日=7
  hourOfDay: 0~23
例如：周二 10 点  -> index = (2-1)*24 + 10 = 34
 public static final int HOURS_PER_WEEK = 7 * 24;

    /**
     * 创建 BitSet 表示的投放时段（小时范围）
     * 例如：周二8-11点 => weekDays = [2], hours = [8,9,10,11]
     */
    public static BitSet buildBitSet(List<Integer> weekDays, List<Integer> hours) {
        BitSet bitSet = new BitSet(HOURS_PER_WEEK);
        for (Integer day : weekDays) {
            for (Integer hour : hours) {
                int index = day * 24 + hour;
                if (index >= 0 && index < HOURS_PER_WEEK) {
                    bitSet.set(index);
                }
            }
        }
        return bitSet;
    }
```

------

#### 四、数据结构设计

##### Redis Key 设计：

| Key 格式                  | 类型   | Value 示例             |
| ------------------------- | ------ | ---------------------- |
| ad:plan:timeslot:{planId} | String | Base64 编码后的 BitSet |

##### BitSet 存储样例（Base64）：

```shell
Key: ad:plan:timeslot:1001  
Value: AQIDBA==   // 实际为 BitSet 的序列化后 Base64 编码
```

------

#### 五、代码实现

##### 工具类

```java
public class TimeSlotBitSetUtil {

    public static final int BIT_SIZE = 7 * 24;

    public static BitSet parseSlotToBitSet(Set<Integer> slotIndices) {
        BitSet bitSet = new BitSet(BIT_SIZE);
        for (Integer index : slotIndices) {
            if (index >= 0 && index < BIT_SIZE) {
                bitSet.set(index);
            }
        }
        return bitSet;
    }
}
```



#### 六、投放时间配置入口示例（运营后台）

将投放时间转换为 BitSet 索引：

- 周五 8 点 → index = (5 - 1) * 24 + 8 = 104
- 周二 16 点 → index = (2 - 1) * 24 + 16 = 40

示例：

```java
Set<Integer> timeSlots = Set.of(40, 104); // 支持多时段
BitSet bitSet = TimeSlotBitSetUtil.parseSlotToBitSet(timeSlots);
String encoded = TimeSlotBitSetUtil.encodeBitSet(bitSet);
redisTemplate.opsForValue().set("ad:plan:timeslot:1001", encoded);
```

------

#### 七、总结优势

| 项目     | 说明                                           |
| -------- | ---------------------------------------------- |
| 存储结构 | 168 位位图，结构紧凑，Base64 编码后仅几十字节  |
| 查询效率 | 批量使用 `multiGet`，结合 BitSet 判断，性能高  |
| 工具解耦 | BitSet处理逻辑集中于工具类，业务层关注投放过滤 |

------

#### 使用示例

```java
TimeSlotRoaringIndex index = new TimeSlotRoaringIndex();

// 计划 1001：周一20点-22点
index.addPlanToTimeSlot(1001, DayOfWeek.MONDAY, 20, 22);

// 查询当前可投计划
ZonedDateTime now = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
RoaringBitmap plans = index.getPlansAtTime(now);
```



### 频次控制

####   业务背景

在广告投放过程中，如果某个广告对同一用户重复展示次数过多，容易造成：

- **用户体验下降**：频繁看到相同广告，导致用户产生厌烦甚至反感
- **预算浪费**：广告主为已触达用户持续付费，却未产生新增转化
- **心理疲劳效应**：CTR（点击率）与CVR（转化率）通常在高频曝光下持续下滑

因此，**对广告展示或点击设置频控上限**，已成为广告平台提升投放效果与用户体验的基本能力。

#### 频次Key结构:

```java
  "frequency_" + adPlanId + "_" + frequencyType + "_" + uuid;
```

#### 具体实现：

该方法用于在广告投放过程中，**根据点击频次上限（FrequencyCap）过滤掉超出限制的广告计划**，防止单个用户在一定时间内重复点击某广告计划过多次，影响投放效果与体验。

```plain
函数 matchFrequencyClick(adPlans):

    如果 adPlans 非空:
        构造 Redis Key 列表 = 每个广告计划的 frequencyClickKey

        从 Redis 批量获取这些 key 的点击数值列表

        将 key 列表与 value 列表合并成 map：key -> 已点击次数

        遍历所有 adPlans，筛选出符合频次要求的广告计划：
            条件是：map 中该 key 对应的值 < adPlan.frequencyNum（频次上限）

        返回筛选后的广告计划数组

    否则：
        返回空数组或原数组
```



### 均匀投放

#### 1：业务背景

在广告投放中，常见的计划预算策略包括：

- **抢量投放**：计划一开启，尽可能快速抢占曝光/点击；
- **均匀投放**：希望将预算**合理平滑分配到全天各时段**，控制成本并提升转化。

**本模块解决的问题**是第二种 —— 在 CPC（按点击计费）模式下，根据投放剩余分钟数动态计算“当前小时应该投放多少点击”，并作为频控、节奏控制的依据。

------

#### 2：核心设计思想

| **核心点**          | **说明**                                           |
| ------------------- | -------------------------------------------------- |
| 动态分配点击        | 每分钟刷新，根据预算剩余/时间剩余计算hourClick     |
| 基于 Redis 实时统计 | 从 Redis 获取“当前已投点击”，不依赖Database        |
| 热补偿机制          | 当前小时点击已经消耗部分需“加回”预算，避免重复限制 |
| 每分钟调度刷新      | 保证计划状态“准实时”反映，适配用户流量波动         |
| 与投放线程解耦      | 单独线程或定时任务负责刷新，不影响主链路性能       |

#### 3：关键方法汇总

##### 3.1 cpc均匀投放点击计算

```plain
函数 calculateClick(adPlan):

    如果 adPlan 配置了每日点击预算:
        已点击 = 从 Redis 获取 "当日已点击数"
        剩余点击数 = 日预算 - 已点击
    否则:
        已点击 = 从 Redis 获取 "总点击数"
        剩余点击数 = 总预算 - 已点击

    当前小时已投放点击数 = 从 Redis 获取 "当前小时已投放数"
    剩余点击数 += 当前小时已投放数（加回去，防止重复统计）

    如果 剩余点击数 > 0 且 广告剩余分钟数 > 0:
        剩余小时数 = 向上取整(剩余分钟数 / 60)
        每小时应投放点击数 = 剩余点击数 / 剩余小时数
    否则:
        每小时应投放点击数 = 0

    设置 adPlan.hourClick = 每小时应投放点击数
```

##### 3.4 调度器设置

```plain
组件 UniformDeliveryScheduler:

  每隔 N 秒执行一次:

      调用 deliveryService.calculate()
          -> 遍历所有处于投放状态的广告计划
          -> 对每个广告计划执行 calculateClickData(adPlan)
```

### 量级控制

#### 一、功能目标

本模块的核心目的是在广告投放流程中，实现对 CPC（按点击付费）广告计划的**日点击上限控制**。通过实时查询广告计划当天已消耗的点击次数，判断是否超过预设的点击上限，从而过滤掉点击数&预算达到或超过上限的广告计划，防止超额投放，保障广告主预算的合理使用。

#### 二、业务背景

- 广告系统中，广告主通常会给广告计划设置**每日点击上限**（或者预算），以限制广告计划每天的最大点击量，避免点击费用超出预算。
- 当广告请求到达广告系统时，系统需要在竞价环节之前快速校验广告计划的当前点击数，判断其是否还允许继续投放。
- 由于广告请求量大，点击数据存在 Redis 中，并采用批量读取方式以保证高性能。
- 该机制保障了广告预算的合理控制，防止无效点击或过度曝光造成的资源浪费。

#### 核心代码

```java
函数 matchDailyClickLimit(adPlans):

    如果 adPlans 非空:
        dayClickKeys = 提取每个计划的 Redis 日点击统计 key
        dayClickValues = 批量从 Redis 查询这些 key 的当前值
        dayClickMap = 将 keys 和 values 合并为一个 Map

        adPlans = 对每个 adPlan 执行以下过滤逻辑:
            如果日点击预算 <= 0 或 单次点击价格 <= 0:
                保留该计划
            否则，如果该计划为 CPC 模式:
                当前点击次数 < 设置的每日点击上限
                    -> 保留
                否则
                    -> 过滤

    返回过滤后的 adPlans
```

### 人群画像定向

####     为什么要用Hbase？

​	  HBase是列式存储，能按需存储列，不存在固定列结构，天然支持稀疏列和灵活的Schema，非常适合多维度标签画像数据的存储，广告人群画像通常包含亿级甚至百亿级用户的数据，且每个用户会有多维度、多类型的标签和行为数据，广告画像的主体数据往往是**大规模冷数据+准实时更新**HBase天生支持水平扩展，自动分区，适合海量数据扩容，Hbase多数时不适合做实时用户画像的检索和抉择，但我方并未使用Redis或者Caffine来做热点画像存储。并且时延能够满足媒体需求，同时，其他兄弟部门也在公用Hbase人群画像库，共享技术生态，避免重复搭建和维护多套大数据存储系统，节省运维成本和风险。



#### 1. 设计思路

- - **RowKey设计**：以`userId`作为RowKey，唯一定位单个用户画像。

- **列族设计**：定义一个或多个列簇，如`info`，存储用户画像标签。
- **列限定符（Qualifier）设计**：每个画像标签作为一个列，如年龄、性别、兴趣爱好等。
- **数据特点**：

- - 多维度，标签稀疏，不同用户标签不完全相同。
  - 支持标签的动态扩展，方便新增或修改标签。
  - 支持多版本数据，方便用户画像更新和历史分析。

#### 2. 查询流程

- 根据userId快速定位RowKey。
- 读取该用户所有画像标签，进行画像判断或定向过滤。

# **存储设计**

## **Clickhouse设计**

### **物化视图设计：**

```plsql
CREATE MATERIALIZED VIEW ad_event_agg_mv
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(server_time)
ORDER BY (day, advertiser_id, campaign_id, creative_id)
AS
SELECT
    toDate(server_time) AS day,
    advertiser_id,
    campaign_id,
    creative_id,
    countState() AS event_count_state,
    countIfState(event_type = 'StandExpo') AS standexpo_count_state,
    countIfState(event_type = 'PutExpo') AS putexpo_count_state,
    countIfState(event_type = 'PutClick') AS putclick_count_state,
    sumState(price) AS total_cost_state
FROM ad_event_detail
GROUP BY
    day,
    advertiser_id,
    campaign_id,
    creative_id;
```

### **广告明细表设计：**

```plsql
CREATE TABLE ad_event_detail
(
    -- 时间字段
    server_time DateTime,             -- 服务端接收时间（主时间维度）
    event_time DateTime,              -- 事件发生时间（上游字段，可以用 ALIAS）
    -- 业务维度
    advertiser_id String,             -- 广告主ID
    campaign_id String,               -- 广告计划ID（对应你物化视图里的 adplan_id）
    media_id String,                  -- 媒体ID
    ad_slot_id String,                -- 广告位ID
    creative_id String,               -- 创意ID
    device_id String,                 -- 设备ID，用户唯一标识
    city_id UInt32,                   -- 城市ID（原 IP 解析）
    -- 操作系统类型，枚举类型
    os_type Enum8('ios' = 1, 'android' = 2, 'windows' = 3, 'macos' = 4, 'other' = 5),
    -- 事件类型，明确枚举
    event_type Enum8('StandExpo' = 1, 'PutExpo' = 2, 'PutClick' = 3),
    -- 请求唯一ID，用于去重或追踪
    request_id String,
    -- 扩展字段，灵活存储业务自定义字段
    extend1 String,
    extend2 String,
    -- 页面视图编码，用于业务分析不同页面的效果
    view_code String,
    -- 价格、消耗，单位可根据业务调整
    price Float64 DEFAULT 0,
    -- 用户画像维度，可选保留或拆分成数组
    gender LowCardinality(String),
    age LowCardinality(String),
    -- 唯一键，用于重复数据去重，通常拼接业务关键字段或使用唯一 request_id
    uniq_key String,
    -- 版本字段，辅助去重时保留最新数据
    version UInt64 DEFAULT toUnixTimestamp(server_time)
)
ENGINE = ReplacingMergeTree(version)
PARTITION BY toYYYYMM(server_time)
ORDER BY (
    advertiser_id,
    campaign_id,
    media_id,
    ad_slot_id,
    creative_id,
    server_time
);
```

​	物化视图聚合表引擎采用[AggregatingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/aggregatingmergetree)，此引擎用于存储和查询预计算的聚合数据。适合高效地存储大规模的聚合结果，一般用于针对明细表的聚合表。

### **索引设计**

```plsql
INDEX idx_server_time server_time TYPE minmax GRANULARITY 8192,
INDEX idx_event_type  event_type  TYPE minmax GRANULARITY 8192,
INDEX idx_advertiser_id advertiser_id TYPE set(0) GRANULARITY 8192,
INDEX idx_campaign_id campaign_id TYPE set(0) GRANULARITY 8192,
INDEX idx_creative_id creative_id TYPE set(0) GRANULARITY 8192

SETTINGS index_granularity =8192;
```

server_time 和 event_type 用 minmax 索引适合范围过滤。advertiser_id、campaign_id、creative_id 这类字符串主维度字段用 set 索引，对精确过滤效果好，且开销相对小。索引不会替代排序键，但能有效辅助过滤。



## **Hbase表设计**

### **Rowkey设计**

​	因广告标准化要求，**设备号都是基于MD5格式(32位)进行传输**，因为MD5生成的哈希值(通常是32个字符的16进制字符串)可以将数据均匀分布到HBase的Region中，**避免了热点问题**。因为MD5是32字节的16进制字符串，它的哈希值在理论上是非常随机的，这可以有效地避免数据倾斜。而且在大并发情况下，不需要对Rowkey做二次计算，故**Hbase表的Rowkey设计遵从选择MD5作为Rowkey设计方案**，同时，因每一个设备号都伴随着数据类型参数，所以，Hbase表通过类型做表的区分(OAID、IMEI、IDFA)作为区分不同表的依据。

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476063175-81f08318-9ab5-4823-a986-626039883478.png)

### **Region大小设计**

​	在Hbase中，分区是基本的存储单元，当数据量较大时，HBase 会**自动(\****默认情况下，当一个 Region 达到 10GB 的大小)将数据分成多个 Region**，每个 Region 存储数据的一部分。每个 Region 会映射到一个 RegionServer 上进行管理。合理地分配和管理 Region 可以帮助提高性能、避免热点问题以及实现更好的负载均衡,每个 Region 包含一个连续的行范围(startRowKey，endRowKey)。一般来说，**Region** 的大小应控制在 **10GB 到 20GB** 之间(这个不强求，根据自己系统的实际需要来判断)，如果 Region 大小过小（例如 < 1GB），会导致过多的 Region 会占用过多的资源（比如内存、文件句柄等），增加 RegionServer 的负担，如果 Region 太大，会导致负载不均衡，某些 Region 可能会成为瓶颈，造成负载集中，写入数据到过大的 Region 时可能会增加延迟，影响系统的吞吐量，如果单个 Region 的数据太多，可能导致 RegionServer 内存压力增大，影响性能。

用户的的Hbase相关Region设计如下：

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739477886578-553a3ee0-427c-49ab-b25c-eb3a6e300aea.png)

## **Redis设计**

### **异步操作**

​	因广告系统中有大量Key需要被增量记录次数，同时此操作不应该影响主业务线程逻辑，故采用异步操作来进行增量添加，Spring Data框架中提供了[ReactiveStringRedisTemplate](https://docs.spring.io/spring-data/redis/reference/api/java/org/springframework/data/redis/core/ReactiveStringRedisTemplate.html) 来进行异步操作，异步操作不会阻塞调用线程，可以提高系统的并发性能，尤其是在需要大量 Redis 操作的应用中：

```java
/**
 * 异步增量添加数据
 *
 * @param key           Redis的Key
 * @param increaseValue 要增量的值
 */
private void incrementAsync(String key, long increaseValue) {
    reactiveStringRedisTemplate.opsForValue().increment(key, increaseValue);
}
```

### **批处理**

#### **批量获取：**

​	如以下代码所示：通过传入Key的列表对象，获取Value的列表(**Value的顺序和Key的顺序是一致的，如果Key不存在，那么获取的Value值为null**)，

这段代码的主要功能是将两个列表 keyList 和 valueList 根据索引位置合并成一个 Map，其中 keyList 中的元素作为键，valueList 中的元素作为值。代码首先检查两个列表的大小是否一致，然后使用 Stream.iterate() 创建一个索引流，接着通过 Collectors.toMap() 收集成 Map。如果两个列表大小不一致，则抛出一个异常来避免错误。这种方法适合于**两个列表长度相同的情况**，并且能够高效地将两个列表合并为一个映射。



**批量Set：**

​	Spring Data框架中提供了multiSet，传入的是一个Map对象，Key就是Redis的Key，Value则是此Redis的实际值。

```java
/**
 * 批量设置Redis数据
 *
 * @param redisMap Redis的键值对Map，其中Key为Redis的Key，Value为Redis的具体值
 */
private void multiSetAsync(Map<String, String> redisMap) {
    stringRedisTemplate.opsForValue().multiSet(redisMap);
}
```

​	总结：虽然redis是一个基于内存的缓存神器，操作速度奇快，但是单次操作超快也架不住数以千万甚至亿级的操作，此时如果还是用迭代的单条操作可能带来灾难性的后果，所以除了单条操作外，建议在**不影响主进程业务**的情况下，应当尽可能的使用异步处理，同时，在具体实现中尽量使用批量操作，以减少与中间件中的RPC次数(尽管是内网)。

## **Kafka设计**

​	在广告系统中，**Kafka**帮助系统解耦多个业务模块，提供松耦合的架构，广告系统系统通常需要高吞吐量、低延迟的系统来保证广告实时性和精准性。Kafka 支持大规模并发数据流，并能保证消息的快速消费。Kafka支持将广告数据流从实时系统传递到数据仓库（如HDFS、Hive、ClickHouse），并为数据仓库提供增量数据同步的能力。

- 在线系统： 在实时广告竞价、流量分析等过程中，Kafka 作为缓冲区暂时存储传入的数据，流处理引擎可以后续进行消费。
- 离线系统： Kafka 可以将历史广告数据传送到离线系统用于批量数据处理（如广告效果分析、用户行为建模等）。

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/12670791/1753722526898-829d4855-3603-474a-b5b1-2bfd22318fe5.jpeg)

### **分区设置**

​	Kafka的吞吐量主要依赖于**分区数和消费者**的并发能力，每个分区只能被一个消费者在同一个消费者组中消费。因此，消费者的数量不能超过分区的数量，kafka分区和节点数成倍数关系，这样分区分布的会比较均衡，因为广告消费者系统(离线项目)已知有6个节点，故kafka的节点数设置为12或者24。

### **生产者配置**

```yaml
producer:
  main:
    bootstrapServers: 
    #bootstrapServers: placeholder.kafka.example.com
    #消息key的序列化方式
    keySerializerClass: org.apache.kafka.common.serialization.StringSerializer
    #消息值的序列化方式
    valueSerializerClass: org.apache.kafka.common.serialization.StringSerializer
    #接收回执
    acks: 1
    #producer用于缓存的内存大小(字节)(吞吐量参数)
    bufferMemory: 33554432
    #producer发送消息的重试次数
    retries: 0
    #producer发送消息的每个批次到指定分区的最大发送字节数(字节)(吞吐量参数)
    batchSize: 16384
    #producer发送消息的每个请求的最大发送字节数(字节)(吞吐量参数)
    requestSize: 1048576
    #producer底层TCP的发送缓冲区(字节)(吞吐量参数)
    sendBuffer: 131072
    #producer底层连接的最大空闲时间(毫秒)
    connectionsIdle: 540000
    #producer每个请求的最长等待时间(毫秒)
    blockTime: 30000
    #producer一个connection可以发送的最大请求数(吞吐量参数)
    connectionFlight: 5
    #producer消息延迟发送的等待时间(毫秒)(吞吐量参数)
    linger: 5
    #producer消息压缩类型 压缩速度snappy<gzip 批量发送下有效节省带宽和磁盘空间 (吞吐量参数)
    compression: gzip
    #producer实例个数
    instances: 1
```

### **消费者配置**

```yaml
consumer:
  realLabelEvent:
    bootstrapServers: placeholder.kafka.example.com
    #消费者订阅的topic列表,分割
    topic: example_topic
    #消费组ID
    groupId: example_group
    #消息key的序列化⽅式
    keyDeserializerClass: org.apache.kafka.common.serialization.StringDeserializer
    #消息值的序列化⽅式
    valueDeserializerClass: org.apache.kafka.common.serialization.StringDeserializer
    #consumer底层连接的最⼤空闲时间(毫秒)
    #connectionsIdle: 540000
    #fetch时最⼩拉取的字节数
    fetchMinBytes: 1
    #fetch时每个分区最⼤拉取的字节数
    partitionMaxFetch: 1048576
    #fetch时每个分区最⼤拉取记录数
    pollRecords: 100
    #consumer底层TCP的接收缓冲区(字节)(吞吐量参数)
    receiveBuffer: 131072
    #consumer底层TCP的请求超时时间(毫秒)
    requestTimeout: 30000
    #web session过期时间(毫秒)
    sessionTimeout: 25000
    #consumer业务实例
    instances: 1
    #是否异步接收数据，默认异步，设成false的话为同步
    async: false
    #consumer线程池配置 (新增)
    pool:
      capacity: 200
      coreSize: 5
  offlineLabelEvent:
    bootstrapServers: placeholder.kafka.example.com
    #消费者订阅的topic列表,分割
    topic: example_topic
    #消费组ID
    groupId: example_group
    #消息key的序列化方式
    keyDeserializerClass: org.apache.kafka.common.serialization.StringDeserializer
    #消息值的序列化方式
    valueDeserializerClass: org.apache.kafka.common.serialization.StringDeserializer
    #fetch时每个分区最大拉取的字节数
    partitionMaxFetch: 1048576
    #fetch时每个分区最大拉取记录数
    pollRecords: 1000
    #consumer底层TCP的接收缓冲区(字节)(吞吐量参数)
    receiveBuffer: 131072
    #consumer底层TCP的请求超时时间(毫秒)
    requestTimeout: 30000
    #web session过期时间(毫秒)
    sessionTimeout: 25000
    #consumer业务实例
    instances: 1
    #是否异步接收数据，默认异步，设成false的话为同步
    async: false
    #consumer线程池配置 (新增)
    pool:
      capacity: 200
      coreSize: 5
      maxSize: 10
```

​	以上为kafka的生产者和消费者的基础配置，其中kafka传输全部基于JSON，并且全部为异步执行，**Kafka**帮助系统解耦多个业务模块，提供松耦合的架构，

## **Caffeine缓存设计**

​	  **广告系统** 中，除了将不可变字段存储在 **Caffeine** 中以减少对 **Redis** 的访问频率外，还需要对签名验证等操作进行优化。可以将 **签名验证** 和 **规则判断** 也结合到 Caffeine 中，以进一步提高系统性能和降低延迟。

部分对接的媒体需要检验secret+当前时间的组合作为此次Request是否valid的标志，而secret属于在初始化媒体信息时候就会带有的字段，不会改变，所以将其放入本地缓存中，通过与currentTime做组合，来判断本地请求是否合法；	

```java
public LocalCacheComponent() {
    this.cache = Caffeine.newBuilder()
        //写入或者更新10分钟后，缓存过期并失效，根据具体情况设置即可
        .expireAfterWrite(10, TimeUnit.SECONDS)
        // 初始的缓存空间大小
        .initialCapacity(500)
        // 缓存的最大条数，通过 Window TinyLfu算法控制整个缓存大小
        .maximumSize(5000)
        //打开数据收集功能
        .recordStats()
        .build();
}

public String get(String key) {
    // 首先尝试从一级缓存中获取
    String value = cache.getIfPresent(key);
    if (value == null) {
        // 一级缓存未命中，从二级缓存中获取
        RBucket < String > bucket = redissonClient.getBucket(key, new StringCodec());
        value = bucket.get();
        log.debug("从redis获取值");
        // 如果二级缓存中有值，则将其放入一级缓存中
        if (value != null) {
            cache.put(key, value);
        }
    } else {
        log.debug("从本地获取值");
    }
    return value;
}
```

# **广告系统遇到的挑战**

## **OOM导致业务处理宕机**

### **Netty设置水位**

​	Netty本身是基于出站入站的事件驱动的框架，业务整体流程如下：**业务handler -> write -> ChannelOutboundBuffer -> flush -> SO_SEND_BUF -> 网卡****，**当网络出现阻塞或者Netty服务端负载(QPS)很高的时候，服务端接收速度和处理速度越来越慢，或者直白点说，出站速度跟不上入站速度， 会出现什么情况？？

- TCP的滑动窗口不断缩小，以减少网络数据的发送，直到为0；
- Netty服务端有大量频繁的写操作，不断写入到ChannelOutboundBuffer(出站缓冲区)中；
- 但ChannelOutboundBuffer中的数据flush不到SO_SEND_BUF中，导致ChannelOutboundBuffer中的数据不断堆积，最终撑爆ChannelOutboundBuffer，导致内存溢出而宕机。

```java
@Override
public void run() {
    try {
        WriteBufferWaterMark writeBufferWaterMark = new WriteBufferWaterMark(512 * 1024, 1024 * 1024 * 2);
        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(BOSS_GROUP, WORKER_GROUP)
        .channel(NioServerSocketChannel.class)
        .childHandler(new PMPServerInitializer())
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 1000)
        .option(ChannelOption.SO_BACKLOG, 10000)
        .option(ChannelOption.SO_RCVBUF, 1024 * 1024)
        .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
        .option(ChannelOption.RCVBUF_ALLOCATOR, AdaptiveRecvByteBufAllocator.DEFAULT)
        .option(ChannelOption.WRITE_BUFFER_WATER_MARK, writeBufferWaterMark)
        .childOption(ChannelOption.TCP_NODELAY, true)
        .childOption(ChannelOption.SO_LINGER, 0)    
        .childOption(ChannelOption.SO_KEEPALIVE, true)
        .childOption(ChannelOption.SO_SNDBUF, 1024 * 1024 * 2);
        Channel ch = bootstrap.bind(port).sync().channel();
        log.info("PMP Server at port:{}", port);
        //等待服务端关闭socket
        ch.closeFuture().sync();
    } catch (Exception e) {
        log.error("exception:{}", e.getMessage());
    } finally {
        BOSS_GROUP.shutdownGracefully();
        WORKER_GROUP.shutdownGracefully();
    }
}
```

​	Netty中的高低水位是通过**ChannelOption.WRITE_BUFFER_WATER_MARK**来设置的WRITE_BUFFER_WATER_MARK是一个WriteBufferWaterMark对象，它包含两个属性low和high，分别表示低水位和高水位；

推荐做法：**每次调用channl.write(msg)方法首先调用channel.isWritable()判断是否可写**；如下所示：

```java
/**
 * 回写Netty响应
 *
 * @param ctx     Netty Channel上下文
 * @param content Http Response内容
 * @param status  http响应状态
 */
private void writeResponse(ChannelHandlerContext ctx, String content, HttpResponseStatus status) {
    ByteBuf buf = copiedBuffer(content, CharsetUtil.UTF_8);
    FullHttpResponse httpResponse = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, status, buf);
    httpResponse.headers().set("Content-Length", buf.readableBytes());
    httpResponse.headers().set("Content-Type", "text/plain");
    httpResponse.headers().set("Connection", "keep-alive");
    
    //当前Channel是否可写
    if (ctx.channel().isWritable()) {
        ctx.channel().writeAndFlush(httpResponse);
    }
}
```

​	补充：Netty 是一个事件驱动的 I/O 框架，设计的核心是高效地处理异步 I/O 操作，它使用了多个工作线程（worker threads）来处理客户端请求和响应，虽然 Netty 在配置 WorkerGroup 线程池数量方面提供了一定的调整能力，对于高并发的环境，平衡出站和入站的"**平衡速率**"仍然是关键。过高的入站速率会导致 Netty 的 IO 线程饱和，从而引发性能瓶颈。而在这种情况下，单纯调整 Netty 本身的配置（如增加工作线程池或调整缓冲区大小等）只是部分解决方案，**更重要的是保证业务处理逻辑(BusinessHandler)的高效性,所以**存储与检索、关键业务节点的处理，都是影响系统性能的主要瓶颈。

### **添加引用计数器**

​	Netty使用**引用计数器**来管理对象（[ByteBuf](https://netty.io/4.1/api/io/netty/buffer/ByteBuf.html)）的生命周期，确保内存得到有效释放并防止内存泄漏。引用计数的核心思想是：一个对象被分配后，只有在没有引用时才会被回收，我们在大批量广告投放的时候，系统报出以下错误：

LEAK: ByteBuf.release() was not called before it's garbage-collected

​	这意味着 ByteBuf 在生命周期结束后未正确调用 release() 方法释放内存，导致 Netty 检测到内存泄漏，所以应显示调用引用计数器的release方法来将对象直接进行回收（当引用计数器为0的时候），Netty 提供的简化引用计数操作的工具类：[ReferenceCountUtil](https://netty.io/4.1/api/index.html?io/netty/util/ReferenceCountUtil.html) ， 主要用于管理和释放基于引用计数的对象（如 ByteBuf）。它在正常和异常场景下都可以有效地防止内存泄漏，如果处理完成后不需要将消息传递给下一个 ChannelHandler，需要手动释放它，故在所有业务流程结束后，调用释放方法，直接回收即可。

**ReferenceCountUtil使用Example：**

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {

    if (!(msg instanceof FullHttpRequest)) {
        return;
    }
    FullHttpRequest request = (FullHttpRequest) msg;
    String methodName = request.method().name();
    if (HttpMethod.POST.name().equals(methodName)) {
        postHandler(ctx, request);
    } else {
        writeResponse(ctx, "Request method 'GET' not supported", HttpResponseStatus.BAD_REQUEST);
    }

    ReferenceCountUtil.safeRelease(request);
}
```

​	Netty 使用了 **CAS** 操作来更新引用计数（自旋锁进行decrease操作）。这种方式在底层硬件支持下避免了传统锁的开销。同时确保了内存泄漏的问题。



### **动静分离：**

#### **为什么要动静分离？**

​	实时系统通常要求低延迟、较高的吞吐量，因此在资源分配上可能需要更多的 CPU、内存、网络带宽等，以处理实时流量和并发请求，而**离线系统一般用于处理批量数据**，动静分离后，可以根据不同的任务优化资源的分配，确保系统整体的高效运行。例如，实时系统使用高效的内存和缓存管理，而离线批处理系统可以使用更强大的计算集群来处理数据分析任务，将两者分开，使得每个系统专注于自己的任务，降低系统复杂度，不需要担心彼此对性能的干扰。实时系统的低延迟要求与批处理的复杂计算可以并行运行，不互相制约。

## **protobuf加解密消耗CPU时钟过高；**

### **流式解析**

​	对于大的Protobuf消息体，如果你不需要一次性解析整个消息，而是希望逐步处理，可以使用流式解析，这样Protobuf会**按需解析**数据，而不是将所有内容加载到内存中，这种方式在处理较大数据时特别有效，可以节省内存并提高效率。

### **优化ByteBuf和内存管理**

​	使用 Netty进行数据传输，可以考虑直接使用ByteBuf来包装Protobuf数据，避免额外的内存分配和复制。ByteBuf提供了更高效的内存管理方式，减少了内存分配和 GC 压力。

**改造后的代码：**

```java
ByteBuf byteBuf = Unpooled.wrappedBuffer(request.content());
byte[] bytes = ByteBufUtil.getBytes(byteBuf);
CodedInputStream input = CodedInputStream.newInstance(bytes);
Request Request = Request.parseFrom(input);
```

## **Netty海量长连接；**

### **设置长连接**

​	Netty 提供了 ChannelOption.SO_KEEPALIVE，用于开启 TCP 的 SO_KEEPALIVE 选项。

```java
bootstrap.childOption(ChannelOption.SO_KEEPALIVE, true);
```

### **心跳包设置**

#### 读写设置

```java
//IdleStateHandler 与客户端链接后，根据超出配置的时间自动触发userEventTriggered，以下参数分别是读超时、写超时、读写空闲
pipeline.addLast("idle", new IdleStateHandler(100, 100, 100, TimeUnit.SECONDS));
```

​	如果idle（如客户端由于网络原因导致到服务器的心跳无法送达），则服务器会主动断开连接，释放资源。判断连接是否idle是通过定时任务完成的，但是Netty可能维	持数百万级别的长连接，对每个连接去定义一个定时任务是不可行的，所以如何提升I/O超时调度的效率呢？Netty根据时间轮(Timing Wheel)开发了[HashedWheelTimer](https://netty.io/4.1/api/io/netty/util/HashedWheelTimer.html)工具类，用来优化I/O超时调度(本质上是延迟任务）；之所以采用时间轮(Timing Wheel)的结构还有一个很重要的原因是I/O超时这种类型的任务对时效性不需要非常精准。

```java
@Override
public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
    if (evt instanceof IdleStateEvent) {
        ctx.channel().close();
    } else {
        ctx.fireUserEventTriggered(evt);
    }
}
```

## **Http Response丢包：**

​	如果 HTTP 响应体（Response Body）过大，超过了单个 TCP 包的大小限制，Netty 会将响应体拆分成多个数据包进行发送。如果这些数据包没有正确地组装在客户端或服务器端，可能会出现响应丢包或无法完整接收的情况。排除掉IO阻塞问题，那么问题肯定出在服务器的tcp配置上面

cat /proc/sys/net/ipv4/tcp_mem

​	查看 Linux 系统中 **TCP 内存缓冲区，那么得出结果：首个是**TCP 内存缓冲区的最小值，中间的是TCP 内存缓冲区的默认值，最后一个是TCP 内存缓冲区的最大值。

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476763150-0c3f9723-1543-4d22-a5e6-344bb707c9e2.png)

### **调整HttpObjectAggregator大小:**

​	http服务器把HttpObjectAggregator放入pipeline里，HttpObjectAggregator会把多个消息转换为一个单一的FullHttpRequest或是FullHttpResponse，通过构造参数定义http传输聚合包的大小，根据评估，广告系统的大小调整为**(1024 \* 1024 \* 64)**，单位：字节；

### **调整**操作系统写缓冲区大小:

​	首先使用命令来看下，操作系统内核读写缓冲区大小：

```shell
#这是一个系统全局参数，其单位是页(1页等于4096字节)，表示所有TCP的buffer配置。有三个值：
#第一个值buffer值的下限，
#第二个值表示内存压力模式开始对buffer应于压力的上限；
#第三个值内存使用的上限，超过时，可能会丢弃报文
cat /proc/sys/net/ipv4/tcp_mem
```

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476813531-e9cbeee7-0e3b-4e85-80a4-224ddd7824df.png)

​	注意：当我们在编程中对连接设置了**SO_SNDBUF、SO_RCVBUF**，将会使Linux内核不再对这样的连接执行自动调整功能！！！Netty提供的[ChannelOption](https://netty.io/4.1/api/io/netty/channel/ChannelOption.html)提供了对底层网络参数的精细化控制，可以帮助开发者根据具体场景调优应用的网络性能，其中ChannelOption.SO_RCVBUF 和 ChannelOption.SO_SNDBUF可以直接设置 TCP 的接收和发送缓冲区大小，以映射操作系统底层tcp参数。更灵活的方式是可以动态调整缓冲区的大小，这时候就体现出 ByteBuf 的优势，Netty 提供的 ByteBuf 是可以支持动态调整容量的，而且提供了开箱即用的工具，例如可动态调整容量的接收缓冲区分配器 [AdaptiveRecvByteBufAllocator](https://netty.io/4.1/api/io/netty/channel/AdaptiveRecvByteBufAllocator.html)。

## **Hbase集群CPU使用率特别高；**

### **Get改成批处理：**

​	HBase分别提供了单条get以及批量get的API接口，使用批量get接口可以减少客户端到RegionServer之间的RPC连接数，提高读取性能。另外需要注意的是，**批量get请求要么成功返回所有请求数据，要么抛出异常。**

**以下是Hbase批量获取实例的demo：**

```java
function getColumnValues(tableName, rowKeys, columnNames):
    results = empty map  // Map<String, Map<String, Tuple<String, Long>>>

    try:
        family = DEFAULT_FAMILY
        columnBytes = []
        for columnName in columnNames:
            columnBytes.add(toBytes(columnName))

        tableName = checkNamespace(tableName)
        table = getTable(tableName)

        multiGet = []
        for rowKey in rowKeys:
            get = new Get(toBytes(rowKey))
            for columnByte in columnBytes:
                get.addColumn(toBytes(family), columnByte)
            multiGet.add(get)

        hTableResults = table.get(multiGet)

        if hTableResults is not null:
            for result in hTableResults:
                if result is not empty:
                    rowResult = empty map // Map<String, Tuple<String, Long>>
                    for cell in result.listCells():
                        value = bytesToString(cell.getValue())
                        key = bytesToString(cell.getQualifier())
                        ts = cell.getTimestamp()
                        rowResult[key] = (value, ts)

                    rowKeyStr = bytesToString(result.getRow())
                    results[rowKeyStr] = rowResult

    catch IOException e:
        log error with tableName, rowKeys, columnNames
        throw runtime exception "getColumnValues数据失败"

    finally:
        close(table)

    return results
```

##### **注意点：**

​	Hbase的批量操作可以提高效率，但一次请求太多的行可能会导致内存或网络负载过高。可以将批量Get请求分批执行，每次处理一定数量的行同时，Hbase是支持动态列的数据库，默认是查询所有列操作，故以上方法提供了List columnNames 来查询**有限的列**，因为在HBase中，**列数理论上是\****无限的**，但是在实际应用中，列数过多会对性能和存储产生负面影响以减少数据的传输量。减少滞后性。 

### **调整MemStore大小：**

​	[MemStore](https://hbase.apache.org/2.4/devapidocs/org/apache/hadoop/hbase/regionserver/MemStore.html)是存储数据的内存区域，负责缓存写入的数据，直到这些数据被刷写到磁盘上的HFile。当数据被写入HBase时，首先会被写入到 MemStore 中，然后在 MemStore达到一定大小后(**hbase.regionserver.memstore.flush.size，默认128**MB**)，数据会被刷写到磁盘上。调整 MemStore 的配置对于优化写入性能、内存使用和系统吞吐量非常重要。，根据实际负载进行调整。如果你的写入负载很高，可以考虑增大此值以减少刷写次数，减少磁盘I/O的压力。**hbase.regionserver.global.memstore.upperLimit 默认0.4**，设置 MemStore 占用内存的上限百分比。如果 MemStore 使用的内存超过此值，RegionServer 会触发** flush 操作，将 MemStore 中的数据刷写到磁盘；

### **异步写入：**

​	Hbase提供了[AsyncBufferedMutator](https://hbase.apache.org/2.0/devapidocs/org/apache/hadoop/hbase/client/AsyncBufferedMutator.html)异步写入的功能，它允许客户端在不阻塞的情况下执行Put、Delete等写操作。通过使用异步写入，应用程序可以显著提高写入吞吐量，尤其是在写入超高频次的业务场景下；同时Hbase提供了插入数据的批处理API，可以与异步操作结合使用。

### **调整BlockCache大小**

​	BlockCache读缓存，每一个Region Server只会有一个读缓存，把读到的数据都放到内存中的，另外：BlockCache的大小必须是内存页大小的整数倍(**目前常用页面大小4KB**)。默认值0.2（即20%的堆内存），增加此值可以提高读取性能，尤其是在读取热点数据时。通常，如果你的系统有大量的读取请求且数据访问存在热点（即某些数据频繁访问），可以增大BlockCache的大小。

高读负载系统：如果你的应用有较高的读取负载，尤其是热点数据（频繁访问的列或行），可以增大 BlockCache 的大小。通过**hfile.block.cache.size**。当然，如果频繁读取的数据，可以使用数据预加载在[MemCached](https://zh.wikipedia.org/zh-cn/Memcached)或者Redis中。

## **Clickhouse实时写入条数实在太多：**

### **存储告急：**

​	**ClickHouse将默认使用**[LZ4压缩模式](https://zh.wikipedia.org/wiki/LZ4)，**压缩比**也更多取决于数据列的排序，由于广告属于高度重复的数据，并且属于超高频次频繁的写入，使用ZSTD或LZ4等高效压缩算法，压缩比和速度都能达到一个较好的平衡。压缩比越高，存储空间的占用越小。同时，排序字段 (ORDER BY) 决定了数据在磁盘上的存储顺序。相似值存储在一起可以大幅提高压缩效率。将**高基数字段**（重复率较低的字段）放在排序字段的后面，**低基数字段优先排序**。

#### **调整排序(order by)**

```plsql
ORDER BY (
    server_time,         -- 按天
    advertiser_id,	     -- 首筛维度，必须靠前
    plan_id,             -- 广告主下的具体投放，次筛
    media_id,            -- 媒体渠道（高维聚合常用）
    ad_slot_id,          -- 广告位（媒体维度下钻）
    geo_id,              -- 地域维度（可视化筛选多）
    target_type_id,      -- 定向维度（用于聚合）
    event_time,          -- 时段分析、时间排序
    request_id           -- 唯一ID（做点查用，可建Bloom）
)
```





### **内存顶不住**

#### **优化jvm**

```powershell
JAVA_OPTS="-Xms8192m -Xmx8192m 
-XX:+UnlockExperimentalVMOptions 
-XX:+UseCGroupMemoryLimitForHeap 
-XX:MaxRAMFraction=2 
-XX:MetaspaceSize=512m 
-XX:MaxMetaspaceSize=8192m 
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/app/logs/dumpfile"
```

通过设置 JVM 的启动参数，主要配置了堆内存、元空间和其他内存管理的参数，例：

设置**Xms8192m**	 ：则JVM 初始堆内存为 8GB（8192 MB）。这意味着 JVM 在启动时会分配 8GB 的内存给堆(ps：广告系统的机器内存大小就是8GB)；

设置**Xmx8192m**	 ：则JVM 最大堆内存为 8GB。这意味着 JVM 允许的最大堆内存限制为 8GB；

**MaxRAMFraction** ：是 JVM 的一个内存管理选项，控制 JVM 最大堆内存的分配比例。默认情况下，JVM 可能会根据总内存大小来计算堆内存的大小。

**MetaspaceSize** ：Metaspace是用于存储类的元数据的区域，在JDK 8以后，JVM将类的元数据从永久代(PermGen)移到了 

Metaspace中，Metaspace默认是动态扩展的，但可以通过，-XX:MetaspaceSize 来控制初始分配大小，这里设置了Metaspace 的初始大小为512MB。

**MaxMetaspaceSize** : 这个参数指定 Metaspace 区域的最大大小为 8GB（8192MB）。当 JVM 加载类的数量和大小超出这个限制时，JVM 会抛出OutOfMemoryError 异常，表示类元数据存储区域已满。

总结：通过调整最大堆内存，优化内存的使用并提高稳定性，尤其是应对高负载和内存溢出等问题。

### **Kafka离线项目消费者速度跟不上生产者速度：**

​	由于广告系统上报时并发量太大，kafka生产者发送消息的速率过高，**消费者的消费能力跟不上**，会导致积压和延迟。生产者和消费者之间的速率差异可能会导致队列越来越大，最终可能导致消费者的内存溢出或系统崩溃，常见问题如下图所示：

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/12670791/1753720064746-fbb6edd5-0281-46e2-b951-adc248712b7b.jpeg)

#### **调整Kafka fetch最⼤拉取记录数和最大字节数：**

​	在 Kafka 中，调整消费者（Consumer）拉取每个分区的最大记录数，主要是通过 max.poll.records（表示每次拉取时最多能拉取多少条记录，**默认是500**） 参数进行配置，因为广告埋点应用场景需要处理更大的批量数据，可以调整这个值，同时根据每次拉取的最大字节数(**fetch.max.bytes**)。如果消息体比较大，可能会设置一个较小的 max.poll.records 来限制每次获取的记录数量，如下图所示，广告系统层面，根据Kafka本身配置(8C16G，1T磁盘)，将每次拉取次数改为**1000条**，拉取最大字节数为1048576(说白了，就是1MB)，使用者可根据自己本身数据结构的差异，数据量级的大小，数据写入频次的高低调整这两个参数。

```yaml
#fetch时每个分区最大拉取的字节数
partitionMaxFetch: 1048576
#fetch时每个分区最大拉取记录数
pollRecords: 1000
```

#### **调整*****离线项目***线程池大小：

```java
private static final ThreadPoolExecutor TASK_EXECUTOR = new ThreadPoolExecutor(
    // 核心线程数
    50,
    // 最大线程数
    100,
    // 多长时间把非核心空闲线程交还操作系统
    1,
    TimeUnit.MINUTES,
    // 阻塞队列，队列长度
    new LinkedBlockingQueue<>(10000),
    // 设置线程名
    namedThreadFactory,
    // 拒绝策略，当提交任务数超过最大线程数+队列之和时触发
    new ThreadPoolExecutor.CallerRunsPolicy()
);

TASK_EXECUTOR.submit(() -> {
    try {
        handleEventList(batch);
    } catch (Exception e) {
        log.error("【离线】事件处理失败, 列表大小：{}   events:{}", batch.size(), JSONUtil.obj2Json(e));
    }
});
```

​	如上代码所示，构建了一个自定义的线程池，**corePoolSize 和 maximumPoolSize 的设置需要根据系统的实际需求来调整**。过多的线程可能导致上下文切换过于频繁，过少的线程可能导致任务处理延迟。使用 LinkedBlockingQueue 存放待处理的任务，队列的最大长度为 10000。如果线程池中的线程都在工作，任务会先进入这个队列，等待线程空闲后再执行，当队列已满，且线程数已达到 maximumPoolSize，新的任务将按照指定的拒绝策略处理采用 CallerRunsPolicy 拒绝策略，表示当线程池已满时，将由调用者线程执行该任务，而不是丢弃任务(为了保证数据的完整性)。当线程池中的线程超过corePoolSize，并且这些线程在 keepAliveTime 指定的时间内没有被重新使用，它们将被销毁。即，非核心线程会在空闲 1 分钟后被回收。线程池线程池执行任务时，通过submit方法，将任务被提交到线程池进行异步处理。

### **调整ClickHouse批量插入的队列：**

### **Kafka消息碎片化问题：**

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/12670791/1753720009368-52a68889-d277-4e90-a33d-29e7d1089782.jpeg)

#### **升级Kafka client版本**

```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>2.2.0</version>
</dependency>
```

​	在Kafka Clients 2.2.0中，生产者的批量处理机制得到了优化。生产者将多个小消息打包成更大的批次进行发送，这有助于减少由于发送多个小消息而造成的碎片化问题。

#### **配置参数修改：**

```yaml
batchSize: 32768
linger: 100
compression: gzip
```

##### 批次大小配置

​	可以通过配置**batch.size**来设置生产者每次批量发送的消息大小。通过合理增加batchSize，可以将多个小消息合并成一个更大的批次，避免过多的小碎片。

##### 生产者等待批量数据的时间

​	当设置linger.ms，来增加消息批量化的机会(间隔)，进一步减少碎片化。关于batchSize和Linger的参考内容：https://learn.conduktor.io/kafka/kafka-producer-batching/

##### 压缩类型配置

​	生产者可以通过compression来设置压缩类型，支持gzip、snappy、lz4等。**压缩速度snappy适当的压缩不仅可以减少磁盘空间占用，还能在网络传输时减少碎片。**

## **离线项目重启会丢数据：**

​	在Kubernetes (K8s) 环境中，[@PreDestroy](https://www.cnblogs.com/erlou96/p/13753824.html) 可以与Kubernetes 提供的 **生命周期钩子**（Lifecycle Hooks）一起使用，以确保在容器被关闭或重新启动时执行一些清理工作。这些钩子通常用于处理容器的退出操作，比如优雅关闭（Graceful Shutdown）、停止服务、释放资源等。

Kubernetes 提供了以下两种主要的生命周期钩子：

1. preStop **Hook**：容器停止前执行的钩子。
2. postStart **Hook**：容器启动后的钩子。

### **@PreDestroy 与 Kubernetes preStop Hook 的结合**

​	@PreDestroy 是 Spring 的 Bean 生命周期回调方法，而 Kubernetes 的 preStop 钩子是 Kubernetes 为容器生命周期提供的钩子。这两个机制是独立的，但可以协同工作，实现更细粒度的生命周期控制。

- PreDestroy：确保在 Spring Bean 销毁时执行清理工作，如释放资源、停止线程等。
- preStop：确保在容器被终止之前执行清理工作，如停止后台进程、关闭数据库连接等。

在 Kubernetes 中，当容器被终止时，preStop 钩子会被触发，Kubernetes 会等待钩子完成后再终止容器。这时，可以将 Spring 的 @PreDestroy 与 Kubernetes 的 preStop 配合使用，确保容器终止时有充足的时间进行清理工作。

### **1.****如何结合使用 @PreDestroy 和 Kubernetes preStop Hook**

#### **步骤：**

- 在 Spring 应用中使用 @PreDestroy 注解进行资源清理。
- 配置 Kubernetes 的 preStop 钩子，调用一个脚本或自定义的清理命令，确保容器停止前执行额外的操作。
- Kubernetes 会在收到终止信号时，首先调用 preStop 钩子，然后**等待钩子完成后才终止容器**。

​	在广告系统中，由于离线项目可能有重启的必要，并且实时消费数据量极大，如果处理不当，则会造成大量数据丢失，故在每个具体的消费者类中添加一个cleanup方法，上面标注PreDestroy注解，同时定义一个**AtomicBoolean(flag)**变量，一旦触发cleanup方法，则将flag置为false，同时将剩余资源的队列(BlockingQueue对象)一次性一次性取出(drainTo方法)，放入具体的消费方法中，同时发送钉钉通知，告知用户已经优雅关闭完成。

```java
@PreDestroy
public void cleanup() {
    log.info("【离线】优雅停机开始");
    // 暂停消费
    flag.set(false);
    // 提交剩余队列
    List<JSONObject> batch = new ArrayList<>();
    queue.drainTo(batch); // 至少尝试取出一个元素，或者直到队列空
    if (!batch.isEmpty()) {
        try {
            handleEventList(batch);
        } catch (Exception e) {
            log.error("【离线】事件处理失败, events:{}", JSONObject.toJSONString(batch), e);
        }
    }

    // 等待5秒
    dingTalkMsgSend.msgSend("https://oapi.dingtalk.com/robot/send?access_token=a6ddf9e618f5654836f2acac40bd74f0892b4072442769fc9a4ce14add4c21f0", "优雅停机 已完成", null, false);
    log.info("【离线】优雅停机结束");
}
```



## **跨地域机房网络请求消耗大：**

​	跨地域机房网络请求消耗大通常是由于网络延迟、带宽限制和数据传输的跨地理位置引起的。这种情况常见于不同地区的数据中心之间进行频繁的网络请求时，通过公共互联网传输，可能会面临较高的延迟，尤其是涉及到长距离传输时。

因广告平台请求数据被动对接方，且媒体对于延迟极高，虽然可以采用gzip压缩、Cassandra或TIDB等支持跨地域数据复制的分布式数据库，根据实际需求增加带宽，或者通过流量控制机制（如流量限速、负载均衡），但均不能解决延迟较高问题，具调研，媒体方广告机房在华南、华北、华东均有机房，其中，华东与华南请求量最大，我方服务器部署在华北北京机房，从华南请求跨地域打来就有20—60ms延迟，同时还要处理内部业务逻辑，故将机房分开部署至华东杭州、华南河源，分开部署后，超时率下降至0.1%以下。



## **Netty worker Group线程组不够，需要调整：**

​	Netty线程模型采用“服务端监听线程”和“IO线程”分离的方式，与多线程Reactor模型类似。BossGroup负责事件的connect、accept，负责接收和连接服务，负责创建NioServerSocketChannel，并将服务转到其他的EventLoop上面；Worker Group负责事件的read，将client的请求进行解码、处理，并且处理后续事件(自定义业务handler)，并且处理队列中的任务。如果我们在实例化 NioEventLoopGroup 时, 如果指定线程池大小, 则nThreads就是指定的值, 反之是**处理器核心数** ***** **2** （Runtime.getRuntime().availableProcessors() * 2）， 如果你知道你的应用会有很多并发连接并且每个连接会处理大量数据，你可能需要为 workerGroup 分配更多的线程，以避免瓶颈，从而提高处理能力；

具体demo：广告系统使用500个线程数来处理业务线程代码。

```java
/**
 * 绑定端口号
 */
private int port;
/**
 * bossGroup
 */
protected static final int BIZ_BOSS_THREAD_SIZE = Runtime.getRuntime().availableProcessors() * 2;
/**
 * workerGroup
 */
protected static final int BIZ_WORKER_THREAD_SIZE = 500;

private static final EventLoopGroup BOSS_GROUP = new NioEventLoopGroup(BIZ_BOSS_THREAD_SIZE);
private static final EventLoopGroup WORKER_GROUP = new NioEventLoopGroup(BIZ_WORKER_THREAD_SIZE);
```



# **结果**

## **单机能支撑的QPS**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739477141126-6c1025f7-620d-417d-8beb-671d3dee1f87.png)

## **内部业务处理时长采样(单位：ms)**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739477132993-84555e8f-41f0-4cbb-95bf-dfd34c2b6928.png)

## **采样数据:**

以下内容为连续压测五分钟的平均响应时长数据曲线图

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739477094498-dcfc84ef-9867-4fb9-9052-d7897c9c68f6.png)

## **日常服务器参数指标(已稳定)；**

### CPU&内存&网络传输

​	ps：以下服务器属于测试服务器

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739477079793-63f2bc46-3257-4ec6-8fbc-17ac15621752.png)

### **打开文件数**

#### **为什么要专注打开文件数这个指标？**

##### **1.****支持高并发连接**

- **每个网络连接都会占用一个文件描述符，默认限制可能会导致高并发服务器（对应广告系统的话就是Netty）无法处理超过限制数量的连接。**
- **增加文件描述符限制，允许服务支持更多并发连接，避免 "Too many open files" 错误。**

##### 2. 提升系统稳定性

- **文件描述符不足时，应用可能崩溃或拒绝新连接（服务端返回504 Gateway Timeout）。**
- **增大限制可以避免因描述符耗尽导致的服务不可用。**

##### 3. 支持资源密集型任务

- **大规模文件读写（广告系统日志处理会将请求内容打印，每秒写入文件大小可以达到GB级别）会打开许多文件，同时占用大量文件描述符。**
- **修改限制后，应用程序可以安全地处理更多文件操作。**

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/12670791/1753720207890-840c1fd4-2a35-4ce8-9636-4dd876a4d500.jpeg)

​	增加文件描述符数量有助于提高并发能力，可以确保应用程序在处理大量并发连接时不会遇到文件描述符耗尽的问题，同时打开大量文件描述符，可能会增加内核的调度开销，导致性能问题。在调高文件描述符限制时，应该进行负载测试，确保没有过度使用系统资源(尤其是内存)，确保没有过度使用系统资源。

#### **调整**服务器Linux打开文件数大小

查看Linux打开文件数大小：

```plain
# 查看当前进程的最大文件描述符数
ulimit -n
```

### 	

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476988039-ac5f271f-c0c7-40dc-b214-c554ae714445.png)

## **clickhouse每秒峰值写入条数：**

​	如下图峰值时，鼠标点位移动的的位置的每秒写入速度为242万每秒，最高时大概为270万每秒写入。

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1739476980900-5cf49db1-487e-47c8-93e7-0b05ca3ecb59.png)

## **clickhouse日常写入：**

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/12670791/1753720106754-ddab1a8d-114a-4eb4-aca8-fbe5a6f32f7f.jpeg)





# **总结：**

​	文中提到的技术以及拆分方案，很多技术点，都可以提升吞吐量及性能指标，列举如下：

#### **IO多路复用**

​	管理更多的连接；

#### **线程池技术**

​	挖掘多核cpu的潜力；

#### **zero-copy**

​	减少用户态和内核态交互次数。如Linux中sendfile系统接口；

#### **磁盘顺序写**

​	降低寻址开销。消息队列或数据库日志，都会采用此技术；

#### **更好的协议**

​	网络传输上减少开支，如：自定义(物联网中使用较多)或二进制传输协议；

#### **分区**

​	在存储系统中，分库分表都算分区；而微服务中，设计服务无状态，本身也可以理解为分区；

#### **批量传输**

​	典型数据库batch技术。很多中间件也可以使用，如消息队列、NoSql中；

#### **索引技术**

​	这里不是特指数据库的索引技术。而是我们设计切合业务场景的索引，提供使用率。例：kafka文件的存储，采用hack的索引技巧、clickhouse的索引，降低磁盘使用率等；

#### **空间换时间**

​	其实分区、索引技术、缓存技术都可归为这类。例如：我们使用倒排索引存储数据、使用多份数据多份节点提供服务等；

#### **网络连接的选型**

​	长连接、短连接，可靠、非可靠协议等；

#### **拆包粘包**

​	batch、协议选型于此有些关系；

#### **高性能分布式锁**

​	并发编程中，锁不可避免。尽量使用高性能的分布式锁，能cas乐观锁(注意锁膨胀)，尽量避免悲观锁；如果业务允许，尽量异步锁，不要同步阻塞锁，影响主线程执行效率，同时要减少锁竞争；

#### **柔性事务代替刚性事务**

​	有些异常或者故障，试图通过重试是恢复不了的；

#### **最终一致性**

​	如果业务场景允许，尽量保证数据最终一致性；

#### **非核心业务异步化**

​	把某些任务转化为另外一个队列(消息队列)，消费端可以批量、多消费者处理；

#### **direct IO**

​	例如数据库等构建缓存机制的应用程序，直接使用directIO，放弃操作系统提供的缓存；



注：脱离业务场景，很多只能是纸上谈兵。但不了解手段，遇到场景也会懵逼。客户端请求形成的超级队列，后端如何分而治之、分散逐个击破，才是整体思想。在此过程中，我也沉淀了些许的经验和知识，这些宝贵的经验将在今后其他项目研发和迭代的过程中发挥重要的作用。系统架构设计和性能优化是一项持续的任务，随着业务的发展和技术的进步，我也将不断探索新的优化手段。它需要我们不断地发现问题，寻找解决方案，反复试验和优化。同时，性能优化也不是孤立的，它需要我们与业务、产品、开发等多个团队紧密合作，共同推动项目的进展。受限于篇幅及笔者的个人能力有限，本文仅通过若干实例分享了在高并发系统架构和服务性能优化方面取得的一些微小成果。如读者发现文章内有错误也请及时与我联系，我会虚心接受笔误或能力不足造成的错误，性能优化的重要性不仅仅体现在提高系统性能，降低资源使用上，更重要的是，它可以提升用户体验，支持业务的快速发展，实现公司的降本增效目标。笔者希望这篇文章能够对读者有所启发，无论是在处理类似的性能问题，还是在面临其他的技术挑战，都能从我们的经验中找到一些启示和灵感。







------

## **联系方式 & 更多分享**

如果您对本文档中的技术细节、架构设计有任何疑问或希望进行技术交流，欢迎通过以下方式联系我：

- **微信：**

![img](https://cdn.nlark.com/yuque/0/2025/png/12670791/1753878753944-ecd8dd0a-3b8b-4ca0-939d-e2f0682e63bb.png)

- **邮箱：** 	duwuqiang.qq@gmail.com
- **个人博客：** https://juejin.cn/user/1292681405277277
- **GitHub：** https://github.com/1993Kevin

期待与您共同探讨技术！
