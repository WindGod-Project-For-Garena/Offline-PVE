---
description: FC语言完整编程参考 - 语法、API、组件、事件、最佳实践的全面指南
globs: *.fcg,*.fcc
applyTo: "**/*. {fcc,fcg}"
alwaysApply: true
---

# FC语言完整编程参考

> 基于 Free Fire Craftland 官方文档: https://ffcraftland.garena.com/en/docs/api

##  目录

1. [核心概念](#核心概念)
2. [语法参考](#语法参考)
3. [类型系统](#类型系统)
4. [内置库API参考](#内置库api参考)
5. [组件系统](#组件系统)
6. [事件系统](#事件系统)
7. [编程模式](#编程模式)
8. [最佳实践](#最佳实践)
9. [调试指南](#调试指南)
10. [常见问题](#常见问题)

---

## 核心概念

### FC语言设计理念
FC（Free Fire Code Language）是面向数据设计的DSL，用于Free Fire UGC游戏开发。

**核心特点：**
- 面向数据设计（而非面向对象）
- 实体-组件架构（Entity-Component）
- 强类型系统
- 异步编程支持
- 跨平台兼容

### 文件类型

| 扩展名 | 类型 | 用途 | 示例 |
|--------|------|------|------|
| `.fcc` | 库定义文件 | 定义组件、类型、API、事件、枚举 | `MyLibrary.fcc` |
| `.fcg` | 脚本逻辑文件 | 编写graph、变量、函数、事件处理 | `Script.fcg` |

### 平台声明

```fc
// 客户端脚本
[platform_client]
graph ClientScript {
    // 只能使用客户端支持的API和事件
}

// 服务端脚本（默认）
[platform_server]
graph ServerScript {
    // 服务端逻辑
}
```

---

## 语法参考

### 导入系统

```fc
// 导入内置库
import "StdLibrary.fcc" as StdLib

// 导入编辑器生成库
import "EditorGenLib.fcc" as EditorLib

// 导入用户自定义库
import "MyCustomLib.fcc" as MyLib

// 导入其他脚本
import "../Scripts/Utils.fcg" as Utils
```

### 变量声明

```fc
// 局部变量
var num int = 10
var name = "Player"  // 类型推断
var pos Vector3     // 未初始化

// 脚本成员变量
graph MyGraph {
    PlayerCount int = 0
    GameStarted bool = false
    
    event OnAwake() {
        PlayerCount = 5
    }
}
```

### 函数定义

```fc
// 普通函数
func Add(a int, b int) int {
    return a + b
}

// 异步函数
async func DelayedAction(delay int, out var result string) {
    WaitForMillisecond(delay)
    result = "完成"
}

// 无返回值函数
func LogMessage(msg string) {
    LogInfo(msg)
}
```

### 流程控制

#### 条件语句
```fc
if health > 50 {
    LogInfo("健康状态良好")
} else if health > 20 {
    LogInfo("健康状态一般")
} else {
    LogInfo("健康状态危险")
}
```

#### 循环语句
```fc
// for循环 (start, end, step) - 左闭右开区间
for i = 0, 10, 1 {
    LogInfo("计数: " + i)
}

// for range循环
var myList = List<int>{}
for index, value in myList {
    LogInfo("索引: " + index + ", 值: " + value)
}

// while循环
while playerCount < maxPlayers {
    SpawnPlayer()
    playerCount = playerCount + 1
}
```

### 异步编程

```fc
graph AsyncExample {
    event OnGameStart() {
        // 等待异步完成
        wait DelayedSpawn(3000, out var success)
        if success {
            LogInfo("玩家生成成功")
        }
        
        // 异步执行，不等待
        start BackgroundTask()
    }
    
    async func DelayedSpawn(delay int, out var success bool) {
        WaitForMillisecond(delay)
        // 生成玩家逻辑
        success = true
    }
    
    async func BackgroundTask() {
        // 后台任务
        WaitForMillisecond(1000)
        LogInfo("后台任务完成")
    }
}
```

---

## 类型系统

### 基本类型

| 类型 | 描述 | 示例 |
|------|------|------|
| `bool` | 布尔类型 | `true`, `false` |
| `int` | 32位整型 | `42`, `-10` |
| `int64` | 64位整型 | `9223372036854775807` |
| `float` | 单精度浮点 | `3.14`, `-0.5` |
| `string` | 字符串 | `"Hello World"` |
| `LocString` | 本地化字符串 | 多语言文本 |

### 数学类型

```fc
// 向量类型
var pos2D Vector2 = Vector2{1.0, 2.0}
var pos3D Vector3 = Vector3{1.0, 2.0, 3.0}

// 四元数（旋转）
var rotation Quaternion = Quaternion{0.0, 0.0, 0.0, 1.0}

// 访问分量
var x = pos3D.X
var y = pos3D.Y
var z = pos3D.Z
```

### 容器类型

```fc
// 列表
var numbers List<int> = List<int>{}
numbers.Add(1)
numbers.Add(2)

// 映射
var playerScores Map<string, int> = Map<string, int>{}
playerScores["Player1"] = 100
playerScores["Player2"] = 85

// 访问
var score = playerScores["Player1"]
var firstNumber = numbers[0]
```

### 实体类型

```fc
// 泛型实体类型
var player entity<Player>
var box entity<LevelObject>

// 访问组件
var playerName = player<Entity>.Name
var boxPosition = box<Transform>.Position

// 当前实体
var currentPos = thisEntity<Transform>.Position
```

### 资源ID类型

```fc
// 常用资源ID类型
var meshId MeshID = "mesh_cube_001"
var audioId AudioID = "sound_explosion"
var animId AnimationClipID = "anim_walk_cycle"
var materialId MaterialID = "mat_grass"
var prefabId PrefabID = "prefab_building"
```

---

## 内置库API参考

> 完整API参考：https://ffcraftland.garena.com/en/docs/api

### 核心库 (StdLibrary)

#### 日志函数
```fc
// 基础日志
func LogInfo(content object)     // 信息日志
func LogWarning(content object)  // 警告日志
func LogError(content object)    // 错误日志

// 使用示例
LogInfo("游戏开始")
LogWarning("玩家血量低: " + health)
LogError("关键错误: " + errorMsg)
```

#### 时间函数
```fc
// 等待函数
func WaitForMillisecond(time int)          // 等待毫秒
func WaitForSecond(time float)             // 等待秒
func WaitForFrame(frameCount int)          // 等待帧

// 时间获取
func GetCurrentTimestamp() int64           // 获取时间戳
func GetGameTime() float                   // 获取游戏时间
```

### 数学库 (Math)

#### 数学常量
```fc
// 常量枚举
enum NaturalConstantType {
    Pi = 3.14159265    // 圆周率
    E = 2.71828183     // 自然常数
}
```

#### 向量运算
```fc
// Vector3运算
func DotProduct(a Vector3, b Vector3) float                    // 点积
func CrossProduct(a Vector3, b Vector3) Vector3                // 叉积
func Vector3Distance(a Vector3, b Vector3) float               // 距离
func Vector3Normalize(value Vector3) Vector3                   // 归一化
func Vector3Magnitude(value Vector3) float                     // 长度

// 插值函数
func LerpVector3(a Vector3, b Vector3, t float) Vector3        // 线性插值
func SlerpVector3(a Vector3, b Vector3, t float) Vector3       // 球面插值
```

#### 四元数运算
```fc
// 四元数转换
func QuaternionToEulerAngle(value Quaternion) Vector3          // 转欧拉角
func EulerAngleToQuaternion(value Vector3) Quaternion          // 欧拉角转四元数
func LookRotation(forward Vector3, upward Vector3) Quaternion  // 朝向旋转

// 四元数插值
func SlerpQuaternion(valueA Quaternion, valueB Quaternion, valueT float) Quaternion
```

#### 数值运算
```fc
// 基础数学
func Abs(value float) float                    // 绝对值
func Sqrt(value float) float                   // 平方根
func Pow(base float, exponent float) float     // 幂运算
func Sin(value float) float                    // 正弦
func Cos(value float) float                    // 余弦
func Tan(value float) float                    // 正切

// 范围运算
func Clamp(value float, min float, max float) float           // 限制范围
func Lerp(a float, b float, t float) float                    // 线性插值
func Max(a float, b float) float                              // 最大值
func Min(a float, b float) float                              // 最小值
```

### 物理库 (Physics)

#### 刚体组件
```fc
component Rigidbody {
    Type RigidbodyType          // Static/Dynamic/Kinematic
    UseGravity bool             // 使用重力
    Mass float                  // 质量(kg)
    Velocity Vector3            // 线性速度(m/s)
    AngularVelocity Vector3     // 角速度(rad/s)
    Drag float                  // 阻力系数
    AngularDrag float           // 角阻力系数
    Constrain int               // 约束条件
}
```

#### 碰撞器组件
```fc
component Collider {
    Enable bool                 // 启用状态
    IsTrigger bool             // 是否为触发器
    PhysicalLayer int          // 物理层
}

// 碰撞事件
event OnTriggerEnter(other entity<Entity>)
event OnTriggerExit(other entity<Entity>)
event OnCollisionEnter(other entity<Entity>)
event OnCollisionExit(other entity<Entity>)
```

#### 物理函数
```fc
// 施加力
func AddForce(target entity<Rigidbody>, force Vector3, forceType ForceType)
func AddTorque(target entity<Rigidbody>, torque Vector3, forceType ForceType)

// 射线检测
func Raycast(origin Vector3, direction Vector3, distance float, out var hit RaycastHit) bool
func RaycastAll(origin Vector3, direction Vector3, distance float) List<RaycastHit>
```

### 玩家库 (Player)

#### 玩家组件
```fc
component Player {
    AccountID UUID              // 账户ID
    NickName string            // 昵称
    TeamID int                 // 队伍ID
    Health float               // 血量
    MaxHealth float            // 最大血量
    Speed float                // 移动速度
    State PlayerStateType      // 玩家状态 (Alive/KnockedDown/Dead)
}
```

#### 玩家函数
```fc
// 玩家操作
func KillPlayer(player entity<Player>)                        // 击杀玩家
func RevivePlayer(player entity<Player>)                      // 复活玩家
func TeleportPlayer(player entity<Player>, position Vector3)  // 传送玩家
func SetPlayerHealth(player entity<Player>, health float)     // 设置血量

// 玩家查询
func GetAllPlayers() List<entity<Player>>                      // 获取所有玩家
func GetPlayerByIndex(index int) entity<Player>               // 按索引获取
func GetLocalPlayer() entity<Player>                          // 获取本地玩家
```

### 场景物件库 (LevelObject)

#### 变换组件
```fc
component Transform {
    Position Vector3            // 位置
    Rotation Quaternion         // 旋转
    Scale Vector3              // 缩放
}
```

#### 实体组件
```fc
component Entity {
    Name string                 // 名称
    Active bool                // 激活状态
    Tag string                 // 标签
}
```

#### 场景函数
```fc
// 物件操作
func CreateLevelObject(id LevelObjectID, position Vector3) entity<LevelObject>
func DestroyLevelObject(obj entity<LevelObject>)
func SetObjectActive(obj entity<LevelObject>, active bool)

// 查找物件
func FindLevelObjectByName(name string) entity<LevelObject>
func FindLevelObjectsWithTag(tag string) List<entity<LevelObject>>
```

### 动画库 (Animation)

#### 动画组件
```fc
component AnimationController {
    AttachTarget entity<Entity>    // 绑定目标
    Resource string               // 资源ID
}

component AnimationClip {
    ClipLength float             // 片段长度
    PlaySpeed float              // 播放速度
    IsLoop bool                  // 是否循环
}
```

#### 动画函数
```fc
// 播放动画
func PlayAnimation(controller entity<AnimationController>, clipName string)
func StopAnimation(controller entity<AnimationController>)
func SetAnimationSpeed(controller entity<AnimationController>, speed float)
```

### UI库 (Hud)

#### UI组件示例
```fc
component ButtonWidget {
    Visible bool                // 可见性
    Enable bool                // 启用状态
    Text string                // 按钮文本
}

component TextWidget {
    Visible bool
    Text string
    FontSize int
    Color Vector3               // RGB颜色
}
```

#### UI事件
```fc
// 按钮事件
event OnTapped(player entity<Player>)          // 点击事件
event OnPressed(player entity<Player>)         // 按下事件
event OnReleased(player entity<Player>)        // 释放事件
```

---

## 组件系统

### 组件定义 (.fcc文件)

```fc
// 自定义组件示例
component HealthSystem {
    CurrentHealth float = 100.0
    MaxHealth float = 100.0
    RegenerationRate float = 1.0    // 每秒回复
    IsInvulnerable bool = false
}

component Inventory {
    Items List<ItemID> = List<ItemID>{}
    MaxSlots int = 20
    CurrentWeight float = 0.0
    MaxWeight float = 100.0
}
```

### 组件使用

```fc
graph HealthManager {
    event OnAwake() {
        // 初始化血量系统
        thisEntity<HealthSystem>.CurrentHealth = 100.0
        start HealthRegeneration()
    }
    
    event OnDamaged(damage float) {
        if !thisEntity<HealthSystem>.IsInvulnerable {
            var newHealth = thisEntity<HealthSystem>.CurrentHealth - damage
            thisEntity<HealthSystem>.CurrentHealth = Max(0.0, newHealth)
            
            if newHealth <= 0.0 {
                HandleDeath()
            }
        }
    }
    
    async func HealthRegeneration() {
        while thisEntity<HealthSystem>.CurrentHealth > 0.0 {
            WaitForSecond(1.0)
            var current = thisEntity<HealthSystem>.CurrentHealth
            var max = thisEntity<HealthSystem>.MaxHealth
            var regen = thisEntity<HealthSystem>.RegenerationRate
            
            if current < max && !thisEntity<HealthSystem>.IsInvulnerable {
                thisEntity<HealthSystem>.CurrentHealth = Min(max, current + regen)
            }
        }
    }
    
    func HandleDeath() {
        LogInfo("Entity died: " + thisEntity<Entity>.Name)
        // 死亡逻辑
    }
}
```

---

## 事件系统

### 内置事件

#### 生命周期事件
```fc
event OnAwake()                    // 实体创建时
event OnStart()                    // 游戏开始时
event OnDestroy()                  // 实体销毁时
event OnUpdate()                   // 每帧更新
```

#### 游戏事件
```fc
event OnGameStart()                // 游戏开始
event OnGameEnd()                  // 游戏结束
event OnPlayerJoin(player entity<Player>)      // 玩家加入
event OnPlayerLeave(player entity<Player>)     // 玩家离开
```

#### 物理事件
```fc
event OnTriggerEnter(other entity<Entity>)     // 触发器进入
event OnTriggerExit(other entity<Entity>)      // 触发器退出
event OnCollisionEnter(other entity<Entity>)   // 碰撞开始
event OnCollisionExit(other entity<Entity>)    // 碰撞结束
```

#### UI事件
```fc
event OnTapped(player entity<Player>)          // UI点击
event OnValueChanged(newValue float)           // 数值变化
event OnTextChanged(newText string)            // 文本变化
```

### 自定义事件

#### 定义事件 (.fcc文件)
```fc
// 游戏相关事件
event OnPlayerScoreChanged(player entity<Player>, newScore int)
event OnItemCollected(player entity<Player>, item ItemID)
event OnBossDefeated(boss entity<LevelObject>)
event OnWaveComplete(waveNumber int)

// 系统事件
event OnConfigurationChanged(config string)
event OnNetworkStatusChanged(isConnected bool)
```

#### 触发事件
```fc
graph GameManager {
    PlayerScores Map<entity<Player>, int> = Map<entity<Player>, int>{}
    
    func AddScore(player entity<Player>, points int) {
        var currentScore = PlayerScores[player]
        var newScore = currentScore + points
        PlayerScores[player] = newScore
        
        // 触发自定义事件
        OnPlayerScoreChanged(player, newScore)
    }
    
    // 处理事件
    event OnPlayerScoreChanged(player entity<Player>, newScore int) {
        LogInfo(player<Entity>.Name + " 得分: " + newScore)
        
        if newScore >= 1000 {
            LogInfo("玩家达成高分成就!")
        }
    }
}
```

---

## 编程模式

### 1. 状态机模式

```fc
enum GameState {
    MainMenu = 0
    Playing = 1
    Paused = 2
    GameOver = 3
}

graph GameStateManager {
    CurrentState GameState = GameState.MainMenu
    
    func ChangeState(newState GameState) {
        var oldState = CurrentState
        CurrentState = newState
        
        OnStateExit(oldState)
        OnStateEnter(newState)
    }
    
    func OnStateEnter(state GameState) {
        if state == GameState.Playing {
            StartGame()
        } else if state == GameState.Paused {
            PauseGame()
        } else if state == GameState.GameOver {
            EndGame()
        }
    }
    
    func OnStateExit(state GameState) {
        // 状态退出逻辑
    }
}
```

### 2. 对象池模式

```fc
graph BulletPool {
    PoolSize int = 100
    AvailableBullets List<entity<LevelObject>> = List<entity<LevelObject>>{}
    ActiveBullets List<entity<LevelObject>> = List<entity<LevelObject>>{}
    
    event OnAwake() {
        InitializePool()
    }
    
    func InitializePool() {
        for i = 0, PoolSize, 1 {
            var bullet = CreateLevelObject("BulletPrefab", Vector3{0, -100, 0})
            SetObjectActive(bullet, false)
            AvailableBullets.Add(bullet)
        }
    }
    
    func GetBullet() entity<LevelObject> {
        if AvailableBullets.Count > 0 {
            var bullet = AvailableBullets[0]
            AvailableBullets.RemoveAt(0)
            ActiveBullets.Add(bullet)
            SetObjectActive(bullet, true)
            return bullet
        }
        return null
    }
    
    func ReturnBullet(bullet entity<LevelObject>) {
        ActiveBullets.Remove(bullet)
        AvailableBullets.Add(bullet)
        SetObjectActive(bullet, false)
        bullet<Transform>.Position = Vector3{0, -100, 0}
    }
}
```

### 3. 观察者模式

```fc
graph EventManager {
    ScoreChangeListeners List<entity<Entity>> = List<entity<Entity>>{}
    
    func RegisterScoreListener(listener entity<Entity>) {
        if !ScoreChangeListeners.Contains(listener) {
            ScoreChangeListeners.Add(listener)
        }
    }
    
    func UnregisterScoreListener(listener entity<Entity>) {
        ScoreChangeListeners.Remove(listener)
    }
    
    func NotifyScoreChange(player entity<Player>, newScore int) {
        for i = 0, ScoreChangeListeners.Count, 1 {
            var listener = ScoreChangeListeners[i]
            // 通知监听器（需要监听器实现相应接口）
            OnPlayerScoreChanged(player, newScore)
        }
    }
}
```

### 4. 缓动动画模式

```fc
graph TweenManager {
    // 缓动函数
    func EaseInOut(t float) float {
        if t < 0.5 {
            return 2.0 * t * t
        } else {
            var temp = 1.0 - t
            return 1.0 - 2.0 * temp * temp
        }
    }
    
    func EaseIn(t float) float {
        return t * t
    }
    
    func EaseOut(t float) float {
        return 1.0 - (1.0 - t) * (1.0 - t)
    }
    
    // 通用Tween函数
    async func TweenFloat(from float, to float, duration float, easeType string, out var result float) {
        var frameTime = 16  // 60FPS
        var totalTime = duration * 1000
        var totalFrames = totalTime / frameTime
        
        for frame = 0, totalFrames, 1 {
            var t = frame / totalFrames
            var easedT = t
            
            if easeType == "EaseIn" {
                easedT = EaseIn(t)
            } else if easeType == "EaseOut" {
                easedT = EaseOut(t)
            } else if easeType == "EaseInOut" {
                easedT = EaseInOut(t)
            }
            
            result = from + (to - from) * easedT
            WaitForMillisecond(frameTime)
        }
        
        result = to  // 确保最终值准确
    }
    
    // Vector3插值
    async func TweenVector3(target entity<LevelObject>, from Vector3, to Vector3, duration float, easeType string) {
        var frameTime = 16
        var totalTime = duration * 1000
        var totalFrames = totalTime / frameTime
        
        for frame = 0, totalFrames, 1 {
            var t = frame / totalFrames
            var easedT = EaseInOut(t)  // 默认使用EaseInOut
            
            var currentPos = LerpVector3(from, to, easedT)
            target<Transform>.Position = currentPos
            WaitForMillisecond(frameTime)
        }
        
        target<Transform>.Position = to
    }
}
```

---

## 最佳实践

### 1. 代码组织

#### 文件结构建议
```
Assets/
├── Scripts/
│   ├── Core/
│   │   ├── GameManager.fcg
│   │   └── EventManager.fcg
│   ├── Player/
│   │   ├── PlayerController.fcg
│   │   └── PlayerInventory.fcg
│   ├── UI/
│   │   ├── MainMenu.fcg
│   │   └── HUD.fcg
│   └── Utils/
│       ├── MathUtils.fcg
│       └── TweenUtils.fcg
├── Libraries/
│   ├── GameComponents.fcc
│   ├── GameEvents.fcc
│   └── GameEnums.fcc
└── Data/
    └── GameConfiguration.csv
```

#### 命名约定
```fc
// 组件名称：PascalCase
component PlayerController { }
component HealthSystem { }

// 函数名称：PascalCase
func CalculateDamage() { }
func GetPlayerCount() { }

// 变量名称：camelCase
var playerHealth int
var maxJumpHeight float

// 常量：UPPER_CASE
var MAX_PLAYERS int = 100
var DEFAULT_SPEED float = 5.0

// 事件名称：On + PascalCase
event OnPlayerDeath() { }
event OnGameStart() { }
```

### 2. 性能优化

#### 避免频繁内存分配
```fc
// ❌ 不好：每帧创建新列表
event OnUpdate() {
    var nearbyObjects = List<entity<LevelObject>>{}
    // 处理逻辑
}

// ✅ 好：复用列表
graph ObjectManager {
    NearbyObjectsCache List<entity<LevelObject>> = List<entity<LevelObject>>{}
    
    event OnUpdate() {
        NearbyObjectsCache.Clear()
        // 重复使用同一个列表
    }
}
```

#### 合理使用异步
```fc
// ❌ 不好：阻塞式操作
func SpawnMultipleEnemies() {
    for i = 0, 100, 1 {
        CreateEnemy()
        WaitForMillisecond(100)  // 阻塞10秒
    }
}

// ✅ 好：分批异步处理
async func SpawnMultipleEnemiesAsync() {
    var batchSize = 5
    for batch = 0, 20, 1 {
        for i = 0, batchSize, 1 {
            CreateEnemy()
        }
        WaitForMillisecond(50)  // 每批间隔
    }
}
```

### 3. 错误处理

```fc
graph SafePlayerManager {
    func GetPlayerSafely(index int) entity<Player> {
        var allPlayers = GetAllPlayers()
        
        if index < 0 || index >= allPlayers.Count {
            LogWarning("无效的玩家索引: " + index)
            return null
        }
        
        return allPlayers[index]
    }
    
    func TeleportPlayerSafely(player entity<Player>, position Vector3) {
        if player == null {
            LogError("尝试传送空玩家引用")
            return
        }
        
        if player<Player>.State == PlayerStateType.Dead {
            LogWarning("无法传送死亡玩家")
            return
        }
        
        TeleportPlayer(player, position)
        LogInfo("玩家传送成功: " + player<Entity>.Name)
    }
}
```

### 4. 调试支持

```fc
graph DebugManager {
    DebugMode bool = true
    
    func DebugLog(message string) {
        if DebugMode {
            LogInfo("[DEBUG] " + message)
        }
    }
    
    func DebugDrawLine(from Vector3, to Vector3, color Vector3) {
        if DebugMode {
            // 绘制调试线条（如果支持）
        }
    }
    
    event OnKeyPressed(key string) {
        if DebugMode {
            if key == "F1" {
                ShowDebugInfo()
            } else if key == "F2" {
                TeleportToNextCheckpoint()
            }
        }
    }
}
```

---

## 调试指南

### 1. 日志调试

```fc
graph DebuggingExample {
    func ProcessPlayerAction(player entity<Player>, action string) {
        LogInfo("开始处理玩家动作: " + player<Entity>.Name + " -> " + action)
        
        var startTime = GetCurrentTimestamp()
        
        if action == "jump" {
            HandleJump(player)
        } else if action == "shoot" {
            HandleShoot(player)
        } else {
            LogWarning("未知动作: " + action)
            return
        }
        
        var endTime = GetCurrentTimestamp()
        LogInfo("动作处理完成，耗时: " + (endTime - startTime) + "ms")
    }
}
```

### 2. 断言和验证

```fc
graph ValidationHelper {
    func AssertNotNull(obj object, message string) {
        if obj == null {
            LogError("断言失败: " + message)
            // 可以选择停止执行或其他处理
        }
    }
    
    func ValidatePlayerState(player entity<Player>) bool {
        if player == null {
            LogError("玩家引用为空")
            return false
        }
        
        if player<Player>.Health < 0 {
            LogError("玩家血量异常: " + player<Player>.Health)
            return false
        }
        
        return true
    }
}
```

### 3. 性能监控

```fc
graph PerformanceMonitor {
    FrameCount int = 0
    LastSecondTime int64 = 0
    FPS float = 0.0
    
    event OnUpdate() {
        FrameCount = FrameCount + 1
        var currentTime = GetCurrentTimestamp()
        
        if currentTime - LastSecondTime >= 1000 {
            FPS = FrameCount / ((currentTime - LastSecondTime) / 1000.0)
            LogInfo("当前FPS: " + FPS)
            
            FrameCount = 0
            LastSecondTime = currentTime
        }
    }
}
```

---

## 常见问题

### Q1: 如何在不同平台间共享数据？

**A:** 使用全局变量或数据存储API：

```fc
// 服务端脚本
[platform_server]
graph ServerData {
    GlobalPlayerCount int = 0
    
    event OnPlayerJoin(player entity<Player>) {
        GlobalPlayerCount = GlobalPlayerCount + 1
        // 同步到客户端
        SyncDataToClient("playerCount", GlobalPlayerCount)
    }
}

// 客户端脚本
[platform_client]
graph ClientUI {
    event OnDataReceived(key string, value object) {
        if key == "playerCount" {
            UpdatePlayerCountUI(value as int)
        }
    }
}
```

### Q2: 如何处理大量实体的性能问题？

**A:** 使用对象池和LOD系统：

```fc
graph EntityManager {
    EntityPool Map<string, List<entity<LevelObject>>> = Map<string, List<entity<LevelObject>>>{}
    MaxDistance float = 100.0
    
    func GetPooledEntity(type string) entity<LevelObject> {
        if !EntityPool.ContainsKey(type) {
            EntityPool[type] = List<entity<LevelObject>>{}
        }
        
        var pool = EntityPool[type]
        if pool.Count > 0 {
            var entity = pool[0]
            pool.RemoveAt(0)
            SetObjectActive(entity, true)
            return entity
        }
        
        return CreateLevelObject(type, Vector3{0, 0, 0})
    }
    
    func UpdateEntityLOD() {
        var playerPos = GetLocalPlayer()<Transform>.Position
        var allEntities = FindLevelObjectsWithTag("Managed")
        
        for i = 0, allEntities.Count, 1 {
            var entity = allEntities[i]
            var distance = Vector3Distance(entity<Transform>.Position, playerPos)
            
            if distance > MaxDistance {
                SetObjectActive(entity, false)
            }
        }
    }
}
```

### Q3: 如何实现复杂的UI交互？

**A:** 使用事件驱动的UI系统：

```fc
graph UIManager {
    CurrentMenu string = "MainMenu"
    MenuStack List<string> = List<string>{}
    
    func ShowMenu(menuName string) {
        if CurrentMenu != "" {
            MenuStack.Add(CurrentMenu)
            HideMenu(CurrentMenu)
        }
        
        CurrentMenu = menuName
        DisplayMenu(menuName)
    }
    
    func GoBack() {
        if MenuStack.Count > 0 {
            var previousMenu = MenuStack[MenuStack.Count - 1]
            MenuStack.RemoveAt(MenuStack.Count - 1)
            
            HideMenu(CurrentMenu)
            CurrentMenu = previousMenu
            DisplayMenu(previousMenu)
        }
    }
    
    event OnMenuButtonTapped(player entity<Player>, buttonName string) {
        if buttonName == "Settings" {
            ShowMenu("SettingsMenu")
        } else if buttonName == "Back" {
            GoBack()
        }
    }
}
```

### Q4: 如何处理网络同步？

**A:** 使用事件系统进行状态同步：

```fc
// 服务端权威数据
[platform_server]
graph NetworkManager {
    PlayerPositions Map<entity<Player>, Vector3> = Map<entity<Player>, Vector3>{}
    
    event OnPlayerMove(player entity<Player>, newPosition Vector3) {
        // 验证移动合法性
        if ValidateMovement(player, newPosition) {
            PlayerPositions[player] = newPosition
            BroadcastPlayerPosition(player, newPosition)
        }
    }
    
    func BroadcastPlayerPosition(player entity<Player>, position Vector3) {
        // 广播给所有客户端
        var allPlayers = GetAllPlayers()
        for i = 0, allPlayers.Count, 1 {
            if allPlayers[i] != player {
                SendToClient(allPlayers[i], "playerPosition", player, position)
            }
        }
    }
}

// 客户端接收同步
[platform_client]
graph ClientSync {
    event OnNetworkMessage(messageType string, data object) {
        if messageType == "playerPosition" {
            var playerData = data as PlayerPositionData
            UpdatePlayerPosition(playerData.Player, playerData.Position)
        }
    }
}
```

---

## 🔗 相关资源

- **官方文档**: https://ffcraftland.garena.com/en/docs/api

---

*本文档基于 Free Fire Craftland Studio 官方文档和内置库源码分析生成，包含完整的语法参考、API文档和最佳实践指南。*
