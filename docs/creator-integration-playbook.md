# Creator 接入手册

## 目标

把当前仓库里的游戏模板真正迁进 `Cocos Creator`，并在最短路径内跑通：

- `Boot`
- `Home`
- `Gameplay`
- `Result`

四个场景的空壳流程。

## 第 1 步：初始化 Creator 工程

建议使用：

- `Cocos Creator 3.8.x`

在 `apps/game` 对应位置创建正式项目后，先不要急着接资源，先把空场景流转跑通。

## 第 2 步：先复制哪批文件

优先复制这两层：

### A. 真实组件壳

从 [creator-templates](../apps/game/creator-templates/README.md) 复制：

- [BootSceneComponent.ts](../apps/game/creator-templates/BootSceneComponent.ts)
- [HomeSceneComponent.ts](../apps/game/creator-templates/HomeSceneComponent.ts)
- [GameplaySceneComponent.ts](../apps/game/creator-templates/GameplaySceneComponent.ts)
- [ResultSceneComponent.ts](../apps/game/creator-templates/ResultSceneComponent.ts)
- [shared.ts](../apps/game/creator-templates/shared.ts)

### B. 业务模板层

按需复制这些目录：

- `apps/game/assets/scripts/core`
- `apps/game/assets/scripts/gameplay`
- `apps/game/assets/scripts/ui`
- `apps/game/assets/scripts/platform`
- `apps/game/assets/scripts/cocos`

如果第一轮只想跑空壳，也可以先只复制 `creator-templates`。

## 第 3 步：先建四个场景

需要先建：

- `Boot.scene`
- `Home.scene`
- `Gameplay.scene`
- `Result.scene`

## 第 4 步：每个场景先建哪些节点

### Boot

- `Canvas`
- `Canvas/StatusLabel`

### Home

- `Canvas`
- `Canvas/HUD/BestScoreLabel`
- `Canvas/HUD/LeaderboardLabel`
- `Canvas/HUD/StatusLabel`
- `Canvas/Actions/StartButton`

### Gameplay

- `Canvas`
- `Canvas/HUD/ScoreLabel`
- `Canvas/HUD/ComboLabel`
- `Canvas/HUD/TimerLabel`
- `Canvas/HUD/PowerBar`
- `Canvas/HUD/ReviveButton`
- `Canvas/Input/ShotButton`
- `Canvas/Gameplay/Player`
- `Canvas/Gameplay/Ball`
- `Canvas/Gameplay/Hoop`
- `Canvas/Gameplay/HitEffectRoot`

### Result

- `Canvas`
- `Canvas/HUD/ScoreLabel`
- `Canvas/HUD/BestScoreLabel`
- `Canvas/HUD/ReviveState`
- `Canvas/HUD/SubmitStatusLabel`
- `Canvas/Actions/SubmitButton`
- `Canvas/Actions/HomeButton`

## 第 5 步：先挂哪些字段

字段清单直接看：

- [docs/cocos-field-checklist.md](cocos-field-checklist.md)

挂载顺序建议：

1. `Label`
2. `Button`
3. `ProgressBar`
4. `Node`

先保证 Inspector 不报空，再考虑逻辑。

## 第 6 步：第一轮需要哪些占位资源

先不要做正式美术，用占位资源即可：

- 一个纯色背景
- 一个圆形节点当篮球
- 一个矩形节点当篮板/篮筐
- 一个简单 Sprite 当小鸡
- 一个空节点当特效根节点

音效和动画都可以先空着。

## 第 7 步：第一轮先接哪些事件

### Boot

- `onLoad`

### Home

- `StartButton -> CLICK`

### Gameplay

- `ShotButton -> TOUCH_START`
- `ShotButton -> TOUCH_END`
- `ShotButton -> TOUCH_CANCEL`
- `ReviveButton -> CLICK`

### Result

- `SubmitButton -> CLICK`
- `HomeButton -> CLICK`

## 第 8 步：第一轮验收标准

只要做到下面这些，就算 Creator 接入第一阶段完成：

- 打开项目能进 `Boot`
- `Boot` 自动跳 `Home`
- `Home` 点开始能进 `Gameplay`
- `Gameplay` 长按能看到力度条变化
- 松手后能触发命中或失误分支
- 倒计时结束能进 `Result`
- `Result` 能显示分数和最高分
- `HomeButton` 能回首页

## 第 9 步：第二轮再接什么

第一轮跑通后，再按这个顺序补：

1. 接真实 `GameManager / AppContext`
2. 接真实配置拉取
3. 接真实排行榜
4. 接激励广告
5. 接角色表情和命中特效
6. 最后替换正式资源

## 推荐配套文档

- [docs/cocos-project-plan.md](cocos-project-plan.md)
- [docs/cocos-component-mapping.md](cocos-component-mapping.md)
- [docs/cocos-field-checklist.md](cocos-field-checklist.md)
- [docs/gameplay-node-binding.md](gameplay-node-binding.md)
- [docs/gameplay-feedback-hooks.md](gameplay-feedback-hooks.md)
- [docs/home-result-hooks.md](home-result-hooks.md)
