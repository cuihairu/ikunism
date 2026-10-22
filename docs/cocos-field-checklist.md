# Cocos 字段挂载清单

## 目标

给四个场景各自列出一份 `@property` 级别的字段清单，方便你在 `Cocos Creator` 里直接挂节点。

## BootScene

参考：

- [boot-scene.ts](../apps/game/assets/scripts/cocos/boot-scene.ts)
- [boot-scene-schema.ts](../apps/game/assets/scripts/cocos/boot-scene-schema.ts)

建议字段：

- `statusLabel: Label`

## HomeScene

参考：

- [home-scene.ts](../apps/game/assets/scripts/cocos/home-scene.ts)
- [home-scene-schema.ts](../apps/game/assets/scripts/cocos/home-scene-schema.ts)

建议字段：

- `bestScoreLabel: Label`
- `leaderboardLabel: Label`
- `startButton: Button`

## GameplayScene

参考：

- [gameplay-scene.ts](../apps/game/assets/scripts/cocos/gameplay-scene.ts)
- [gameplay-node-map.ts](../apps/game/assets/scripts/cocos/gameplay-node-map.ts)

建议字段：

- `scoreLabel: Label`
- `comboLabel: Label`
- `timerLabel: Label`
- `powerBar: ProgressBar`
- `reviveButton: Button`
- `shotTouchArea: Node`
- `playerNode: Node`
- `ballNode: Node`
- `hoopNode: Node`
- `hitEffectRoot: Node`

## ResultScene

参考：

- [result-scene.ts](../apps/game/assets/scripts/cocos/result-scene.ts)
- [result-scene-schema.ts](../apps/game/assets/scripts/cocos/result-scene-schema.ts)

建议字段：

- `scoreLabel: Label`
- `bestScoreLabel: Label`
- `reviveStateNode: Node`
- `submitButton: Button`
- `homeButton: Button`

## Gameplay 事件接线

### ShotTouchArea

推荐绑定：

- `Node.EventType.TOUCH_START`
- `Node.EventType.TOUCH_END`
- `Node.EventType.TOUCH_CANCEL`

### ReviveButton

推荐绑定：

- `Button.EventType.CLICK`

### SubmitButton / HomeButton / StartButton

推荐绑定：

- `Button.EventType.CLICK`

## 一个简单做法

可以先在 Creator 里写一个真正的组件壳：

```ts
@ccclass("GameplaySceneComponent")
export class GameplaySceneComponent extends Component {
  @property(Label)
  scoreLabel: Label | null = null;
}
```

然后把仓库里的 `GameplayScene` 作为业务层实例挂进去，只让组件做字段绑定和事件转发。
