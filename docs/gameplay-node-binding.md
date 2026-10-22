# Gameplay 节点绑定清单

## 目标

给 `Gameplay` 场景提供一份可以直接照着建节点的最小绑定清单，减少你在 Creator 里来回找节点。

## 推荐节点路径

代码默认参考 [gameplay-node-map.ts](../apps/game/assets/scripts/cocos/gameplay-node-map.ts)。

建议最小节点：

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

## 每个节点职责

### HUD

- `ScoreLabel`：显示当前得分
- `ComboLabel`：显示当前连击和情绪状态
- `TimerLabel`：显示剩余时间
- `PowerBar`：显示当前蓄力比例
- `ReviveButton`：局内复活按钮或结算复活入口

### Input

- `ShotButton`：承接按下和松开事件

### Gameplay

- `Player`：小鸡角色
- `Ball`：篮球
- `Hoop`：篮筐
- `HitEffectRoot`：命中和连击特效挂载点

## 输入节奏

当前最小模板采用“长按蓄力，松手投篮”。

参考代码：

- [shot-input.ts](../apps/game/assets/scripts/gameplay/shot-input.ts)
- [gameplay-scene.ts](../apps/game/assets/scripts/cocos/gameplay-scene.ts)

规则：

- 按下：开始蓄力
- 长按期间：`PowerBar` 增长
- 松开：
  - 蓄力在完美区间内：按 `3` 分命中
  - 蓄力在普通区间内：按 `2` 分命中
  - 过短或过长：判定失误

## Creator 接入建议

正式接入时，`ShotButton` 可以有两种做法：

1. 用 `Node` 的触摸事件：
   - `TOUCH_START`
   - `TOUCH_END`
   - `TOUCH_CANCEL`
2. 用按钮区域包一层透明输入节点，不直接依赖 `Button` 组件点击

第二种通常更适合做长按蓄力。
