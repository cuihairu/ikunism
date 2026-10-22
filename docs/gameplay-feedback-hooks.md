# Gameplay 反馈钩子

## 目标

把 `Gameplay` 场景里“出手结果”和“视觉反馈”拆成稳定接口，避免后面接动画、音效、对象池时再改主流程。

## 当前结果判定

参考：

- [shot-input.ts](../apps/game/assets/scripts/gameplay/shot-input.ts)
- [shot-feedback.ts](../apps/game/assets/scripts/gameplay/shot-feedback.ts)

结果类型：

- `perfect`
- `hit`
- `miss`

## 当前反馈入口

参考：

- [gameplay-scene.ts](../apps/game/assets/scripts/cocos/gameplay-scene.ts)
- [GameplaySceneComponent.ts](../apps/game/creator-templates/GameplaySceneComponent.ts)

建议把后续表现统一收口到这些方法：

- `handleShotSuccess`
- `handleShotMiss`
- `playHitEffect`

## 建议接入内容

### `handleShotSuccess`

- 更新得分
- 更新连击
- 切换角色表情
- 刷新 HUD

### `handleShotMiss`

- 重置连击
- 切换失败表情
- 播放失败反馈

### `playHitEffect`

- 命中特效
- 完美出手特效
- 失误特效
- 命中音效
- 连击音效

## 为什么先拆这层

因为 Creator 真正开始接资源后，最容易膨胀的就是：

- UI 刷新
- 角色动画
- 特效播放
- 音效播放

先把钩子位置固定，后面往里填就不会把投篮输入和计时逻辑搅乱。
