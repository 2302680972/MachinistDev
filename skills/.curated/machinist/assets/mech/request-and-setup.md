# 申请与设置

摘录自示例工程的 TS 视图片段，用于对照"从对象池申请并复位到目标状态"的写法。

## 设置子弹

```ts
/**
  * name: Bullet'SetBullet
  * sourcePath: map.map_/329/scripts/Bullet'SetBullet.code
  */
@BeScript({ name: "Bullet'SetBullet", color: [0, 138, 255], comment: "设置一个普通子弹" })
public Script__X_Bullet_x27_SetBullet($pos: BeVector3, $direction: BeVector3, $type: BeLong, $faction: BeLong, $player: BeStruct): BeBool {
  // @locals-begin
  var $success: BeBool
  var $bulletKey: BeString
  var $info: BeStruct
  var $modelKey: BeString
  var $unusedList: BeList
  var $index: BeFloat
  var $exist: BeBool
  var $obj: BeMech
  // @locals-end

  $bulletKey = G.create.string($type)
  $info = this.BulletRegInfo.get<"BeStruct">($bulletKey, BeBool.fromBeConst("0"))
  $modelKey = $info.get<"BeString">(BeString.fromBeConst("modelKey"), BeBool.fromBeConst("0"))
  $unusedList = this.BulletUnused.get<"BeList">($modelKey, BeBool.fromBeConst("0"))
  $index = $unusedList.count()
  $exist = G.float.gt($index, BeFloat.fromBeConst("0"))
  if (G.create.bool($exist)) {
    $index.minusEqual(BeFloat.fromBeConst("1"))
    $obj = $unusedList.read<"BeMech">($index)
    $obj.setActive(BeBool.fromBeConst("1"))
    $unusedList.remove($index)
  }
  if (G.bool.not($exist)) {
    $obj = Act.self<Device_Scenario_329>(this).Script__X_Bullet_x27_New($modelKey, BeVector3.fromBeConst("1e+03,0,0"), $direction)
  }
  // 初始化子弹属性...
  $success = G.create.bool(BeBool.fromBeConst("1"))
  // --- 隐式返回勿修改
  return $success
}
```

## 设置礼物模型

```ts
/**
  * name: GiftModel'SetGiftModel
  * sourcePath: map.map_/329/scripts/GiftModel'SetGiftModel.code
  */
@BeScript({ name: "GiftModel'SetGiftModel", color: [0, 138, 255], comment: "设置一个普通子弹" })
public Script__X_GiftModel_x27_SetGiftModel($modelKey: BeString, $text: BeString, $player: BeStruct, $skipUI: BeBool, $mode: BeLong): void {
  // @locals-begin
  var $unusedList: BeList
  var $index: BeFloat
  var $exist: BeBool
  var $obj: BeMech
  // @locals-end

  $unusedList = this.BulletUnused.get<"BeList">($modelKey, BeBool.fromBeConst("0"))
  $index = $unusedList.count()
  $exist = G.float.gt($index, BeFloat.fromBeConst("0"))
  if (G.create.bool($exist)) {
    $index.minusEqual(BeFloat.fromBeConst("1"))
    $obj = $unusedList.read<"BeMech">($index)
    $obj.setActive(BeBool.fromBeConst("1"))
    $unusedList.remove($index)
  }
  if (G.bool.not($exist)) {
    $obj = Act.self<Device_Scenario_329>(this).Script__X_GiftModel_x27_New($modelKey)
  }
  $obj.callFun(BeString.fromBeConst("Generic'PlayAnimation"), $modelKey, $text, $player, $skipUI, $mode)
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...], comment: "..." })` 格式
- 先从 Unused 列表尝试获取可复用对象
- 如果没有可复用对象，则创建新对象
- 获取后设置 `setActive(true)` 并从 Unused 列表移除
