## Intro
### 案例：共享单车

从共享单车市场分析出发，理解为什么其能拿到大量融资
1. 你准备提供的产品（服务），解决什么需求？目标群体？刚需 or 改善型？ 高频需求，还是低频？
	1. 解决出行需求，刚需，高频。
2. 这个需求的市场是否足够大？如果是个小市场，那就不是创业，是糊口
3. 你的模式动了谁的奶酪？有多大的阻力？面临怎么样的竞争？
	1. 网约车？面临和国有 **公交车、地铁** 竞争
	2. 单车外部没有竞争对手。
4. 你的模式，是否容易被复制、被抄袭？
	1. **共享单车**非常容易被复制和抄袭
5. 盈利模式——**最重要的**：你怎么赚到钱？
	- **共享单车**实际是靠金融来挣钱的。
	- 广泛募款。——资金是业务的本质


### 公司理财的一般框架
- **国家第一个重要的经济问题**
> 充分就业


- **经济是怎么发展的**？
	- 中国货币狂飙突进
	- 1990: 1.53 万亿 2000: 13.46 万亿 2020: 218.68 万亿
	- 现在 (2025-8) 的 M2 货币供应量是 300 万亿
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250829150047.png)
	- COVID-19 期间放出了 100 万亿的货币
	- **货币**对于**经济**的拉动作用可能存在时空差异


### some infomation

- **重要内容**
	- **财务基础工具**
	- **投资决策**
	- **融资决策**
	- **其他**

- **score**
	- 10% 考勤
	- 30% 期中（实训，cos 公司）
	- 60% 考试
		- 单选
		- 多选
		- 判断题
		- 大题


## lecture 2

- **技术分析**
	- **常用工具**：K 线图、MA（移动平均线）
	- **K 线**
		- 开盘价，收盘价，最高价，最低价
		- **画在一个图上**。![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250905140632.png)
		- **十字星**
			- 特殊的 K 线形态
			- 当一只股票的**开盘价和收盘价几乎相等时**，它的K线实体部分会非常小，形成一条横线，而最高价和最低价则形成上下影线。这种形状看起来就像一个 **“十”字** ，因此得名。
		- **MA**（移动平均线）
			- 短期、中期、长期
		- 
	- **我们经常研究历史市场数据来预测未来价格走势**
		- 核心思想：市场行为，包容消化一切

	- **研究角度**
		- **宏观**
			- 整个市场的指数
		- **行业**
		- **基本面**
			- 选一些更有代表性、成长性的股票
		- **技术分析**


### Section 2
金融理财基本技能-货币时间价值和应用


- **年金**
	- 在相同的间隔时间内陆续收到或付出的**相同金额**的款项
- **年金的分类**
	- 普通年金
		- 后付。期末收入/付出
	- 即付年金
		- 期初
	- 永续年金
		- 无限期内，时间间隔相同，金额相等
- **计算**
	- 复利

- **普通年金终值的计算**


- FV
	- Future Value
	- 终值（流入流出）
- PV
	- Present Value
	- 现值
- RATE
	- Returns the interest rate per period of an annuity.
- NPER
	- Returns the **number of periods** for an investment based on periodic, constant payments and a constant interest rate.
- PMT
	- **Payment Per Period**
	- Calculates the payment for a loan based on constant payments and a constant interest rate.


- SOME other types
	- **PPMT** 
		- Payment on the Principal
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912140853.png) 
	- 基于固定利率及等额分期付款方式，返回投资在某一给定期间内的本金偿还额
	- **IPMT**
		- Interest on the Principal
		- 基于固定利率及等额分期付款方式，返回给定期数内对投资的**利息**偿还额
- 事实上，`PMT=PPMT+IPMT`


- eg：
	- 刘姥姥五年后孙子上学，每年存 5 块大洋，年利息 10%，五年后取出。六年学习，每年需要 10 块现大洋。假如刘姥姥不再工作（五年后），这笔钱够不够供上学？如果不够，还需要工作几年？
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912142752.png)
	- 可以看出暂时不够

	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912142946.png)
	- 可能还要考虑孙子的扣除
	- 我们以 `30.5255` 作为基准，继续计算
	- 获得期数，用 NPER
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912143254.png)
		- **NPER (Number of Periods)** 函数是 Excel 中用来计算达到某个目标所需的**期数**的财务函数。它非常适合解决“我需要存多少年/月才能攒够一笔钱”这类问题。
- **澄清**：现金流
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912144322.png)
	- **注意学费**在期初交。
	- 用 NPV 函数，可以计算贴现到现在的值。
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912144758.png)
		- **可以看出**
			- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912144823.png)
		- 工作 5 年不可以
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912144903.png)
		- 再工作 5 年就可以了。

#### NPER 函数的语法和参数

`=NPER(rate, pmt, pv, [fv], [type])`

- `rate`：**每期利率**。在刘姥姥的问题中，年利率是 10%。
    
- `pmt`：**每期付款**。刘姥姥每年存 5 块大洋，所以 `pmt` 是 5。
    
- `pv`：**现值**（Present Value）。在刘姥姥这个例子中，这个参数非常重要，它代表**期初已经拥有的金额**。如果我们把计算起始点放在五年后，刘姥姥已经存下了 30.53 块大洋，但她需要支付 60 块。所以她**还差** 29.47 块。这个缺口就是我们未来需要通过定投弥补的“现值”。因此，`pv` 应该是 **-29.47**。这里使用负数是因为它代表了一个**资金流出**（一个你需要去弥补的负债）。
    
- `[fv]`：可选，**未来值**（Future Value）。这是你希望达到的最终金额。在我们的例子中，刘姥姥的目标是完全弥补 29.47 的缺口，所以最终需要的未来值是 0，即 `fv` 为 0。
    
- `[type]`：可选，付款时间。0 或省略表示期末付款，1 表示期初付款。我们假设她在年末存钱，所以可以省略或设为 0。
    

#### 如何用 NPER 函数解决刘姥姥的问题

首先，我们需要明确，刘姥姥在五年后需要再存下的钱是：

- 六年学费总额：10×6=60 大洋
    
- 五年后已存总额：30.53 大洋（用 FV 函数计算得出）
    
- **资金缺口**：60−30.53=29.47 大洋
    

现在，我们可以把这个缺口看作是刘姥姥的“负债”，她需要通过每年定投来清偿这个负债，直到清零。

**使用 NPER 函数来计算所需年数：**

- `rate` = 10%
    
- `pmt` = -5 （这里使用负数，因为它代表一笔**资金流出**，与 `pv` 的正负号保持一致，即如果你把 `pv` 看作是负债，那么 `pmt` 就是用来偿还负债的流出资金）
    
- `pv` = 29.47
    
- `fv` = 0
    

将这些参数代入 NPER 函数：

`=NPER(10%, -5, 29.47, 0)`

这个公式计算的结果会是 **4.7 年左右**。



### Risk
- **我们**看的是增量而不是存量
	- 以中美关系为例：变得更差还是更好？
	- **而不要过度关心大局**

- **离散系数**
$$
\frac{\sigma}{E[\cdot]}
$$
- 单位收益所承担的风险。
- **风险收益比**


**风险收益**无差异曲线
- 投资者有不同的风险偏好


- **影响客观风险承受能力的因素**
	- 年龄？态度？
	- 风险问卷...



## 银行资本
- **账面资本**
- 监管资本（RC）：法律概念
	- **监管者根据巴塞尔资本协议要求银行持有的最低资本**。
	- 监管者对银行资本的外部管理
- **经济资本**
	- 在一个给定的容忍度下，用来吸收由所有风险带来的非预期损失的资本，它是衡量银行真正所需要的资本的一个风险尺度。

- 确定风险要发生：资本与风险之间的关系
	- 定价转移预期损失
	- **资本**覆盖 **非预期损失**


# 复习课

- **什么是公司理财**
	- 企业管理的一部分，是有关企业资金的获取和管理
- **公式的计算**
	- 贴现
	- 股利分配
	- PV，FV 的计算
- **资本预算**
	- 知道一个资产负债表如何排列
	- 一般把流动性强的放在上面

- Chapter 2 up
- 货币时间价值
- 单利的计算
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219140820.png)

- 复利的计算
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219140831.png)
- 复利现值的计算
- $P_{n}=P_{0}(1+i)^{n}$
- $P_{0}=P_{n}(1+i)^{-n}$  

- 年金的计算
- 年金——指在相同的间隔时间内陆续收到或付出的相同金额的款项
- ![](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141011.png)
- 等比数列求和
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141050.png)
- **先付年金**：同理，就是下标都往外挪一个就可以了
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141118.png)
- **递延年金**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141417.png)
- **永续年金**
$$
PV_{Perp}=\lim_{ n \to \infty } \left( v_{0}{\frac{\cdot{1-(1+r)^{-n}}}{r}} \right)=\frac{v_{0}}{r} 
$$

有一些习题，**看一下**



---

- **风险的度量和管理**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141636.png)
- **组合风险**：对各部分协方差加权求和
- $Var(r_{p})=\sigma_{p}^{2}=\sum \sum w_{i}w_{j}cov(i,j),cov(i,i)=\sigma_{i}^{2}$


- *有效投资组合*： 
	- 满足以下条件
	- 给定风险条件，提供最大预期收益率
	- 给定预期收益率，提供最小风险

- Beta 系数 (CAPM) 模型
- 阅读[[金融经济学#资产定价关系 CAPM]]

- 风险收益无差异曲线：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219141914.png)
- **构成相同的期望效用**
- 不同风险形状：风险偏好的向下偏

- **经济资本**
	- 经济资本是指在一个给定的容忍度下，用来**吸收由所有风险带来的非预期损失的资本**，它是测量银行真正所需资本的一个风险尺度。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219142412.png)
- 定价转移预期损失
- 资本覆盖非预期损失
- 压力测试应对极端损失


---

**资本负债表**
- 左边资金占用，右边资金来源
- 变现能力强的在上面，变现能力弱的在下面

**损益表**- 资产负债表的变化

**现金流量表**
- 功能
	- 提供企业在某一特定期间内有关现金和现金等价物的流入和流出的信息

---
公司金融的职能

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219142717.png)
- 规划、预算



---

**财务分析**
- 静态分析
- 趋势分析
- 同行业比较
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219143900.png)

---
**投资决策**

**估值方法**：
- **净现值法**
	- 一项投资的市值和成本之间的差值。
	- 需要估计折现率
- **回收期法则**
	- 收回我们初始投资所需要的时间
- **贴现回收期**
	- 由于回收期没有考虑货币的时间价值，所以产生了贴现回收期
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219144539.png)
- **平均会计报酬率**
	- ARR
	- `平均净利润 / 平均账面价值`


- **内含报酬率**（IRR）
	- 一项投资的 IRR 就是 NPV 等于 0（或者短期内正好收回成本）时的贴现率
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219144649.png)
	- **内含报酬率**（Internal Rate of Return，简称 **IRR**）是指使投资项目**现金流净现值（NPV）等于零**的折现率。简单来说，它是项目预期能产生的年化复合收益率，是评估投资项目盈利能力的最常用指标之一。

 **IRR 的定义（数学公式）**
IRR 是以下方程的解 r：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219155323.png)
- IRR **缺点**
	- 不修正现金流


- **修正后的内含报酬率**（MIRR）
	- 修正现金流

- **获利能力指数**

- **资本预算实务**


- **项目分析与评估**
	- 保本点分析 (Break-even analysis)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219150355.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219150432.png)


---
***融资决策***

- 公开发售证券
- 认股权
	- 对现有股东发行普通股：认股发行（rights offering）
	- 简称发行或优先认股 (privileged subscription)


- 资本成本与长期财务决策
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219150907.png)

	- **资本重组**：restructuring
	- 公司用来改变现有资本结构的活动

- **MM 第二定理** #必考 
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219160935.png)
- 看一下 25 题

- **利息税盾**
- 利息可以在税前扣除

- **静态资本结构理论**
	- 静态资本结构理论说明公司的债务数量会到达某一点，在这一点上，每1美元的债务所产生的税盾好处正好等于提高财务困境概率的成本。我们把这个关系称为静态理论，因为它假设从资产和经营的角度来看，公司是固定的，因而它只考虑债务权益率可能的变动。

- **啄食顺序理论**
	- 核心：企业更倾向于使用内部融资
	- 理由：出售有价证券的融资成本更高，所以企业会尽可能避免。
- **并购**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219151818.png)

---

**股利分配决策**

**群落效应**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219151953.png)
- 不同的群体：`clientele`
- 群落效用：不同的投资者全体要求不同的股利水平

- 股利分配的另一种形式：

- 股利分拆


## Review

选择-多选-判断-填空-名词解释-简答-计算

![feb0ffc919772e13be2052054a43c708.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/feb0ffc919772e13be2052054a43c708.png)

（1）某公司今年年初有流动资产 410 万元，流动负债 140 万元。年末流动资产 450 万、流动负债 190 万，请问该公司的净营运资本变动额是多少？
变动：
（经营营运资本 = 经营性流动资产 - 经营性流动负债）

260w - 270w = -10w

（2）

流动比率为 1:
增加长期贷款


杜邦财务分析体系的核心指标：
- ROE 净资产收益率

6. 计算 1万元五年后的价值：复利的终值系数
7. 回收期法的优缺点：
	1. 容易理解
	2. 调整不确定性
	3. 偏向于高流动性
	4. 没有考虑时间的不确定性！（**缺点**）
8. 公司理财的基本观念
	1. 时间价值、风险价值
	2. 不包括生产价值
9. 货币时间价值：
	1. 不只和时间长短有关


![9e8d98826188e98089a681bebfdc7842.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/9e8d98826188e98089a681bebfdc7842.png)

11. 
	- **递延年金**：理解一样，只是多了一笔等待期
	- NPV：net present valu
12. 提高折现率会导致

13. 200 万流通在外，我需要 4 份。
14. C：其实规模小，不稳定的常见，风险溢价。选择 D
15. 区分没有那么重要
16. 获得税收


![f75541710b7ad52e36619a892304ff03_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/f75541710b7ad52e36619a892304ff03_720.png)

- 18: 协同效应也有税的影响
- C 是对的：只有获益才会做

![e5a993a26a0bcafaf86cf2182af702e7_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/e5a993a26a0bcafaf86cf2182af702e7_720.png)

- 19: A
- 不能投资 B，会有亏损。

- 20: 敏感性分析：**就是改变一个变量**，同时其他变量不变。选择 B


![c52e9f783a5aaf5fd1ebc768ad94a514_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/c52e9f783a5aaf5fd1ebc768ad94a514_720.png)

- 21
	- 这是一个投资组合
	- 判别风险。选择 D 协方差最小，**所以风险最好**
	- 选择 C
- 21（两个 21， 233）
	- 分散化：**剩下的就是整个市场组合的风险**
- 22
	- **年金现值**：
- 22. CAPM
	- D 是错误的。beta 衡量非系统风险
- 23
	- 加权资本成本为 16%, 预测明年税收债务成本下降 3%，修正债务比率为 50%，甲醛资本成本为
	- $16\%=\frac{1}{2}R_{E}+\frac{1}{2}(R_{F}-3\%)=\frac{1}{2}R_{E}+\frac{1}{2}R_{F}-1.5\%=16\%-1.5\%$
	- ans: 14.5%
![c907eff4e81550fe2aacdb335f67cf9a_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/c907eff4e81550fe2aacdb335f67cf9a_720.png)

- 25
	- 无负债时的资本成本为 9%，考虑采用 50%负债的新资本结构，负债的利息率为 4%，如果所得税率为 25%，则新资本结构下的权益资本成本为（）
	- grok 老师：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219160714.png)
	- 带入公式，得到 C：`12.75%`
	- 

![b1beb9f742bb6f12fb59d66447fece00.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/b1beb9f742bb6f12fb59d66447fece00.png)
 
 - 26
	- **纵向并购正确答案**
	- 横向并购：同行业


- 27
	- 折现率 13%，NPV 50w，折现率 15%，净现值-20W，内含报酬率最接近？
	- 可以知道在 13%和 15%之间。所以是 14%。
	- 因为 IRR 是使得 NPV 为 0 的贴现率



![a575d4441ac41bac0deb7648d1588e6c_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/a575d4441ac41bac0deb7648d1588e6c_720.png)

28.  短期偿债：流动比率？

- 29
	- CAPM
	- r= ()

![85571efb0d666e61ad0a58d7672fbdf4_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/85571efb0d666e61ad0a58d7672fbdf4_720.png)

- 33

- 34
	- 优点：三点都是
	- 精确性综合性通俗性
- 35 财务杠杆公式
	- 息税前利润/(息税前利润-债务利息)

![aef3b11f4103a4d17ab2a19956b33ed6.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/aef3b11f4103a4d17ab2a19956b33ed6.png)

- 36
	- **权益成本计算公式**
- 37
	- **换股比例**：并购、兼并都可以采用换股的方式
	- 最低换股比例：
		- 新发行股票占最新的股票应该等于 16 万元



---

![2d1694cf0ceae34450f3469752ce6f09.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/2d1694cf0ceae34450f3469752ce6f09.png)

- 2
	- 经济复苏适合股票 A
- 5
	- 流动性原则
- 8 
	- 单利计算。
	- 掌握公式
- 
![85b4ec16ebb7d5fed029d97d349d6aab_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/85b4ec16ebb7d5fed029d97d349d6aab_720.png)

- 10
	- 不属于：
- 11 递延年金特点：没有第一期
- 12
	- 风浪越大鱼越多
- 13 **协方差**。回忆加权公式
- 14
	- **可以利用财务杠杆**
	- 增加企业价值
- 15
	- **优先股**、**普通股**的定义
	- 组成要素
	- 优先股是双重性质的债券

![28fadda977b2ce028b223c15625f62f3_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/28fadda977b2ce028b223c15625f62f3_720.png)

- 16
	- 债券最大风险
	- 股票只是一部分财务风险最小
- 17
	- 面值和利息分别计算。
- 18:
	- 计算期限
	- `100 / (1 + 0.12)^n = 88.70`
	- time:

- 21
	- 利润什么时候覆盖投资
- 22
	- 忽略了现金流状况：
	- 只在乎收回投资，不在乎之后的现金流状况。A

![bcd0abf19cdfbd3d6598af8883f11ba6_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/bcd0abf19cdfbd3d6598af8883f11ba6_720.png)

- 26
	- **识别关键因素**
		- 敏感度分析法。
		- ~~控制变量~~
- 27
	- 盈亏
	- #TODO 
- 28
	- 不存在杠杆：


![059d65d518591898eb9e53e754eda0ce_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/059d65d518591898eb9e53e754eda0ce_720.png)


- 30
	- 经营杠杆产生：**不变的固定成本**
- 31
	- 财务杠杆系数计算

![eefe389aaef8cd322527d925dc6d95d4_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/eefe389aaef8cd322527d925dc6d95d4_720.png)

- 37
	- 15%
	- **固定增长模型**
- 38
	- 企业发行债券
	- 平价/溢价/折价发行
	- 平价发行：价格一致
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219164250.png)


- 39
	- **加权平均资本成本** C
- 40
- 41
	- **资本结构**
	- 债务和权益配比变化：structure reshaping
	- **影响财务风险**

![1584f4b85bfbed136ade3768a0c3329c_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/1584f4b85bfbed136ade3768a0c3329c_720.png)


![7d134fc8277b2cd543f909c8344f34b1_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/7d134fc8277b2cd543f909c8344f34b1_720.png)

- 47
	- 不是一起分红
	- 也不是同股同权
	- 选 A
- 48
	- **国债**
	- A（定义）
- 49
	- 技术分析：
	- 市场有效性：
		- 弱势
		- 中势
		- 强势

- 50


---
DUOXUAN

![c71fd902ccd28357670e637a30711e41_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/c71fd902ccd28357670e637a30711e41_720.png)


2. 越大越好
	1. 回收期不是。
	2. ACD
- 3. 财务操纵基本形式


![690082ba671600571151693f659db0e2_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/690082ba671600571151693f659db0e2_720.png)


- 5 
	- 增加资本
- 6
	- **系统性风险定义**
	- ACD
- 7
- 8 公司金融的主要内容
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219163617.png)

![ec77f26b56600ddaa8c6b7e689f9f879_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/ec77f26b56600ddaa8c6b7e689f9f879_720.png)

- 11 三因子模型


![731e00d9f18a697e64518c4ea6f8b9cd_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/731e00d9f18a697e64518c4ea6f8b9cd_720.png)

- 1 时间价值

![d23405a69ff5177140b606b81741fca3_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/d23405a69ff5177140b606b81741fca3_720.png)

- **净现值计算**

- 11
- VAR：精确性、综合性、通俗性

- 12
	- 证券投资收益
	- A、C、B
- 13
	- 组合
	- 不可能分摊所有风险
	- 取决于协方差比率。

![eeecbc7778adc615fbff09e2eae1b356_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/eeecbc7778adc615fbff09e2eae1b356_720.png)


- 17
	- 降低经营风险
	- B、C、D？

### 计算题
![73573d128b204e023c380837b5e34baf_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/73573d128b204e023c380837b5e34baf_720.png)

WACC：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219164611.png)



---



## 速查表
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219163959.png)

- 有些问题，还需要额外的查询课件和做题