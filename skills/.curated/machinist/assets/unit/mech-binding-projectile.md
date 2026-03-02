# 机械绑定与投射物

摘录自示例工程的 TS 视图片段，用于对照"投射物放置时携带玩家结构"的写法。

```ts
/**
  * name: Launcher'指定坐标放置
  * sourcePath: map.map_/331/scripts/Launcher'指定坐标放置.code
  */
@BeScript({ name: "Launcher'指定坐标放置", comment: "发射飞刀用" })
public Script__X_Launcher_x27_指定坐标放置(position: BeVector3, direction: BeVector3, type: BeLong, player: BeStruct, HPMul: BeFloat, crit: BeFloat, initHp: BeFloat, useManualSpeed: BeBool, manualSpeed: BeFloat): BeBool {
  // @locals-begin
  var ret: BeBool
  // @locals-end

  ret = Act.self<Device_基地_331>(this).Script__X_Launcher_x27_AbstractSetBullet(position, direction, type, this.X_Attr_x27_Faction, player, HPMul, crit, initHp, useManualSpeed, manualSpeed)
  // --- 隐式返回勿修改
  return ret
}

/**
  * name: Launcher'AbstractSetBullet
  * sourcePath: map.map_/331/scripts/Launcher'AbstractSetBullet.code
  */
@BeScript({ name: "Launcher'AbstractSetBullet", color: [255, 0, 0], comment: "放置子弹的抽象接口" })
public Script__X_Launcher_x27_AbstractSetBullet($position: BeVector3, $direction: BeVector3, $type: BeLong, $faction: BeLong, $player: BeStruct): BeBool {
  // @locals-begin
  var $ret: BeBool
  // @locals-end

  $ret = this.map_handler.callReturn<"BeBool">(BeString.fromBeConst("Bullet'SetBullet"), $position, $direction, $type, $faction, $player)
  // --- 隐式返回勿修改
  return $ret
}

/**
  * name: Public'AddBullet
  * sourcePath: map.map_/331/scripts/Public'AddBullet.code
  */
@BeScript({ name: "Public'AddBullet", comment: "注入各种子弹" })
public Script__X_Public_x27_AddBullet(key: BeString, amount: BeLong, player: BeStruct): void {
  // @locals-begin
  // @locals-end

  Act.self<Device_基地_331>(this).Script__X_Skills_x27_Increase(key, amount, player)
  if (G.string.equal(key, BeString.fromBeConst("2000"))) {
    Act.self<Device_基地_331>(this).Script__X_UI_x27_守方刷新弹药数()
  }
  if (G.string.equal(key, BeString.fromBeConst("1000"))) {
    Act.self<Device_基地_331>(this).Script__X_UI_x27_攻方刷新弹药数()
  }
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...], comment: "..." })` 格式
- 投射物放置时携带 `player: BeStruct` 参数
- 通过 `map_handler.callReturn` 调用地图脚本
