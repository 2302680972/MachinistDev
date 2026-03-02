# 回收与物理清理

摘录自示例工程的 TS 视图片段，用于对照"回收前的物理清理 + 回收进对象池"的写法。

## 回收模型

```ts
/**
  * name: Bullet'OnReclaim
  * sourcePath: map.map_/329/scripts/Bullet'OnReclaim.code
  */
@BeScript({ name: "Bullet'OnReclaim", color: [0, 138, 255], comment: "回收模型" })
public Script__X_Bullet_x27_OnReclaim($obj: BeMech, $modelType: BeLong): void {
  // @locals-begin
  var $key: BeString
  var $unused: BeList
  // @locals-end

  $key = G.create.string($modelType)
  $unused = this.BulletUnused.get<"BeList">($key, BeBool.fromBeConst("0"))
  $unused.add($obj)
  $obj.setActive(BeBool.fromBeConst("0"))
  // --- 隐式返回勿修改
  return
}
```

## 回收礼物模型

```ts
/**
  * name: GiftModel'OnReclaim
  * sourcePath: map.map_/329/scripts/GiftModel'OnReclaim.code
  */
@BeScript({ name: "GiftModel'OnReclaim", color: [0, 138, 255], comment: "回收事件" })
public Script__X_GiftModel_x27_OnReclaim($obj: BeMech, $modelKey: BeString): void {
  // @locals-begin
  var $unused: BeList
  // @locals-end

  $unused = this.BulletUnused.get<"BeList">($modelKey, BeBool.fromBeConst("0"))
  $unused.add($obj)
  $obj.setActive(BeBool.fromBeConst("0"))
  // --- 隐式返回勿修改
  return
}
```

## 强制回收子弹

```ts
/**
  * name: Bullet'ForceReclaimByModel
  * sourcePath: map.map_/329/scripts/Bullet'ForceReclaimByModel.code
  */
@BeScript({ name: "Bullet'ForceReclaimByModel", comment: "按照使用的模型强制回收子弹" })
public Script__X_Bullet_x27_ForceReclaimByModel($key: BeString): void {
  // @locals-begin
  var $bullet: BeList
  var $index: BeFloat
  var $obj: BeMech
  var $value: BeFloat
  var $faction: BeLong
  // @locals-end

  $bullet = this.Bullet.get<"BeList">($key, BeBool.fromBeConst("0"))
  $index = $bullet.count()
  while (G.float.gt($index, BeFloat.fromBeConst("0"))) {
    $index.minusEqual(BeFloat.fromBeConst("1"))
    $obj = $bullet.read<"BeMech">($index)
    $value = G.create.float(BeFloat.fromBeConst("99999"))
    $faction = G.create.long(BeLong.fromBeConst("0"))
    $obj.callFun(BeString.fromBeConst("Public'Damage"), $value, $faction)
  }
  // --- 隐式返回勿修改
  return
}
```

## 强制回收全部子弹

```ts
/**
  * name: Bullet'ForceReclaimAll
  * sourcePath: map.map_/329/scripts/Bullet'ForceReclaimAll.code
  */
@BeScript({ name: "Bullet'ForceReclaimAll", color: [0, 138, 255], comment: "强制回收全部子弹" })
public Script__X_Bullet_x27_ForceReclaimAll(): void {
  // @locals-begin
  // @locals-end

  Act.self<Device_Scenario_329>(this).Script__X_Bullet_x27_ForceReclaimByModel(BeString.fromBeConst("0"))
  Act.self<Device_Scenario_329>(this).Script__X_Bullet_x27_ForceReclaimByModel(BeString.fromBeConst("1"))
  // ... 其他模型类型
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...], comment: "..." })` 格式
- 回收时将对象加入 Unused 列表并设置 `setActive(false)`
- 强制回收通过调用 `Public'Damage` 触发死亡流程
