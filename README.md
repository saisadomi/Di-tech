# Di-Tech算法大赛
## 赛题详情
在出行问题上，中国市场人数多、人口密度大，总体的出行频率远高于其他国家，这种情况在大城市尤为明显。然而，截至目前中国拥有汽车的人口只有不到10%，这意味着在中国人们的出行更加依赖于出租车、公共交通等市场提供的服务。另一方面，滴滴出行占领了国内绝大部分的网络呼叫出行市场，面对着巨大的数据量以及与日俱增的数据处理需求。截至目前，滴滴出行平台日均需处理1100万订单，需要分析的数据量达到50TB，路径规划服务请求超过90亿。面对如此庞杂的数据，我们需要通过不断升级、完善与创新背后的云计算与大数据技术，从而保证数据分析及相关应用的稳定，实现高频出行下的运力均衡。供需预测就是其中的一个关键问题。
供需预测的目标是准确预测出给定地理区域在未来某个时间段的出行需求量及需求满足量。调研发现，同一地区不同时间段的订单密度是不一样的，例如大型居住区在早高峰时段的出行需求比较旺盛，而商务区则在晚高峰时段的出行需求比较旺盛。如果能预测到在未来的一段时间内某些地区的出行需求量比较大，就可以提前对营运车辆提供一些引导，指向性地提高部分地区的运力，从而提升乘客的整体出行体验。

## 定义及评估标准
### 1. 问题定义
乘客打开滴滴出行app，输入出发地和目的地并点击“呼叫”后就完成一次发单(request)，有司机接单后就完成一次应答(answer)。

将一个城市划分为n个互不重叠的正方形区域$D={d_1,d_2,...,d_n}$，将每一天的24小时划分为144个10分钟长的时间片t1,t2,⋯,t144。

对于区域$d_i$，在时间片$t_j$，有$r_{ij}$个乘客发单，有$a_{ij}$个司机成功应答了$a_{ij}$次发单。

对于区域$d_i$，在时间片$t_j$，定义需求$demand_{ij}$=$r_{ij}$，供给$supply_{ij}$=$a_{ij}$，则有供需缺口$gap_{ij}$：$gap_{ij} = r_{ij} - a_{ij}$

给定每个区域在时间片$t_j$,$t_{j-1}$...的各项数据，预测$gap_{i,j+1}$, ∀$d_i$∈D。

### 2. 评价指标

对n个区域和q个时间片，区域$d_i$在时间片$t_j$的供需缺口为$gap_{ij}$，选手预测值为$s_{ij}$，
以MAPE作为最终的评价指标：
$$MAPE=\frac{1}{n}\sum_{d_i}{( \frac{1}{q}\sum_{t_j}{|\dfrac{gap_{ij}-s_{ij}}{gap_{ij}}} |)}$$
MAPE越小越好。

### 3. 选手提交结果
选手提交的数据格式为：区域ID，时间片，预测值。其中每个字段的具体描述如下：

|数据名称|数据类型|示例|
|:---:|:---:|:---:|
|区域ID| string | 1,2,3,4 （与区域映射ID一致）|
|时间片| string |2016-01-23-1（即2016年1月23日第1个时间片，时间片是将每天的时间按10分钟间隔划分到1-144个片中）|
|预测值| double | 6.0 |

## 数据形式
训练集中给出了M市2016年连续三周的数据信息，需预测M市第四周和第五周中某五天的某些事件段供需。测试集中给出了每个需预测的时间片的前半小时的数据信息，具体需预测的时间片见说明文件（说明文件含在数据集下载包内）。具体数据如下，其中订单信息表，天气信息表和POI信息表为数据库中的直接表信息；而区域定义表，拥堵信息表是由数据库中的其他表衍生的信息。

##### 订单信息表
订单信息表主要覆盖了一张订单的基本信息，包括这张订单的乘客，以及接单的司机（driver_id =NULL表示driver_id为空，即这个订单没有司机应答），及出发地，目的地，价格和时间。

|字段| 类型 | 含义 | 示例 |
|:---:|:---:|:---:|:---:|
| order_id | string | 订单ID | 70fc7c2bd2caf386bb50f8fd5dfef0cf |
| driver_id | string | 司机ID	| 56018323b921dd2c5444f98fb45509de |
| passenger_id	| string	| 用户ID	| 238de35f44bbe8a67bdea86a5b0f4719 |
| start_district_hash | string | 出发地区域哈希值 | d4ec2125aff74eded207d2d915ef682f |
| dest_district_hash | string | 目的地区域哈希值 | 929ec6c160e6f52c20a4217c7978f681 |
| Price | double | 价格 | 37.5 |
|Time | string | 订单时间戳 | 2016-01-15 00:35:11 |

##### 区域定义表

区域定义表主要表示比赛评测区域的信息，选手需选择区域定义表中的区域来做预测，并在最终提交的结果中需将区域哈希值映射为其相应的ID。

|字段| 类型 | 含义 | 示例 |
|:---:|:---:|:---:|:---:|
| district_hash | string | 区域哈希值 | 90c5a34f06ac86aee0fd70e2adce7d8a |
| district_id | string | 区域映射ID | 1 |

##### POI信息表

POI信息表主要表征区域的地域属性，由其中所含的不同类别设施的数量表示，如2#1:22表示在此区域中含有类别为2#1的设施22个，2#1表示一级类别为2，二级类别为1，例如休闲娱乐#剧院，购物#家电数码，运动健身#其他等等。不同类别及其数量以\t分割。

|字段| 类型 | 含义 | 示例 |
|:---:|:---:|:---:|:---:|
| district_hash | string | 区域哈希值 | 74c1c25f4b283fa74a5514307b0d0278 |
| poi_class | string | POI类目及其数量 | 1#1:41 2#1:22 2#2:32 |


##### 拥堵信息表
拥堵信息表主要表示区域中道路的总体拥堵情况，其中主要包括不同时间段不同区域的不同拥堵情况的路段数，其中的拥堵级别是越大越拥堵。

|字段| 类型 | 含义 | 示例 |
|:---:|:---:|:---:|:---:|
| district_hash | string | 区域哈希值 | 1ecbb52d73c522f184a6fc53128b1ea1 |
| tj_level | string | 不同拥堵程度的路段数 | 1:231 2:33 3:13 4:10 |
| tj_time | string | 时间戳 | 2016-01-15 00:35:11 |


##### 天气信息表
天气信息表主要表示整个城市的每天间隔10分钟段的天气情况。其中的weather字段表示天气的实时描述信息，而温度以摄氏温度表示，PM2.5为实时空气污染指数。

|字段| 类型 | 含义 | 示例 |
|:---:|:---:|:---:|:---:|
| Time | string | 时间戳 | 2016-01-15 00:35:11 |
| Weather | int | 天气 | 7 |
| temperature | double | 温度 | -9 |
| PM2.5 | double | pm25 | 66 |
