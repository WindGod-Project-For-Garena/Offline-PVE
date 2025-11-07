---
description: FC语言的完整开发示例，每个示例都是原子化的，可以独立使用。
globs: *.fcg,*.fcc
applyTo: "**/*. {fcc,fcg}"
alwaysApply: true
---

# FC语言开发示例大全

本文档包含FC语言的完整开发示例，每个示例都是原子化的，可以独立使用。
参考示例的实现，如遇到引用缺失类问题，请务必遍历fcc库查找是否可以修复引用缺失问题。

## 基础图形定义

### 普通图形
```fc
import "StdLibrary.fcc" as std

graph MyScript {
    PlayerCount int = 0
    
    event OnAwake() {
        LogInfo("Script initialized")
    }
}
```

### 静态图形（全局脚本）
```fc
import "StdLibrary.fcc" as std
import "List.fcc" as list

static graph GlobalManager {
    MaxPlayers int = 10
    
    func GetPlayerCount() int {
        return list.Length(GetAllPlayers())
    }
}
```

## 实体和组件操作

### 访问当前实体组件
```fc
import "StdLibrary.fcc" as std

graph EntityHandler {
    event OnAwake() {
        // 访问Transform组件
        thisEntity<Transform>.Position = Vector3{0, 1, 0}
        thisEntity<Transform>.Rotation = Quaternion{0, 0, 0, 1}
        
        // 访问外观组件
        thisEntity<BasicAppearance>.DiffuseColor = #FF0000FF
    }
}
```

### 检查组件是否存在
```fc
import "StdLibrary.fcc" as std

graph ComponentChecker {
    event OnAwake() {
        if HasComponent(thisEntity, typeof(Player)) {
            thisEntity<Player>.RunSpeedScale = 1.5
            LogInfo("Player component found")
        }
    }
}
```

### 实体类型转换
```fc
import "StdLibrary.fcc" as std

graph EntityConverter {
    event OnEntityEnter(enterEntity entity<Entity>) {
        if HasComponent(enterEntity, typeof(Player)) {
            var player = enterEntity<Player>
            LogInfo("Player name: " + player.Name)
        }
    }
}
```

## 预制体和资产操作

### 从预制体创建实体
```fc
import "StdLibrary.fcc" as std

graph PrefabCreator {
    event OnAwake() {
        // 创建场景方块
        CreateFromPrefab(out var cube, EResPrefab.SceneCube)
        cube<Transform>.Position = Vector3{5, 0, 5}
        
        // 创建炸弹
        CreateFromPrefab(out var bomb, EResPrefab.SceneBomb)
        bomb<Transform>.Position = thisEntity<Transform>.Position
    }
}
```

### 获取场景物件
```fc
import "StdLibrary.fcc" as std
import "Player.fcc" as player

graph SceneObjectGetter {
    event OnAwake() {
        // 获取场景中的胜利点
        var winnerView = EResSceneIsleLand.WinnerView as entity<LevelObject>
        var targetPos = winnerView<Transform>.Position
        
        // 传送玩家到胜利点
        Teleport(thisEntity<Player>, targetPos, winnerView<Transform>.Rotation)
    }
}
```

### 动态获取脚本资产
```fc
import "StdLibrary.fcc" as std
import "EditorGenLib.fcc" as gen
import Res from "EditorGenLib.fcc"

graph DynamicScriptLoader {
    event OnAwake() {
        var scriptKey = "SomeEffect"
        
        // 检查脚本是否存在
        if ContainKey(Res.FCG, scriptKey) {
            // 动态添加脚本
            AddScript(thisEntity, Res.FCG[scriptKey as EResKeyFCG])
        }
    }
}
```

## UI操作

### 创建自定义UI
```fc
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud

graph UICreator {
    MainUI entity<CustomHud>
    
    event OnAwake() {
        // 创建玩家HUD
        CreateCustomUI(out var ui, thisEntity<Player>, EResUI.BombHUD)
        MainUI = ui
        
        // 初始隐藏UI
        MainUI<CustomUI>.IsVisible = false
    }
    
    event OnMyGameStart(level int) {
        // 游戏开始时显示UI
        MainUI<CustomUI>.IsVisible = true
    }
}
```

### UI按钮点击事件
```fc
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud

graph ButtonHandler {
    ClickCount int = 0
    
    event OnTapped(player entity<Player>) {
        ClickCount += 1
        LogInfo("Button clicked by: " + player<Player>.Name)
        LogInfo("Click count: " + ClickCount)
        
        // 执行点击后的逻辑
        // ...
    }
}
```

### 设置UI控件属性
```fc
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud

graph UIController {
    event OnUpdate() {
        // 根据条件控制按钮是否可点击
        var canUse = CheckSomeCondition()
        thisEntity<UIWidget>.Active = canUse
    }
    
    func CheckSomeCondition() bool {
        return true // 你的逻辑
    }
}
```

### 设置图标资源
```fc
import "StdLibrary.fcc" as std

graph IconSetter {
    event OnAwake() {
        // 设置道具图标
        thisEntity<ItemTrigger>.Icon<Icon>.Resource = EResSprite.ui_icon_skill_Awakening
    }
}
```

## 异步编程

### 基础异步函数
```fc
import "StdLibrary.fcc" as std

graph AsyncHandler {
    event OnAwake() {
        // 等待异步函数完成
        wait DelayedAction()
        
        // 异步执行，不等待
        start BackgroundTask()
    }
    
    async func DelayedAction() {
        WaitForMillisecond(2000)
        LogInfo("Delayed action completed")
    }
    
    async func BackgroundTask() {
        WaitForNextFrame()
        LogInfo("Background task running")
    }
}
```

### 带输出参数的异步函数
```fc
import "StdLibrary.fcc" as std

graph AsyncCalculator {
    event OnAwake() {
        wait CalculateSum(10, 20, out var result)
        LogInfo("Result: " + result)
    }
    
    async func CalculateSum(a int, b int, out var sum int) {
        WaitForMillisecond(1000)
        sum = a + b
    }
}
```

## 集合操作

### 列表操作
```fc
import "StdLibrary.fcc" as std
import "List.fcc" as list

graph ListHandler {
    PlayerList List<entity<Player>>
    
    event OnAwake() {
        // 创建新列表
        PlayerList = list.New(0, 0)
        
        // 添加元素
        for index, player in GetAllPlayers() {
            list.Append(PlayerList, player)
        }
        
        // 遍历列表
        for i, player in PlayerList {
            LogInfo("Player " + i + ": " + player<Player>.Name)
        }
        
        // 检查是否包含元素
        if list.Contain(PlayerList, somePlayer) {
            LogInfo("Player found in list")
        }
    }
}
```

### 字典操作
```fc
import "StdLibrary.fcc" as std
import "Map.fcc" as map

graph MapHandler {
    EntityMapping Map<int, entity>
    
    event OnAwake() {
        // 创建新字典
        EntityMapping = map.New()
        
        // 添加键值对
        EntityMapping[1] = thisEntity
        EntityMapping[2] = someOtherEntity
        
        // 检查键是否存在
        if ContainKey(EntityMapping, 1) {
            var entity = EntityMapping[1]
            LogInfo("Entity found: " + entity<Entity>.Name)
        }
        
        // 移除键值对
        map.Remove(EntityMapping, 1)
    }
}
```

## 标签系统

### 添加和检查标签
```fc
import "StdLibrary.fcc" as std

graph TagManager {
    event OnAwake() {
        // 添加标签
        AddTag(thisEntity, "KeyPlayer")
        AddTag(thisEntity, "SpecialUnit")
        
        // 检查标签
        if HasTag(thisEntity, "KeyPlayer") {
            LogInfo("This entity has the key")
        }
        
        // 移除标签
        RemoveTag(thisEntity, "SpecialUnit")
    }
}
```

## 事件系统

### 分发事件给单个实体
```fc
import "StdLibrary.fcc" as std
import "EditorGenLib.fcc" as gen

graph EventDispatcher {
    event OnAwake() {
        // 分发事件给全局实体
        DispatchEvent(OnMyGameStart, globalEntity, List<object>{1})
        
        // 分发事件给特定实体
        DispatchEvent(gen.OnSceneCubeCreated, thisEntity, nil)
    }
}
```

### 分发事件给所有玩家
```fc
import "StdLibrary.fcc" as std

graph PlayerEventDispatcher {
    func NotifyAllPlayers(message string) {
        for index, player in GetAllPlayers() {
            DispatchEvent(OnPlayerMessage, player, List<object>{message})
        }
    }
    
    event OnAwake() {
        NotifyAllPlayers("Game started!")
    }
}
```

### 触发器事件处理
```fc
import "StdLibrary.fcc" as std

graph TriggerHandler {
    event OnEntityEnter(enterEntity entity<Entity>) {
        // 检查进入的实体类型
        if HasComponent(enterEntity, typeof(Player)) {
            var player = enterEntity<Player>
            LogInfo("Player entered: " + player.Name)
            
            // 执行相应逻辑
            HandlePlayerEnter(player)
        }
    }
    
    func HandlePlayerEnter(player entity<Player>) {
        // 处理玩家进入逻辑
    }
}
```

## 战斗和生命周期

### 玩家消除事件
```fc
import "StdLibrary.fcc" as std
import "Combat.fcc" as combat
import "Player.fcc" as player

graph CombatHandler {
    event OnEliminated(attackerEntity entity<Entity>, attackedType DamageType, hitBodyPart HitBodyPart) {
        LogInfo("Player eliminated")
        
        // 检查是否携带钥匙
        if HasTag(thisEntity, "KeyPlayer") {
            // 掉落钥匙
            RemoveTag(thisEntity, "KeyPlayer")
            CreateFromPrefab(out var keyTrigger, EResPrefab.KeyTrigger)
            keyTrigger<Transform>.Position = thisEntity<Player>.Position
        }
    }
}
```

### 消除和复活玩家
```fc
import "StdLibrary.fcc" as std
import "Combat.fcc" as combat
import "Player.fcc" as player

graph PlayerLifecycle {
    async func EliminateAndRevive(player entity<Player>) {
        // 消除玩家
        Eliminate(player)
        
        // 等待3秒后复活
        WaitForMillisecond(3000)
        Revive(player)
    }
    
    event OnSomeEvent() {
        start EliminateAndRevive(somePlayer)
    }
}
```

## 数学和随机数

### 随机数生成
```fc
import "StdLibrary.fcc" as std
import "Math.fcc" as math

graph RandomGenerator {
    event OnAwake() {
        // 生成随机整数 (1-9)
        var randomInt = math.RandomInt(1, 10)
        
        // 生成随机浮点数 (0.0-1.0)
        var randomFloat = math.RandomFloat(0.0, 1.0)
        
        // 使用随机数设置位置
        var randomPos = Vector3{randomInt, 0, randomInt}
        thisEntity<Transform>.Position = randomPos
        
        LogInfo("Random position: " + randomPos)
    }
}
```

### 数学计算
```fc
import "StdLibrary.fcc" as std
import "Math.fcc" as math

graph MathOperations {
    func CalculateDistance(pos1 Vector3, pos2 Vector3) float {
        var dx = pos2.X - pos1.X
        var dz = pos2.Z - pos1.Z
        return math.Sqrt(dx*dx + dz*dz)
    }
    
    event OnUpdate() {
        var playerPos = thisEntity<Transform>.Position
        var targetPos = Vector3{0, 0, 0}
        var distance = CalculateDistance(playerPos, targetPos)
        
        if distance < 2.0 {
            LogInfo("Player is near target")
        }
    }
}
```

## 字符串操作

### 字符串处理
```fc
import "StdLibrary.fcc" as std
import "Strings.fcc" as strings

graph StringHandler {
    event OnAwake() {
        var playerName = thisEntity<Player>.Name
        
        // 检查字符串长度
        if strings.Length(playerName) > 0 {
            LogInfo("Player name: " + playerName)
        }
        
        // 字符串拼接
        var message = "Welcome, " + playerName + "!"
        LogInfo(message)
        
        // 格式化字符串
        var formattedMsg = Format("Player %v joined the game", List<string>{playerName})
        LogInfo(formattedMsg)
    }
}
```

## 物理检测

### 区域检测
```fc
import "StdLibrary.fcc" as std
import "Physics.fcc" as physics

graph PhysicsChecker {
    func CheckExplosionArea(center Vector3, size float) {
        for index, player in GetAllPlayers() {
            // 检查玩家是否在爆炸范围内
            var inX = physics.IsInsideBox(player, center, size, 1, 1, Vector3{0,0,0})
            var inZ = physics.IsInsideBox(player, center, 1, size, 1, Vector3{0,0,0})
            
            if inX || inZ {
                LogInfo("Player in explosion area: " + player<Player>.Name)
                // 处理爆炸效果
            }
        }
    }
}
```

## 提示和通知

### 显示玩家提示
```fc
import "StdLibrary.fcc" as std

graph NotificationManager {
    func ShowTipToPlayer(player entity<Player>, message string, color int, duration int) {
        NotifyShowTips(player, message, color, duration)
    }
    
    func ShowTipToAllPlayers(message string) {
        for index, player in GetAllPlayers() {
            NotifyShowTips(player, message, #FFFFFFFF, 3000)
        }
    }
    
    event OnAwake() {
        ShowTipToAllPlayers("Game initialized!")
    }
}
```

## 传送和移动

### 玩家传送
```fc
import "StdLibrary.fcc" as std
import "Player.fcc" as player

graph TeleportManager {
    func TeleportToSpawn(player entity<Player>) {
        var spawnPoint = Vector3{0, 1, 0}
        var spawnRotation = Quaternion{0, 0, 0, 1}
        
        Teleport(player, spawnPoint, spawnRotation)
        LogInfo("Player teleported to spawn")
    }
    
    func TeleportToWinArea(player entity<Player>) {
        var winnerView = EResSceneIsleLand.WinnerView as entity<LevelObject>
        Teleport(player, winnerView<Transform>.Position, winnerView<Transform>.Rotation)
    }
}
```

## 游戏流程控制

### 游戏阶段管理
```fc
import "StdLibrary.fcc" as std
import "Player.fcc" as player

graph GamePhaseManager {
    event OnMyGameStart(level int) {
        LogInfo("Game started - Level: " + level)
        
        // 禁用玩家移动
        SetAllPlayersMoveStatus(false)
        
        // 初始化关卡
        wait InitializeLevel(level)
        
        // 启用玩家移动
        SetAllPlayersMoveStatus(true)
    }
    
    async func InitializeLevel(level int) {
        LogInfo("Initializing level: " + level)
        WaitForMillisecond(1000)
        LogInfo("Level initialized")
    }
    
    event OnMyGameEnd() {
        LogInfo("Game ended")
        WaitForMillisecond(3000)
        EndCurrentPhase()
    }
}
```

### 倒计时系统
```fc
import "StdLibrary.fcc" as std
import "Strings.fcc" as strings

graph CountdownManager {
    async func StartCountdown(seconds int) {
        for i = seconds, 0, -1 {
            var message = Format("Game starts in: %v", List<int>{i})
            
            // 通知所有玩家
            for index, player in GetAllPlayers() {
                NotifyShowTips(player, message, #FFFFFFFF, 1000)
            }
            
            WaitForMillisecond(1000)
        }
        
        // 倒计时结束
        DispatchEvent(OnMyGameStart, globalEntity, List<int>{1})
    }
}
```

## 资源清理

### 实体销毁和清理
```fc
import "StdLibrary.fcc" as std

graph ResourceCleaner {
    MyUI entity<CustomHud>
    
    event OnDestroy() {
        // 清理UI资源
        if MyUI != nil {
            Destroy(MyUI)
            MyUI = nil
        }
        
        LogInfo("Resources cleaned up")
    }
    
    func DestroyAfterDelay(target entity, delay int) {
        start DelayedDestroy(target, delay)
    }
    
    async func DelayedDestroy(target entity, delay int) {
        WaitForMillisecond(delay)
        Destroy(target)
    }
}
```

## 常见错误和修正

### 错误示例对比

#### ❌ 错误的组件访问
```fc
// 错误：使用点号语法
thisEntity.Transform.Position = Vector3{0, 1, 0}
```

#### ✅ 正确的组件访问
```fc
// 正确：使用尖括号语法
thisEntity<Transform>.Position = Vector3{0, 1, 0}
```

#### ❌ 错误的实体类型声明
```fc
// 错误：直接使用组件类型
var player Player
```

#### ✅ 正确的实体类型声明
```fc
// 正确：使用entity包装
var player entity<Player>
```

#### ❌ 错误的异步调用
```fc
// 错误：直接调用异步函数
DelayedFunction()
```

#### ✅ 正确的异步调用
```fc
// 正确：使用wait或start关键字
wait DelayedFunction()
start DelayedFunction()
```

#### ❌ 错误的事件分发
```fc
// 错误：直接传递参数
DispatchEvent(eventID, entity, param1, param2)
```

#### ✅ 正确的事件分发
```fc
// 正确：参数包装在List中
DispatchEvent(eventID, entity, List<object>{param1, param2})
``` 