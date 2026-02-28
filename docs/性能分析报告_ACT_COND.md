# 工程性能分析报告（ACT / COND 及全工程）

> 分析日期：基于当前代码库  
> 重点目录：`Assets/Scripts/BehaviorTree/ACT`、`Assets/Scripts/BehaviorTree/COND`

---

## 一、严重性能问题（优先修复）

### 1. 【COND】Cond_FindTarget.fcg：每 Tick 调用 GetAllPlayers() + 遍历

**位置**：`Assets/Scripts/BehaviorTree/COND/Cond_FindTarget.fcg` 第 38 行

**问题**：
- `PlayCondition` 在行为树**每次 Tick** 时会被调用（各怪物的 `TickInterval = 1`，即每帧或极高频率）。
- 当没有仇恨目标且当前目标失效时，会执行：
  - `GetAllPlayers()`：可能涉及内部集合构建/拷贝。
  - `for i, player in GetAllPlayers()`：对**所有玩家**做距离判断。
- 场上约 14 只怪（`FC_PVEMonsterManager.MaxMonstersOnField = 14`），每只每 Tick 都可能走这段逻辑 → **GetAllPlayers() 调用次数 = O(怪物数 × Tick 频率)**，极易成为 CPU 热点。

**影响**：高。条件节点在树中常处于上层，调用频率极高。

**建议**：
- **方案 A（推荐）**：在**没有当前目标**时再做“找新目标”逻辑，并加**时间节流**，例如每 200–500ms 才执行一次“遍历玩家找目标”，其余 Tick 直接返回 `false`（或沿用上一帧结果），避免每帧每怪都 `GetAllPlayers()`。
- **方案 B**：若 PVE 仅 1 个玩家，用 `GetLocalPlayer()` 或单玩家接口替代 `GetAllPlayers()`，只做一次距离判断。
- **方案 C**：在全局/管理器层维护“当前存活玩家列表”的缓存，定时（如每 0.5s）更新一次，COND 只读缓存并做距离判断，避免每 Tick 调用 `GetAllPlayers()`。

---

### 2. 【ACT】Act_Move2Target / Act_Move2Target_ShieldZombie / Act_Attack_BossButcher_Spin：SingleRaycast + 每次创建 List<int>{0}

**位置**：
- `Assets/Scripts/BehaviorTree/ACT/Act_Move2Target.fcg` 约第 84 行
- `Assets/Scripts/BehaviorTree/ACT/Act_Move2Target_ShieldZombie.fcg` 约第 84 行
- `Assets/Scripts/BehaviorTree/ACT/Act_Attack_BossButcher_Spin.fcg` 约第 85 行

**问题**：
- 在“间隔更新寻路”分支内调用：  
  `SingleRaycast(..., List<int>{0}, ...)`  
  每次进入该分支都会 **new 一个 List**，用于 layerMask。
- 虽有时间间隔（约 1000ms 一次），但多怪同时寻路时仍会产生**多余的小对象分配**，增加 GC 压力。

**影响**：中高。单次分配小，但调用点重复、怪物数量叠加后明显。

**建议**：
- 在 graph 内定义**成员变量**，例如：  
  `_RaycastLayerMask List<int> = List<int>{0}`  
  在 OnAwake 或首次使用前初始化一次，SingleRaycast 始终传入该成员，避免在 PlayAction 内重复 `List<int>{0}`。

---

### 3. 【COND】Cond_IsHit.fcg：逻辑错误导致可能重复评估

**位置**：`Assets/Scripts/BehaviorTree/COND/Cond_IsHit.fcg` 第 11–18 行

**问题**：
- 当 `Owner<Mob>.StatusDic["Hit"]` 为 true 时，只设置了 `StatusDic["InHit"] = true`，**没有 `return true`**。
- 函数在“Hit 为 true”的分支末尾没有返回值，行为未定义或等价于 false，导致“被击”条件可能永远不成立，或依赖引擎对未定义返回值的处理，从而引发**多余的条件重算**或逻辑错误。

**影响**：中。既影响正确性，也可能增加无效的条件评估。

**建议**：
- 在设置 `InHit` 后显式 **return true**；否则分支 **return false**。保证返回值明确，并减少无效评估。

---

## 二、中等性能影响（建议优化）

### 4. 【COND】频繁的 GetNodeVariable + StatusDic 字典访问

**涉及文件**：  
Cond_CanAttack.fcg、Cond_CanAttack_RandCD.fcg、Cond_CanAttack_ShieldZombie_AOE.fcg、Cond_CanPatrol.fcg、Cond_FindTarget.fcg 等。

**问题**：
- `GetNodeVariable(thisEntity<BevTreeCustomNode>, "SeekRange", ...)`、`"AttackName"`、`"KeepDist"`、`"CD"` 等在每个 PlayCondition 内按字符串查找。
- `owner<Mob>.StatusDic["CD_xxx"]`、`StatusDic["LastAttackTime_xxx"]` 等多次字典访问。
- 条件在行为树 Tick 时可能被频繁执行，字符串键查找 + 字典访问会放大开销。

**建议**：
- 在 **OnNodeEnter** 或首次进入时，用 GetNodeVariable 把常用参数取到 **graph 成员变量**（如 SeekRange、AttackName、KeepDist、CD），PlayCondition 内只读成员变量。
- 对同一帧内多次用到的 StatusDic 键（如同一攻击的 CD、LastAttackTime），可先取到局部变量再判断，减少重复字典访问。

---

### 5. 【ACT】Act_Patrol.fcg：GetTargetPos 内多次 Random 与 List 使用

**位置**：`Assets/Scripts/BehaviorTree/ACT/Act_Patrol.fcg` 中 `GetTargetPos` 及 Patrol 逻辑。

**问题**：
- `GetTargetPos` 内使用 `math.RandomFloat`、`math.RandomInt` 等，且与 `newTargetPosInterval` 的随机间隔一起，在每次取新目标时执行。
- 若 PathList 或其它列表在每 Tick 有创建/扩展，会带来额外分配（当前代码中 PathList 已注释或未在热路径使用，可保持不分配）。

**建议**：
- 保持“随机巡逻目标 + 随机间隔”的设计即可，避免在 **PlayAction 每 Tick** 里创建新的 List 或大对象。
- 若有其它 ACT 在 PlayAction 内 `List<Vector3>{}` 或类似写法，建议改为 graph 成员复用。

---

### 6. 全工程：GetAllPlayers() 的用法汇总

**调用位置**（仅列出可能在高频或关键路径的）：

| 文件 | 用途 | 频率/场景 | 风险 |
|-----|------|-----------|------|
| **Cond_FindTarget.fcg** | 无目标时遍历找玩家 | 每 Tick 每怪 | **高** |
| FC_Global_MobSkillHandler.fcg | 技能伤害范围检测、getPlayersInRange | 技能释放/异步 | 中（单次） |
| FC_Utilities.fcg | NotifyAllPlayers | 按需通知 | 低 |
| PVEController.fcg | 关卡/结算逻辑 | 按事件 | 低 |
| FinalStats.fcg | 结算 | 结算时 | 低 |
| FlowerZombie.fcg | getPlayersInRange（技能/事件） | 事件驱动 | 中（单次） |

**建议**：  
- 仅 **Cond_FindTarget** 需要按“节流 + 单玩家或缓存”做重点优化（见上文）。  
- 其它位置保持“按需调用、不放进每帧/每 Tick 热路径”即可。

---

## 三、ACT/COND 以外工程热点（简要）

- **FC_Global_MobSkillHandler**：技能中多次 `GetAllPlayers()` 与 `getPlayersInRange` 是在**单次技能流程**内，属事件驱动，影响小于 COND 每 Tick 调用。若同屏多怪同时放范围技，可考虑在全局层做“玩家列表/距离缓存”复用。
- **FC_PVEMonsterManager**：对象池、列表操作、CSV 读取已有缓存设计（如 LevelRowInfos、WaveMonsterInfos），保持现有“按需加载、避免每帧分配”即可。
- 未发现 **OnUpdate/OnFixedUpdate** 在 Scripts 下直接使用，行为树由 Tick 驱动，热点主要在 **Cond_FindTarget** 与 **PlayAction 内的射线/列表**。

---

## 四、修复优先级小结

| 优先级 | 项目 | 位置 | 建议动作 |
|--------|------|------|----------|
| P0 | GetAllPlayers 每 Tick 调用 | Cond_FindTarget.fcg | 节流 + 单玩家或全局玩家缓存 |
| P1 | SingleRaycast 的 List 分配 | Act_Move2Target / Act_Move2Target_ShieldZombie / Act_Attack_BossButcher_Spin | 使用 graph 成员 List，避免每次 new |
| P1 | 条件返回值缺失 | Cond_IsHit.fcg | 分支中补上 return true / return false |
| P2 | GetNodeVariable + StatusDic 频繁访问 | 多个 COND | 缓存到成员变量、减少字典与字符串查找 |
| P2 | 其它 GetAllPlayers 使用 | 见上表 | 保持事件驱动，不放入每帧/每 Tick |

---

## 五、建议的 Cond_FindTarget 节流示例思路（方案 A）

```text
// 在 graph 中增加成员：
LastFindTargetTime int = 0
FindTargetInterval int = 300   // 毫秒，例如 300ms 才重新找一次目标

// 在 PlayCondition 中，当 curTarget == nil 或需要重新找目标时：
var gameTime = globalEntity<Global>.GameTimeCount
if (gameTime - LastFindTargetTime) < FindTargetInterval {
    return false   // 未到间隔，不执行 GetAllPlayers
}
LastFindTargetTime = gameTime

// 然后再执行现有的“仇恨目标检查 + 当前目标检查 + for i, player in GetAllPlayers()”逻辑
```

单玩家时可将 `for i, player in GetAllPlayers()` 改为对 `GetLocalPlayer()`（或项目内单玩家 API）做一次距离判断，进一步减少开销。

以上为基于当前代码的结论与建议，实际效果以真机/编辑器 Profiler 为准。修改后建议重点对比：行为树 Tick 耗时、GC 频率、Cond_FindTarget 调用次数。
