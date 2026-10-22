# Home 与 Result 页面钩子

## 目标

给首页和结算页补上更接近真实页面的状态流，避免后面在 Creator 里还要重新定义加载态和提交态。

## HomeScene

参考：

- [HomeSceneComponent.ts](../apps/game/creator-templates/HomeSceneComponent.ts)

建议状态：

- `Loading leaderboard...`
- 正常展示排行榜
- `Load failed`

建议字段：

- `bestScoreLabel`
- `leaderboardLabel`
- `statusLabel`
- `startButton`

## ResultScene

参考：

- [ResultSceneComponent.ts](../apps/game/creator-templates/ResultSceneComponent.ts)

建议状态：

- `Submit`
- `Submitting...`
- `Submitted`
- `Submit failed`

建议字段：

- `scoreLabel`
- `bestScoreLabel`
- `reviveStateNode`
- `submitButton`
- `homeButton`
- `submitStatusLabel`

## 为什么这层很重要

小游戏原型常常先能跑，但页面状态乱。先固定这些状态位，后面接真接口和真实 UI 时就不会再反复改按钮逻辑。
