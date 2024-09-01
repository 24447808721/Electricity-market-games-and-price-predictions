# Electricity-market-games-and-price-predictions
对于电力市场中主体博弈产生的结算价格的预测具有重要的理论和现实意义。电力部门推动能源转型促进可持续发展的主体。因此，对电力市场主体行为的分析以及最终的市场结算价格的预测能够促进多学科融合，推动电力部门以及电力市场的产业转型。此外，可靠的电力系统保障社会稳定与安全，提升应急响应能力，并增强国家的全球竞争力和未来应对挑战的能力。
## 背景
该项目主要针对“电力现货市场”（可以类比证券交易市场），泛指短时间内的电能量交易市场。
市场中有大量发电机组（供给者，会发电并卖出电力）按照交易规则，在指定的平台上采取集中竞价的方式确定电能的交易量和价格。这些电能会被输送给个体户、商业用户等，从而利用市场机制实现资源优化配置。
在最理想的情况下，市场完全竞争（市场参与者众多，以至于没有任何一方能够影响价格，不存在控制价格的可能性），没有任何博弈行为。每个发电机组都诚实报价，市场出清价格稳定可靠，达到最优效率。
但现实中，电力现货市场有以下特点：
- 寡头竞争（几家电力公司独大，对价格有显著影响）
- 不完全信息（不同机组信息不互通，存在打信息差牟利的可能）
- 非合作博弈（机组之间各谋其利，追求各自的利益最大化）
- 参与者有限理性（受限于经济知识和对市场的了解，机组不一定能做出最优决策）
从而不同机组之间有复杂的博弈行为，这就让市场出清价格难以估计。
因此，要求针对电力现货市场价格和市场博弈主体（549个发电机组）的信息，用ABM方法建模这些机组在报价上的博弈行为，使最终模拟的市场出清报价接近现实中的市场出清价格。
## 数据描述
electricity price.csv：电力市场的市场出清价格，市场需求等信息。训练集范围为2021年12月1日到2023年7月1日，共计55392个点；测试集范围为2023年7月1日到2024年4月18日，共计28228个点
1. Day/Time：交易时间，中国电力现货市场15分钟结算一次，一天共96个交易点
2. demand：区域内电力总负荷（总需求），单位为MW
3. clearing price (CNY/MWh)：市场出清电价，单位为元/MW·h

| day	time | demand | clearing price (CNY/MWh) |
| :-----| ----: | :----: |
| 2021/12/1 0:15 | 40334.18 | 350.8 |
| 2021/12/1	0:30 | 40523.15 | 350.8 |
| 2021/12/1	0:45 | 40374.74 | 350.8 |	 	

unit.csv：存放市场供给者（各发电机组）的参数信息
机组数据包含549个不同的火电机组
1. unit ID：每个机组唯一的ID
2. Capacity（MW）：机组的额定容量（额定功率），越高机组的发电能力越强
3. utilization hour (h) ：电厂的年平均运行小时数，需要注意多个机组可能共同属于一个电厂，有相同的值
4. coal consumption (g coal/KWh)：每发一度电需要耗费多少煤炭，为成本参数
5. power consumption rate：电厂单位时间内耗电量与发电量的百分比，例如单位时间耗电量为500度电，发电量为10000度电，利用率就是500/10000=5%。

|  unit ID | Capacity (MW) | Utilization Hour (h) | Coal Consumption (g coal/KWh) |  Power Consumption Rate (%) |
|  ----  | ----  | ----  | ----  | ----  |
| 1 | 110 | 2069.12 | 266.07 | 6.91 |
| 2 | 160 | 5509.22 | 292.7 | 6.91 |
| 3 | 160 | 3562.79 | 293.35 | 6.91 |

## 项目任务
项目本质是一个回归（预测目标为连续值，例如根据年龄预测身高）问题，需要预测2023年7月1日到2024年4月18日每15分钟的市场出清价格，虽然可以纯粹依靠时间序列模型完成，但比赛方也提到期待选手使用ABM模型建模获取市场出清价格，因此可以在两方面进行尝试上分（ps：如果感觉ABM较难，可以专注于时间序列挖掘哦）。
最终评价指标为MSE（均方误差）和RMSE（均方根误差）的均值（事实上这和只用RMSE或者MSE是一样的），值越小越好，公式如下：
$$\text{MSE}=\frac{\sum_{i=1}^{n} (y_i - \hat{y}_i)^2}{n}
$$
$$\text{Final Metric}=\frac{\text{MSE}+\sqrt{\text{MSE}}}{2
}$$
其中：
- $$y_i
$$为真实市场出清价格
- $$\hat y_i$$为预测的市场出清价格
- $$n$$为样本数量，这里是测试集的大小28228

## 电力市场出清价格
### 电力市场概述
2000以前，国内并不存在电力市场，而是叫计划电力经济。发电侧为卖方，核算发电成本和利润上报国家，审核通过后就是上网电价。用户侧为买方，被动执行国家制定的分时电价。计划电力经济的优势为：电价相对稳定，企业用电成本核算相对简单；但是问题也比较突出，特别是煤价疏导滞后，体制机制僵化，资源配置粗放，不能灵敏准确的反映发电成本、发现电力价格。
2002年，电力市场化改革文件《国务院关于印发电力体制改革方案的通知》指出：打破垄断，引入竞争，提高效率，降低成本，健全电价机制，优化资源配置，促进电力发展，推进全国联网，构建政府监管下的政企分开、公平竞争、开放有序、健康发展的电力市场体系。通知发布后，原国家电力公司拆分为5大发电集团（家能源投资集团、中国华能集团、中国华电集团、中国大唐集团、国家电力投资集团）与2大电网（国家电网、南网）。发电厂试行竞价上网，成立国家电监会，对市场行为进行监管，从一定程度上打破了垄断。但这一阶段改革成效不彻底，其主要原因在于销售侧电价没有放开，所谓“放开两头，管住中间”只是在发电这一头产生了一定的成效，发电成本的变化并没有得到及时有效的传导。
2015年中发9号文《关于进一步深化电力体制的若干意见》指出：让发电企业和用户（公共事业、居民和农业用户仍执行政府定价）进入市场，通过报量报价进行交易撮合和价格出清，形成了真正的电力市场，基本达到了发现价格、优化配置的目标。
### 电力现货市场的价格出清机制
市场价格出清是通过交易系统完成的，即买方和卖方均通过交易系统提交买（卖）数量和价格的申请，然后通过交易系统进行匹配，最后形成一个价格。价格一旦形成，将被所有成员接受。最后形成的价格被称为边际出清价格。市场出清电价是指在竞争定价的电力市场中，能够实现市场供-需平衡的度电价格。
电力市场的出清价格形成类似证券市场早上9:15-9:25的集合竞价，由于数据里只提供了总需求，我们可以认为需求曲线是一条直线。
出清价格的形成步骤如下：
1. 所有发电机组申报自己卖出的电价和电量
2. 市场根据机组报价，从低到高排序，依次从低价开始成交
3. 当成交的容量和大于等于总需求时，达到市场出清（供需平衡），这时候最后一个达成交易的机组报价为市场出清价格
我们用一个实际案例来解释，假如市场总需求为3000MW，四个机组依次报价报量如下：
- 机组1：报价150元，容量500MW
- 机组2：报价200元，容量500MW
- 机组3：报价250元，容量2000MW
- 机组4：报价400元，容量2500MW
按价格从低到高，机组1先成交，随后是机组2、机组3，此时容量和等于总需求，不再有额外的电力需求，市场达到出清状态，市场出清价格为250元。
这里我们还可以发现一些现象：
- 机组4由于过高的价格竞标失败，导致没有卖出任何电力。现实中机组4不得不停掉一些发电机。但煤电机组频繁关停会影响设备寿命和性能，还可能延误后续高价时段的正常发电，最终只能亏本卖出。
- 机组1实际上能卖更多钱，如果他能预料到机组2和3报价为200和250，他就可以报价300元，保证最后一个出清的是他，而不是只能卖150元（但它无法报特别高的价，因为电力是公共物品，受政府管控，设置了价格上下限）
因此报价本身就是一个充满策略博弈的行为，就像在股市中总想做买入价格最低，卖出价格最高的那个人。
![image](https://github.com/user-attachments/assets/30329453-bf4a-4e44-88ad-496221354fda)

# PS
首先感谢dataWhale提供这次难得学习机会，还有感谢上海科学智能研究院提供的这次比赛，很遗憾因为个人原因没有提交最后的结果导致错失了决赛，但是也从这次比赛中学到了很多，在此对两个团队人员表示真挚的感谢。这篇内容是对这次学习的一个总结，利用ABM策略加时序预测来进行了一个优化，ABM策略中加入了光伏发电机组方便模拟外部环境对于电力市场影响，时间步设置为了一天（24*4=96），时序预测了主要用到LGB和线性回归两个模型，通过调整比例发现线性回归模型预测内容可能更接近实际效果，故而设置了3：7的比例。同时通过数据可视化看到市场符合鸭子曲线，所以通过简单线性设计来模拟鸭子曲线，除此之外还利用特征工程进行了数据完善。照例提一下优化思路：1.ABM策略可以考虑较长的时间步或考虑博弈出价进行设计。2.考虑更多的特征工程内容比如外部天气，煤的价格等 3.采用非线性方式进行模拟鸭子曲线。
