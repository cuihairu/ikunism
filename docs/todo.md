# Todo(代码深审 2026-11-01 转出)

> 格式:一行一需求,无装饰;来源见 `docs/review/2026-11-01-code-review.md`。

- [ ] B1 回合结束后禁止再累加分数,`finishRound` 只触发一次(gameplay-session.ts / gameplay-scene-controller.ts)
- [ ] B2 `/leaderboard/submit` 入参加长度与分数上限(server.go)
- [ ] B3 Creator 模板 `update` 的 `finishRound` 加 once 守卫(GameplaySceneComponent.ts)
- [ ] B4 投篮判定窗口改引用统一常量/配置,去硬编码(GameplaySceneComponent.ts)
- [ ] B5 HomeSceneComponent.loadHomeData 的不可达 `catch` 修正或删除
- [ ] B6 删除或导出 `SceneFlow`/`PlatformBridge` 孤儿类
- [ ] B7 素材清单脚本将目标路径约束在素材根内(rename-from-manifest.ps1)
