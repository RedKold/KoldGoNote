
[[mine-combat/how-to-play-a-card]]


**简介**
- 作为Minecraft启发的卡牌游戏，设计开发采取开放式的策略，旨在提供方便的接口制作Mod。卡牌创新性的引入材料和合成表系统。在不同场景合成卡牌，打败怪物，成为我的卡牌之神吧！

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251026223036.png)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251026223239.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251031135924.png)

- `灰色`
	- `#c6c6c6`



- **生成什么样的卡牌**是通过 `--- CREATORS --- ` 中的 `CardViewCreator` 实现的



## 项目说明

主要说明当前 `Game` 场景的内容

- `--- SYSTEM ---`
	- 这里多数项目都是单例 `Singleton`，单例的意义可以查询网络。
	- 他们负责管理全局性的内容，挂载了一个脚本来管理。
	- 具体说明
		- `CardDragSystem`
			- 此系统用于管理卡牌拖拽行为，包括放开之后检测是否打到某 `Player` 从而出发卡牌打出行为。
				- 检测 `hit` 是通过鼠标发出的 `RaycastHit` 和用户的 `collidor` 碰撞，然后读取信息完成的
			- `CardDragSystem` 中定义了一个 `Interface` 类型，即 `IPlayArea`，我实现了一个玩家级别的 `IPlayer` 的具体实现即 `GameAction/PlayerPlayArea.cs`，可以查阅。
		- `SinglePlayerSystem`
			- 此系统用于单人游戏管理。默认注册了一个玩家，并提供了访问方法。其他脚本可以通过这个单例访问到。
			- 同时其也注册了一个 `Enemy`，这个可以后来改一改。
			- 为了让其完成玩家的初始化，防止 null 错误，我们应当最先执行这个脚本。`[DefaultExecutionOrder(-100)]  // 数字` 越小越早执行 `
		- `TestSystem` 
			- 是当前测试的入口。其 `Update()` 方法中是卡牌的